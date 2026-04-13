# Unity Project Architecture Template
> Tổng hợp từ kinh nghiệm thực chiến — áp dụng được cho mọi dự án Unity mới.
> Đây là nguồn sự thật duy nhất về cấu trúc, convention, và quyết định kiến trúc.

Unity Version Target: 2022.3.62f2
---

## Mục lục

1. [Cấu trúc thư mục](#1-cấu-trúc-thư-mục)
2. [Bootstrap & Service Locator](#2-bootstrap--service-locator)
3. [ScriptableObject Configs](#3-scriptableobject-configs)
4. [Event System](#4-event-system)
5. [Booster / Command Pattern](#5-booster--command-pattern)
6. [Tracking / Analytics](#6-tracking--analytics)
7. [Obstacle / Registry Pattern](#7-obstacle--registry-pattern)
8. [Code Standards](#8-code-standards)
9. [Branch Strategy](#9-branch-strategy)
10. [Refactor Checklist Template](#10-refactor-checklist-template)
11. [Những thứ KHÔNG làm](#11-những-thứ-không-làm)

---

## 1. Cấu trúc thư mục

```
Assets/
├── Scripts/                        ← TẤT CẢ CODE MỚI ĐI VÀO ĐÂY
│   ├── Bootstrap/                  GameContext (singleton, init orchestrator)
│   ├── Services/                   Các service (Audio, UserData, UI, ...)
│   │   ├── Audio/
│   │   ├── UserData/
│   │   └── ...
│   ├── GamePlay/                   Code gameplay đã refactor
│   │   ├── Controller/
│   │   ├── Element/
│   │   └── ...
│   ├── Config/                     ScriptableObject config classes
│   ├── Events/                     GameEventService
│   ├── Analytics/                  AnalyticsService, ITrackingEvent, event classes
│   ├── Booster/                    BoosterCommand, BoosterService
│   ├── Obstacles/                  IObstacleElement, ObstacleRegistry, ObstacleDefinition
│   └── Utils.cs                    ← MỌI utility/helper function đều vào đây (1 file duy nhất)
│
└── _PROJECT/Scripts/               ← CODE CŨ — không đụng vào, xóa dần theo từng phase
    └── ...
```

**Nguyên tắc:**
- Code mới hoàn toàn tách biệt khỏi code cũ.
- Không copy code cũ sang rồi vá víu — viết mới từ đầu, dùng code cũ làm tham chiếu.
- Xóa code cũ từng file, chỉ khi replacement đã chạy ổn định.

---

## 2. Bootstrap & Service Locator

### Quyết định kiến trúc
- **Chỉ một singleton duy nhất:** `GameContext` — MonoBehaviour, `DontDestroyOnLoad`.
- Tất cả service được đăng ký qua một ServiceLocator.
- Không dùng `.instance` trên từng service riêng lẻ.
- Truy cập service ở bất kỳ đâu: `ServiceLocator.Get<AudioService>()`.

### Template: GameContext.cs

```csharp
// Assets/Scripts/Bootstrap/GameContext.cs
namespace YourGame.Bootstrap
{
    public class GameContext : MonoBehaviour
    {
        #region Properties
        public static GameContext Instance { get; private set; }

        /// <summary>Settings SO chứa tất cả config references.</summary>
        [field: SerializeField] public GameContextSettings Settings { get; private set; }
        #endregion

        #region Unity Callbacks
        private void Awake()
        {
            if (Instance != null) { Destroy(gameObject); return; }
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }

        private void Start()
        {
            RegisterServices();
            BeginInitialization();
        }
        #endregion

        #region Private Methods
        private void RegisterServices()
        {
            // Đăng ký theo thứ tự dependency — service nào phụ thuộc service khác thì đăng ký sau
            ServiceLocator.Add(new GameEventService());
            ServiceLocator.Add(new AudioService(Settings.AudioConfig));
            ServiceLocator.Add(new UserDataService());
            ServiceLocator.Add(new UIService());
            // ... tiếp tục
        }

        private void BeginInitialization()
        {
            // Async init nếu cần (SDK callbacks, remote config, ...)
        }
        #endregion
    }
}
```

### Template: ServiceLocator.cs (nếu tự build)

```csharp
// Assets/Scripts/Bootstrap/ServiceLocator.cs
namespace YourGame.Bootstrap
{
    public static class ServiceLocator
    {
        private static readonly Dictionary<Type, object> _services = new();

        /// <summary>Đăng ký service. Chỉ gọi từ GameContext.RegisterServices().</summary>
        public static void Add<T>(T service) where T : class
        {
            _services[typeof(T)] = service;
        }

        /// <summary>Lấy service. Throws nếu chưa đăng ký.</summary>
        public static T Get<T>() where T : class
        {
            if (_services.TryGetValue(typeof(T), out var service))
                return (T)service;
            throw new KeyNotFoundException($"Service {typeof(T).Name} chưa được đăng ký.");
        }
    }
}
```

### Lấy service trong code game

```csharp
// Inject qua constructor (khuyến nghị)
public class SomeController : MonoBehaviour
{
    private AudioService _audio;
    private GameEventService _events;

    private void Start()
    {
        _audio  = ServiceLocator.Get<AudioService>();
        _events = ServiceLocator.Get<GameEventService>();
    }
}
```

---

## 3. ScriptableObject Configs

### Quyết định kiến trúc
- Mỗi service có một config SO riêng.
- **Không** dùng serialized field trên MonoBehaviour — tất cả config thuộc về SO.
- SO có thể map sang RemoteConfig key (string field tùy chọn).
- `GameContextSettings` là SO trung tâm, chứa references đến tất cả config SO khác.

### Pattern SO

```csharp
// Assets/Scripts/Config/AudioConfigSO.cs
namespace YourGame.Config
{
    [CreateAssetMenu(menuName = "Config/Audio")]
    public class AudioConfigSO : ScriptableObject
    {
        #region Fields
        [Header("Volumes")]
        public float defaultMusicVolume = 0.8f;
        public float defaultSfxVolume   = 1f;

        [Header("Remote Config Keys — để trống nếu không dùng")]
        public string remoteKey_defaultMusicVolume;
        public string remoteKey_defaultSfxVolume;
        #endregion
    }
}
```

### GameContextSettings — SO trung tâm

```csharp
// Assets/Scripts/Bootstrap/GameContextSettings.cs
namespace YourGame.Bootstrap
{
    [CreateAssetMenu(menuName = "Config/GameContextSettings")]
    public class GameContextSettings : ScriptableObject
    {
        #region Properties
        [field: SerializeField] public AudioConfigSO    AudioConfig    { get; private set; }
        [field: SerializeField] public GridConfigSO     GridConfig     { get; private set; }
        [field: SerializeField] public BoosterConfigSO  BoosterConfig  { get; private set; }
        [field: SerializeField] public PrefabReferencesSO Prefabs      { get; private set; }
        // ... thêm các SO khác
        #endregion
    }
}
```

### PrefabReferencesSO

```csharp
// Assets/Scripts/Config/PrefabReferencesSO.cs
namespace YourGame.Config
{
    [CreateAssetMenu(menuName = "Config/PrefabReferences")]
    public class PrefabReferencesSO : ScriptableObject
    {
        #region Properties
        // Đặt tên PascalCase, dùng [field: SerializeField]
        [field: SerializeField] public GameObject BasketElement  { get; private set; }
        [field: SerializeField] public GameObject WallElement    { get; private set; }
        // ... tiếp tục
        #endregion
    }
}

// Truy cập:
// GameContext.Instance.Settings.Prefabs.BasketElement
```

---

## 4. Event System

### Quyết định kiến trúc
- `GameEventService` là plain class, không có interface.
- Mọi delegate trở thành public property (PascalCase, action-oriented).
- Không dùng `static` event bus.
- `GameStateManager` là state machine riêng biệt — không gộp vào event system.

### Template: GameEventService.cs

```csharp
// Assets/Scripts/Events/GameEventService.cs
namespace YourGame.Events
{
    public class GameEventService
    {
        #region Gameplay Events
        /// <summary>Fired khi player chọn một ô trên lưới.</summary>
        public Action<SlotOnElement> OnCellChoose { get; set; }

        /// <summary>Fired khi level hoàn thành.</summary>
        public Action OnLevelComplete { get; set; }

        /// <summary>Fired khi player thua.</summary>
        public Action OnLevelFail { get; set; }
        #endregion

        #region UI Events
        public Action<int> OnCoinChanged  { get; set; }
        public Action<int> OnLivesChanged { get; set; }
        #endregion
    }
}
```

### Cách dùng

```csharp
// Subscribe
_events.OnLevelComplete += HandleLevelComplete;

// Unsubscribe (luôn luôn unsubscribe khi destroy)
private void OnDestroy()
{
    _events.OnLevelComplete -= HandleLevelComplete;
}

// Fire
_events.OnLevelComplete?.Invoke();
```

---

## 5. Booster / Command Pattern

### Quyết định kiến trúc
- Tất cả booster logic sống trong `BoosterService` + từng `BoosterCommand`.
- Controller chỉ gọi `BoosterService.Execute(type, context)` — không biết gì về logic.
- Thêm booster mới = thêm một `BoosterCommand` subclass, không đụng gì code hiện tại.

### Template

```csharp
// Assets/Scripts/Booster/BoosterCommand.cs
namespace YourGame.Booster
{
    public abstract class BoosterCommand
    {
        /// <summary>Thực thi booster. Override trong từng subclass.</summary>
        public abstract void Execute(BoosterContext context);
    }
}

// Assets/Scripts/Booster/MagnetBoosterCommand.cs
namespace YourGame.Booster
{
    public class MagnetBoosterCommand : BoosterCommand
    {
        public override void Execute(BoosterContext context)
        {
            // Logic hút cát
        }
    }
}

// Assets/Scripts/Booster/BoosterService.cs
namespace YourGame.Booster
{
    public class BoosterService
    {
        #region Fields
        private readonly Dictionary<BoosterType, BoosterCommand> _commands = new()
        {
            { BoosterType.Magnet, new MagnetBoosterCommand() },
            { BoosterType.Undo,   new UndoBoosterCommand()   },
            // ...
        };
        #endregion

        #region Public Methods
        /// <summary>Thực thi booster theo type.</summary>
        public void Execute(BoosterType type, BoosterContext context)
        {
            if (_commands.TryGetValue(type, out var command))
                command.Execute(context);
            else
                Debug.LogError($"Booster {type} chưa được đăng ký.");
        }
        #endregion
    }
}
```

---

## 6. Tracking / Analytics

### Quyết định kiến trúc
- Chỉ `AnalyticsService` được gọi SDK tracking trực tiếp.
- Mọi class khác chỉ gọi `AnalyticsService.Track(ITrackingEvent)`.
- Tên event là constants, không dùng inline string literal.

### Template

```csharp
// Assets/Scripts/Analytics/ITrackingEvent.cs
namespace YourGame.Analytics
{
    public interface ITrackingEvent
    {
        string EventName { get; }
        Dictionary<string, object> Properties { get; }
    }
}

// Assets/Scripts/Analytics/Events/LevelCompleteEvent.cs
namespace YourGame.Analytics
{
    public class LevelCompleteEvent : ITrackingEvent
    {
        private const string EVENT_NAME = "level_complete"; // constant, không inline

        public string EventName => EVENT_NAME;
        public Dictionary<string, object> Properties { get; }

        public LevelCompleteEvent(int level, int moves, float duration)
        {
            Properties = new Dictionary<string, object>
            {
                { "level",    level    },
                { "moves",    moves    },
                { "duration", duration }
            };
        }
    }
}

// Assets/Scripts/Analytics/AnalyticsService.cs
namespace YourGame.Analytics
{
    public class AnalyticsService
    {
        #region Public Methods
        /// <summary>Gửi event tracking. Đây là điểm duy nhất gọi SDK.</summary>
        public void Track(ITrackingEvent trackingEvent)
        {
            // YourSDK.TrackEvent(trackingEvent.EventName, trackingEvent.Properties);
        }
        #endregion
    }
}

// Cách dùng ở bất kỳ đâu:
_analytics.Track(new LevelCompleteEvent(level: 5, moves: 30, duration: 120f));
```

---

## 7. Obstacle / Registry Pattern

Dùng khi game có nhiều loại object/element cần spawn động và muốn thêm loại mới mà **không sửa code cũ**.

### Template: Interface

```csharp
// Assets/Scripts/Obstacles/IObstacleElement.cs
namespace YourGame.Obstacles
{
    public interface IObstacleElement
    {
        /// <summary>Setup ban đầu từ cell data.</summary>
        void SpawnFromCellData(CellData cell, GridCell gridCell);

        /// <summary>Gọi sau khi toàn bộ grid đã init xong — dùng để cross-reference.</summary>
        void OnPostInit(SlotsController controller);

        /// <summary>True nếu obstacle chiếm slot trên grid.</summary>
        bool UsesCellSlot { get; }
    }
}
```

### Template: ObstacleDefinition SO

```csharp
// Assets/Scripts/Obstacles/ObstacleDefinition.cs
namespace YourGame.Obstacles
{
    [CreateAssetMenu(menuName = "Obstacles/ObstacleDefinition")]
    public class ObstacleDefinition : ScriptableObject
    {
        #region Fields
        public ElementType type;
        public GameObject  prefab;
        public Texture2D   editorIcon;
        public bool        usesCellSlot;
        #endregion
    }
}
```

### Template: ObstacleRegistry SO

```csharp
// Assets/Scripts/Obstacles/ObstacleRegistry.cs
namespace YourGame.Obstacles
{
    [CreateAssetMenu(menuName = "Obstacles/ObstacleRegistry")]
    public class ObstacleRegistry : ScriptableObject
    {
        #region Fields
        [SerializeField] private List<ObstacleDefinition> obstacles = new();

        private Dictionary<ElementType, ObstacleDefinition> _cache;
        #endregion

        #region Public Methods
        public ObstacleDefinition Get(ElementType type)
        {
            BuildCacheIfNeeded();
            _cache.TryGetValue(type, out var def);
            return def;
        }

        public bool Has(ElementType type)
        {
            BuildCacheIfNeeded();
            return _cache.ContainsKey(type);
        }
        #endregion

        #region Private Methods
        private void BuildCacheIfNeeded()
        {
            if (_cache != null) return;
            _cache = obstacles.ToDictionary(d => d.type);
        }
        #endregion
    }
}
```

### Checklist thêm obstacle mới (sau khi hệ thống đã live)

1. Thêm giá trị vào enum `ElementType` — **không bao giờ** dùng lại hoặc đánh số lại giá trị cũ.
2. Tạo `[Name]Element.cs` implement `IObstacleElement`.
3. Tạo prefab trong Unity.
4. Tạo asset `ObstacleDefinition`, điền đủ type / prefab / icon.
5. Add definition vào asset `ObstacleRegistry` trong Inspector.

**Xong. Không file nào khác bị sửa.**

---

## 8. Code Standards

### Namespaces — bắt buộc

```csharp
// File: Assets/Scripts/Services/Audio/AudioService.cs
namespace YourGame.Services.Audio { ... }

// File: Assets/Scripts/GamePlay/Element/Basket/BasketElement.cs
namespace YourGame.GamePlay.Element.Basket { ... }
```

Namespace phải khớp với cấu trúc thư mục.

### Regions — bắt buộc

```csharp
public class FooService
{
    #region Fields
    // private fields
    #endregion

    #region Properties
    // public/protected properties
    #endregion

    #region Unity Callbacks
    // Awake, Start, OnDestroy, ...
    #endregion

    #region Public Methods
    // public API
    #endregion

    #region Private/Protected Methods
    // internal logic
    #endregion
}
```

Chỉ include region nào có nội dung.

### Naming Convention

| Loại | Convention | Ví dụ |
|---|---|---|
| Class / Interface / SO | PascalCase | `AudioService`, `ITrackingEvent` |
| Public property | PascalCase | `public float Volume { get; set; }` |
| Private field | _camelCase | `private float _volume;` |
| Constant | UPPER_SNAKE | `private const string EVENT_NAME = "level_complete";` |
| Method | PascalCase | `public void PlaySound()` |
| Event property | PascalCase, action-oriented | `OnLevelComplete`, `OnCellChoose` |

### Comments

```csharp
// ✅ Comment giải thích TẠI SAO, không giải thích CÁI GÌ
// Delay 1 frame để đảm bảo tất cả Awake() đã chạy xong trước khi đăng ký event
yield return null;

// ❌ Comment thừa, chỉ đọc lại code
// Gán giá trị volume cho biến _volume
_volume = value;
```

- Tất cả comment bằng **tiếng Anh**.
- Mọi public method, property, và field quan trọng cần có XML doc comment (`/// <summary>`).
- Không commit code bị comment out.

### Giới hạn độ phức tạp

- Method dài hơn **~30 dòng** → đang làm quá nhiều việc, hãy tách ra.
- Ưu tiên code phẳng, rõ ràng hơn code thông minh, lồng nhau nhiều tầng.
- Không over-engineer. Giải quyết bài toán hiện tại, không phải bài toán tưởng tượng.
- Interface chỉ đáng tạo khi có **hai implementation thực sự tồn tại**.

### Utils

```csharp
// Assets/Scripts/Utils.cs — MỘT file duy nhất, MỘT class static duy nhất
namespace YourGame
{
    public static class Utils
    {
        /// <summary>Clamp giá trị về range [0, 1].</summary>
        public static float Clamp01(float value) => Mathf.Clamp(value, 0f, 1f);

        // Tất cả helper functions đều vào đây
        // Không tạo thêm utility class riêng lẻ
    }
}
```

---

## 9. Branch Strategy

```
main              ← production, luôn ổn định
dev               ← active feature development
dev-refactor      ← refactor song song với dev (nếu cần)
```

**Khi refactor lớn:**
- Tạo branch `dev-refactor` chạy song song với `dev`.
- Bugfix trên `dev` phải được port thủ công sang `dev-refactor` trong thời gian cả hai cùng active.
- Giữ phase ngắn để giảm divergence.
- Khi `dev-refactor` ổn định → replace `dev` → merge vào `main`.

---

## 10. Refactor Checklist Template

Dùng template này khi bắt đầu refactor một dự án mới.

```markdown
## Phase 1 — Bootstrap & Service Locator
- [ ] Tạo GameContext MonoBehaviour
- [ ] Tạo GameContextSettings SO
- [ ] Implement ServiceLocator
- [ ] Đăng ký tất cả service trong GameContext.Start()
- [ ] Validate: game khởi động và chạy đến trạng thái Playing

## Phase 2 — ScriptableObject Configs
- [ ] Xác định tất cả singleton/MonoBehaviour đang giữ config
- [ ] Tạo SO tương ứng cho từng nhóm config
- [ ] Migrate từng field, test sau mỗi field
- [ ] Xóa MonoBehaviour cũ sau khi SO đã hoạt động

## Phase 3 — Event System
- [ ] Implement GameEventService
- [ ] Đăng ký trong ServiceLocator
- [ ] Migrate từng static event delegate sang property
- [ ] Xóa static event bus cũ

## Phase 4 — Singleton Removal
- [ ] Liệt kê tất cả singleton còn lại
- [ ] Tạo Service tương ứng cho từng singleton
- [ ] Migrate từng call-site
- [ ] Xóa singleton sau khi service hoạt động ổn

## Phase 5+ — Feature-Specific Refactors
- [ ] [Tùy dự án]

## Phase N — Post-Refactor Cleanup
- [ ] Audit toàn bộ // TODO: comment
- [ ] Audit naming convention
- [ ] Xóa unused using directives
- [ ] Xóa dead code
- [ ] Final test toàn bộ game
```

---

## 11. Những thứ KHÔNG làm

| Đừng | Làm thay vào đó |
|---|---|
| Copy code cũ sang, patch lại | Viết mới từ đầu, dùng code cũ làm tham chiếu |
| Dùng `.instance` trên từng service | `ServiceLocator.Get<T>()` |
| Serialized field trên MonoBehaviour | Dùng ScriptableObject |
| Interface khi chỉ có một implementation | Viết class thẳng, interface sau khi cần |
| Inline string cho event/tracking name | Dùng `const string` |
| Nhiều utility file/class | Một file `Utils.cs`, một class `Utils` |
| Comment giải thích code làm gì | Comment giải thích tại sao |
| Method dài hơn 30 dòng | Tách method |
| Code cũ bị comment out trong commit | Xóa hẳn |
| Rename/refactor framework bên thứ ba | Để nguyên, hệ thống mới sẽ tự ngưng dùng |
| Sửa enum value đang được dùng trong data | Chỉ thêm mới, không bao giờ sửa/xóa |
| Xóa file cũ trước khi replacement ổn định | Xóa sau khi đã test ổn |

---

## Tham chiếu nhanh — Service Registration Order

```csharp
// Thứ tự khuyến nghị: service nào không phụ thuộc gì → đăng ký trước
ServiceLocator.Add(new GameEventService());      // 1. Event bus — không phụ thuộc gì
ServiceLocator.Add(new UserDataService());       // 2. Data — không phụ thuộc gì
ServiceLocator.Add(new AudioService(config));    // 3. Audio — có thể phụ thuộc config
ServiceLocator.Add(new UIService());             // 4. UI — có thể fire events
ServiceLocator.Add(new BoosterService());        // 5. Booster — dùng events và user data
ServiceLocator.Add(new AnalyticsService());      // 6. Analytics — dùng sau các service khác
ServiceLocator.Add(new GameSessionService());    // 7. Session — orchestrate các service khác
```

---

*Template này được tổng hợp từ kinh nghiệm thực chiến. Cập nhật khi có quyết định kiến trúc mới.*
