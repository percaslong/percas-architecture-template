# Unity Project Architecture Template
> Kiến trúc chuẩn cho mọi dự án Unity mới tại Percas Studio.
> Đây là nguồn sự thật duy nhất về cấu trúc, convention, và quyết định kiến trúc.
> Tất cả dự án mới **bắt buộc** tuân theo template này từ ngày đầu tiên.

Unity Version Target: 2022.3.62f2

| Version | Ngày | Thay đổi |
|---|---|---|
| v3.0 | 2026-04-13 | Chuyển sang kiến trúc cho dự án mới (bỏ legacy/refactor framing). UniTask cài qua GitHub Release. |
| v2.0 | 2026-04-13 | Thêm Quick Start, Dependency Diagram, Service Lifecycle, UniTask Convention, Object Pooling, UI Architecture, Save/Load (JsonConvert), Error Handling, Testing & Debug. |
| v1.0 | — | Bản gốc. |
---

## Mục lục

1. [Quick Start — Từ 0 đến game chạy](#1-quick-start--từ-0-đến-game-chạy)
2. [Dependency Diagram](#2-dependency-diagram)
3. [Cấu trúc thư mục](#3-cấu-trúc-thư-mục)
4. [Bootstrap & Service Locator](#4-bootstrap--service-locator)
5. [Service Lifecycle — Init / Dispose](#5-service-lifecycle--init--dispose)
6. [Async Convention — UniTask](#6-async-convention--unitask)
7. [ScriptableObject Configs](#7-scriptableobject-configs)
8. [Event System](#8-event-system)
9. [Booster / Command Pattern](#9-booster--command-pattern)
10. [Tracking / Analytics](#10-tracking--analytics)
11. [Obstacle / Registry Pattern](#11-obstacle--registry-pattern)
12. [Object Pooling](#12-object-pooling)
13. [UI Architecture](#13-ui-architecture)
14. [Save / Load — Data Persistence](#14-save--load--data-persistence)
15. [Error Handling & Defensive Coding](#15-error-handling--defensive-coding)
16. [Testing & Debug](#16-testing--debug)
17. [Code Standards](#17-code-standards)
18. [Branch Strategy](#18-branch-strategy)
19. [Project Setup Checklist](#19-project-setup-checklist)
20. [Những thứ KHÔNG làm](#20-những-thứ-không-làm)

---

## 1. Quick Start — Từ 0 đến game chạy

> Dành cho người mới vào team hoặc bắt đầu dự án mới. Làm đúng 5 bước này, bấm Play là game chạy.

### Bước 1 — Tạo Scene `Bootstrap`

1. File → New Scene → Save as `Bootstrap.scene`.
2. Đặt scene này **đầu tiên** trong Build Settings (index 0).
3. Tạo empty GameObject tên `GameContext`, attach script `GameContext.cs`.

### Bước 2 — Tạo Config SOs

1. Project window → Create → Config → `GameContextSettings`.
2. Tạo các SO con: `AudioConfigSO`, `GridConfigSO`, `BoosterConfigSO`, `PrefabReferencesSO`, ...
3. Kéo các SO con vào slot tương ứng trên `GameContextSettings`.
4. Kéo `GameContextSettings` vào slot `Settings` trên `GameContext` Inspector.

### Bước 3 — Verify ServiceLocator

1. Bấm Play.
2. Mở Console — không có `KeyNotFoundException` = OK.
3. Thử gọi `ServiceLocator.Get<GameEventService>()` từ bất kỳ script nào → nhận được instance = OK.

### Bước 4 — Tạo Scene `Game`

1. Tạo scene gameplay chính.
2. Trong `GameContext.BeginInitialization()`, load scene này bằng `SceneManager.LoadSceneAsync`.
3. `GameContext` sống ở scene `Bootstrap` với `DontDestroyOnLoad` — các scene khác chỉ chứa gameplay.

### Bước 5 — Kiểm tra flow

```
[Play] → Bootstrap scene load
       → GameContext.Awake() — singleton init
       → GameContext.Start() — RegisterServices() → BeginInitialization()
       → Game scene load
       → Gameplay scripts gọi ServiceLocator.Get<T>() → OK
```

**Nếu gặp lỗi:** xem [Error Handling & Defensive Coding](#15-error-handling--defensive-coding).

---

## 2. Dependency Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    Bootstrap Scene                       │
│  ┌───────────────────────────────────────────────────┐  │
│  │              GameContext (Singleton)               │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │           GameContextSettings (SO)          │  │  │
│  │  │  ┌──────────┐ ┌──────────┐ ┌────────────┐  │  │  │
│  │  │  │AudioConf │ │GridConf  │ │BoosterConf │  │  │  │
│  │  │  └──────────┘ └──────────┘ └────────────┘  │  │  │
│  │  │  ┌──────────────┐ ┌─────────────────────┐  │  │  │
│  │  │  │PrefabRefs    │ │ ...other configs    │  │  │  │
│  │  │  └──────────────┘ └─────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │                      │                            │  │
│  │                      ▼                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │         ServiceLocator (Static)             │  │  │
│  │  │  ┌────────────┐  ┌──────────────────────┐   │  │  │
│  │  │  │ GameEvent   │  │ UserDataService     │   │  │  │
│  │  │  │ Service     │  │ (JsonConvert+File)  │   │  │  │
│  │  │  └──────┬─────┘  └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ AudioService         │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ UIService            │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ BoosterService       │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ AnalyticsService     │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         └────────▶│ GameSessionService   │   │  │  │
│  │  │                   └──────────────────────┘   │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
         │
         ▼ LoadSceneAsync
┌─────────────────────────────────────────────────────────┐
│                     Game Scene                          │
│  Controllers / Elements / UI                            │
│  → Tất cả gọi ServiceLocator.Get<T>() để lấy service   │
└─────────────────────────────────────────────────────────┘
```

**Mũi tên `──▶` = "subscribe / dùng event từ"**. GameEventService là trung tâm, các service khác subscribe event từ nó.

---

## 3. Cấu trúc thư mục

```
Assets/
├── Scripts/                        ← TẤT CẢ CODE MỚI ĐI VÀO ĐÂY
│   ├── Bootstrap/                  GameContext, ServiceLocator, IInitializable
│   ├── Services/                   Các service (Audio, UserData, UI, ...)
│   │   ├── Audio/
│   │   ├── UserData/               UserDataService, SaveData model
│   │   ├── UI/                     UIService, BaseScreen, BasePopup
│   │   └── ...
│   ├── GamePlay/                   Code gameplay chính
│   │   ├── Controller/
│   │   ├── Element/
│   │   └── ...
│   ├── Config/                     ScriptableObject config classes
│   ├── Events/                     GameEventService
│   ├── Analytics/                  AnalyticsService, ITrackingEvent, event classes
│   ├── Booster/                    BoosterCommand, BoosterService
│   ├── Obstacles/                  IObstacleElement, ObstacleRegistry, ObstacleDefinition
│   ├── Pool/                       ObjectPool<T>, PoolManager
│   └── Utils.cs                    ← MỌI utility/helper function đều vào đây (1 file duy nhất)
│
├── Plugins/                        ← Package .unitypackage import vào đây (UniTask, DOTween, ...)
│
└── Resources/                      ← Chỉ dùng cho asset cần load động qua Resources.Load (hạn chế tối đa)
```

**Nguyên tắc:**
- Tất cả code nằm trong `Assets/Scripts/` — không tạo thư mục script ở nơi khác.
- Thư mục phải khớp với namespace (xem [Code Standards](#17-code-standards)).
- Không tạo thư mục rỗng — chỉ tạo khi có file đầu tiên cần đặt vào.

---

## 4. Bootstrap & Service Locator

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
// Lấy service trong bất kỳ MonoBehaviour nào
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

## 5. Service Lifecycle — Init / Dispose

### Quyết định kiến trúc
- Service nào cần init async (load data, chờ SDK) → implement `IInitializable`.
- Service nào cần cleanup (save data, flush analytics) → implement `System.IDisposable`.
- `GameContext` gọi init theo thứ tự dependency, gọi dispose khi `OnApplicationQuit`.

### Template: IInitializable.cs

```csharp
// Assets/Scripts/Bootstrap/IInitializable.cs
namespace YourGame.Bootstrap
{
    /// <summary>Service cần init async implement interface này.</summary>
    public interface IInitializable
    {
        /// <summary>Init async. GameContext gọi theo thứ tự đăng ký.</summary>
        UniTask InitializeAsync();
    }
}
```

### Template: ServiceLocator mở rộng

```csharp
// Assets/Scripts/Bootstrap/ServiceLocator.cs
namespace YourGame.Bootstrap
{
    public static class ServiceLocator
    {
        private static readonly Dictionary<Type, object> _services = new();
        private static readonly List<object> _registrationOrder = new();

        public static void Add<T>(T service) where T : class
        {
            _services[typeof(T)] = service;
            _registrationOrder.Add(service);
        }

        public static T Get<T>() where T : class
        {
            if (_services.TryGetValue(typeof(T), out var service))
                return (T)service;
            throw new KeyNotFoundException($"Service {typeof(T).Name} chưa được đăng ký.");
        }

        /// <summary>Gọi InitializeAsync trên tất cả service có IInitializable, theo thứ tự đăng ký.</summary>
        public static async UniTask InitializeAllAsync()
        {
            foreach (var service in _registrationOrder)
            {
                if (service is IInitializable initializable)
                    await initializable.InitializeAsync();
            }
        }

        /// <summary>Gọi Dispose trên tất cả service có IDisposable, theo thứ tự ngược.</summary>
        public static void DisposeAll()
        {
            for (int i = _registrationOrder.Count - 1; i >= 0; i--)
            {
                if (_registrationOrder[i] is System.IDisposable disposable)
                {
                    try { disposable.Dispose(); }
                    catch (System.Exception e) { Debug.LogError($"Dispose error: {e}"); }
                }
            }
            _services.Clear();
            _registrationOrder.Clear();
        }
    }
}
```

### GameContext cập nhật

```csharp
// Assets/Scripts/Bootstrap/GameContext.cs
private async void Start()
{
    RegisterServices();
    await ServiceLocator.InitializeAllAsync();
    // Tất cả service đã sẵn sàng — load game scene
    await SceneManager.LoadSceneAsync("Game", LoadSceneMode.Additive).ToUniTask();
}

private void OnApplicationQuit()
{
    ServiceLocator.DisposeAll();
}
```

### Ví dụ: Service vừa init vừa dispose

```csharp
public class UserDataService : IInitializable, System.IDisposable
{
    public async UniTask InitializeAsync()
    {
        // Load save file từ disk
        await LoadAsync();
    }

    public void Dispose()
    {
        // Save data trước khi game tắt
        Save();
    }
}
```

**Nguyên tắc:**
- Không bắt buộc service nào cũng implement — chỉ service nào **cần** thì mới implement.
- Init chạy **tuần tự** theo thứ tự đăng ký — đảm bảo dependency order.
- Dispose chạy **ngược** — service đăng ký sau (phụ thuộc nhiều nhất) cleanup trước.

---

## 6. Async Convention — UniTask

### Quyết định kiến trúc
- **Dùng UniTask cho tất cả async** — không dùng Coroutine, không dùng `System.Threading.Tasks.Task`.
- **Không dùng Coroutine** — kể cả cho logic đơn giản. UniTask cover toàn bộ use case.
- **Tuyệt đối không** dùng `async void` — trừ Unity callbacks (`Start`, `OnDestroy`, ...).

### Package Setup

1. Vào [https://github.com/Cysharp/UniTask/releases](https://github.com/Cysharp/UniTask/releases).
2. Tải file `.unitypackage` của **version mới nhất**.
3. Trong Unity: Assets → Import Package → Custom Package → chọn file vừa tải.
4. Import toàn bộ → UniTask sẽ nằm trong `Assets/Plugins/UniTask/`.

> **Lưu ý:** Không dùng `Add package by name` vì UniTask không nằm trên Unity Registry chính thức.
> Khi cần update: tải `.unitypackage` mới nhất từ GitHub, import đè lên.

### Quy tắc cụ thể

| Tình huống | Cách làm | Ví dụ |
|---|---|---|
| Method async trả về giá trị | `UniTask<T>` | `async UniTask<int> LoadLevel()` |
| Method async không trả về | `UniTask` | `async UniTask ShowPopup()` |
| Unity callback cần async | `async void` + try/catch | `async void Start()` |
| Chờ một frame | `await UniTask.Yield()` | Thay `yield return null` |
| Chờ thời gian | `await UniTask.Delay(ms)` | Thay `yield return new WaitForSeconds` |
| Chờ điều kiện | `await UniTask.WaitUntil(() => condition)` | Thay `yield return new WaitUntil` |
| Hủy task khi destroy | `CancellationToken` từ `this.GetCancellationTokenOnDestroy()` | Xem template bên dưới |
| Load scene | `.ToUniTask()` | `SceneManager.LoadSceneAsync("X").ToUniTask()` |

### Template: Async trong MonoBehaviour

```csharp
public class LevelLoader : MonoBehaviour
{
    #region Unity Callbacks
    private async void Start()
    {
        // async void CHỈ dùng ở Unity callback — luôn wrap try/catch
        try
        {
            await LoadLevelAsync(this.GetCancellationTokenOnDestroy());
        }
        catch (OperationCanceledException)
        {
            // GameObject bị destroy giữa chừng — bình thường, không log error
        }
        catch (System.Exception e)
        {
            Debug.LogError($"[LevelLoader] Init failed: {e}");
        }
    }
    #endregion

    #region Private Methods
    private async UniTask LoadLevelAsync(CancellationToken ct)
    {
        var data = await LoadDataFromDisk(ct);
        await UniTask.Delay(500, cancellationToken: ct); // fade-in delay
        BuildLevel(data);
    }
    #endregion
}
```

### Template: Async trong Service (non-MonoBehaviour)

```csharp
public class RemoteConfigService : IInitializable
{
    #region Fields
    private Dictionary<string, string> _config;
    #endregion

    #region IInitializable
    public async UniTask InitializeAsync()
    {
        // Service không có GetCancellationTokenOnDestroy()
        // Dùng timeout thay vì CancellationToken nếu cần
        var (isCanceled, result) = await FetchRemoteConfig()
            .TimeoutWithoutException(TimeSpan.FromSeconds(10));

        _config = isCanceled
            ? LoadDefaultConfig()   // Timeout → dùng config mặc định
            : result;
    }
    #endregion
}
```

### Những thứ KHÔNG làm với async

| Đừng | Làm thay vào đó |
|---|---|
| `async void` ở method thường | `async UniTask` |
| `StartCoroutine(DoSomething())` | `DoSomethingAsync().Forget()` — Coroutine bị cấm |
| `yield return new WaitForSeconds(1f)` | `await UniTask.Delay(1000, cancellationToken: ct)` — không dùng yield |
| Quên CancellationToken trong MonoBehaviour | Luôn dùng `this.GetCancellationTokenOnDestroy()` |
| `Task.Run(() => ...)` | UniTask đã chạy trên main thread, không cần thread pool |
| Fire-and-forget không kiểm soát | `.Forget()` có ý thức, hoặc `try/catch` trong `async void` |

---

## 7. ScriptableObject Configs

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

## 8. Event System

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

### ⚠️ Event Safety — Tránh Memory Leak

Delegate-based event system là nguồn memory leak **phổ biến nhất**. Tuân thủ checklist này:

**Checklist bắt buộc khi subscribe event:**

1. **Subscribe ở `Start()` hoặc `OnEnable()`** — không subscribe ở `Awake()` (service có thể chưa sẵn sàng).
2. **Unsubscribe ở `OnDestroy()`** — bắt buộc, không ngoại lệ.
3. **Handler là method riêng, không phải lambda** — lambda không unsubscribe được.
4. **Code review phải check:** mỗi `+=` phải có `-=` tương ứng.

```csharp
// ✅ ĐÚNG — dễ unsubscribe
private void Start()
{
    _events = ServiceLocator.Get<GameEventService>();
    _events.OnLevelComplete += HandleLevelComplete;
}

private void OnDestroy()
{
    _events.OnLevelComplete -= HandleLevelComplete;
}

private void HandleLevelComplete() { /* ... */ }

// ❌ SAI — không thể unsubscribe lambda
_events.OnLevelComplete += () => Debug.Log("Done");
```

---

## 9. Booster / Command Pattern

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

## 10. Tracking / Analytics

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

## 11. Obstacle / Registry Pattern

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

## 12. Object Pooling

### Quyết định kiến trúc
- Bất kỳ object nào spawn/destroy **nhiều hơn 10 lần/giây** → dùng pool.
- Dùng `ObjectPool<T>` generic — một pool cho mỗi prefab type.
- `PoolManager` là service, đăng ký trong `ServiceLocator`.
- Object được pool **phải** implement `IPoolable` để reset state khi trả về pool.

### Template: IPoolable

```csharp
// Assets/Scripts/Pool/IPoolable.cs
namespace YourGame.Pool
{
    public interface IPoolable
    {
        /// <summary>Reset state về mặc định khi trả về pool.</summary>
        void OnReturnToPool();

        /// <summary>Setup khi lấy ra từ pool.</summary>
        void OnGetFromPool();
    }
}
```

### Template: ObjectPool

```csharp
// Assets/Scripts/Pool/ObjectPool.cs
namespace YourGame.Pool
{
    public class ObjectPool<T> where T : Component, IPoolable
    {
        #region Fields
        private readonly T _prefab;
        private readonly Transform _parent;
        private readonly Queue<T> _available = new();
        private const int DEFAULT_PREWARM = 10;
        #endregion

        #region Constructor
        public ObjectPool(T prefab, Transform parent, int prewarmCount = DEFAULT_PREWARM)
        {
            _prefab = prefab;
            _parent = parent;
            Prewarm(prewarmCount);
        }
        #endregion

        #region Public Methods
        public T Get(Vector3 position, Quaternion rotation)
        {
            var obj = _available.Count > 0
                ? _available.Dequeue()
                : Object.Instantiate(_prefab, _parent);

            obj.transform.SetPositionAndRotation(position, rotation);
            obj.gameObject.SetActive(true);
            obj.OnGetFromPool();
            return obj;
        }

        public void Return(T obj)
        {
            obj.OnReturnToPool();
            obj.gameObject.SetActive(false);
            _available.Enqueue(obj);
        }
        #endregion

        #region Private Methods
        private void Prewarm(int count)
        {
            for (int i = 0; i < count; i++)
            {
                var obj = Object.Instantiate(_prefab, _parent);
                obj.gameObject.SetActive(false);
                _available.Enqueue(obj);
            }
        }
        #endregion
    }
}
```

### Cách dùng

```csharp
// Tạo pool
var sandPool = new ObjectPool<SandElement>(sandPrefab, poolParent, prewarmCount: 20);

// Lấy ra
var sand = sandPool.Get(spawnPos, Quaternion.identity);

// Trả về (thay vì Destroy)
sandPool.Return(sand);
```

### Khi nào KHÔNG dùng pool
- Object chỉ tạo 1 lần (UI popup, singleton).
- Object có lifecycle phức tạp cần track riêng.
- Prototype phase — chỉ thêm pool khi profiler xác nhận GC spike.

---

## 13. UI Architecture

### Quyết định kiến trúc
- `UIService` quản lý tất cả Screen và Popup — là entry point duy nhất để show/hide UI.
- **Screen** = toàn màn hình, chỉ 1 active tại một thời điểm (Home, Game, Result).
- **Popup** = overlay, có thể stack nhiều popup cùng lúc.
- Show/Hide dùng `CanvasGroup` (fade alpha) — **không** dùng `SetActive` trực tiếp trên root.
- Mỗi screen/popup là 1 prefab riêng, load on-demand.

### Template: BaseScreen

```csharp
// Assets/Scripts/Services/UI/BaseScreen.cs
namespace YourGame.Services.UI
{
    [RequireComponent(typeof(CanvasGroup))]
    public abstract class BaseScreen : MonoBehaviour
    {
        #region Fields
        private CanvasGroup _canvasGroup;
        #endregion

        #region Unity Callbacks
        protected virtual void Awake()
        {
            _canvasGroup = GetComponent<CanvasGroup>();
        }
        #endregion

        #region Public Methods
        /// <summary>Gọi khi screen được show. Override để populate data.</summary>
        public virtual void OnShow()
        {
            _canvasGroup.alpha = 1f;
            _canvasGroup.interactable = true;
            _canvasGroup.blocksRaycasts = true;
        }

        /// <summary>Gọi khi screen bị hide. Override để cleanup.</summary>
        public virtual void OnHide()
        {
            _canvasGroup.alpha = 0f;
            _canvasGroup.interactable = false;
            _canvasGroup.blocksRaycasts = false;
        }
        #endregion
    }
}
```

### Template: BasePopup

```csharp
// Assets/Scripts/Services/UI/BasePopup.cs
namespace YourGame.Services.UI
{
    [RequireComponent(typeof(CanvasGroup))]
    public abstract class BasePopup : MonoBehaviour
    {
        #region Public Methods
        public virtual void OnShow() { /* tương tự BaseScreen */ }
        public virtual void OnHide() { /* tương tự BaseScreen */ }

        /// <summary>Gọi khi user tap nút Close hoặc back.</summary>
        public void Close()
        {
            ServiceLocator.Get<UIService>().ClosePopup(this);
        }
        #endregion
    }
}
```

### Template: UIService

```csharp
// Assets/Scripts/Services/UI/UIService.cs
namespace YourGame.Services.UI
{
    public class UIService
    {
        #region Fields
        private BaseScreen _currentScreen;
        private readonly Stack<BasePopup> _popupStack = new();
        private readonly Dictionary<System.Type, BaseScreen> _screenCache = new();
        private readonly Dictionary<System.Type, BasePopup> _popupCache = new();
        private readonly Transform _screenRoot;
        private readonly Transform _popupRoot;
        private readonly PrefabReferencesSO _prefabs;
        #endregion

        #region Constructor
        public UIService(Transform screenRoot, Transform popupRoot, PrefabReferencesSO prefabs)
        {
            _screenRoot = screenRoot;
            _popupRoot = popupRoot;
            _prefabs = prefabs;
        }
        #endregion

        #region Public Methods
        /// <summary>Chuyển screen. Hide screen cũ, show screen mới.</summary>
        public T ShowScreen<T>() where T : BaseScreen
        {
            _currentScreen?.OnHide();
            var screen = GetOrCreate<T>(_screenCache, _screenRoot);
            screen.OnShow();
            _currentScreen = screen;
            return screen;
        }

        /// <summary>Show popup, push vào stack.</summary>
        public T ShowPopup<T>() where T : BasePopup
        {
            var popup = GetOrCreate<T>(_popupCache, _popupRoot);
            popup.OnShow();
            _popupStack.Push(popup);
            return popup;
        }

        /// <summary>Close popup cụ thể.</summary>
        public void ClosePopup(BasePopup popup)
        {
            popup.OnHide();
            // Rebuild stack nếu popup không phải top — nhưng thường là top
        }
        #endregion

        #region Private Methods
        private T GetOrCreate<T>(Dictionary<System.Type, T> cache, Transform root)
            where T : MonoBehaviour
        {
            if (cache.TryGetValue(typeof(T), out var existing))
                return existing;

            // Quy ước: prefab tên class = tên type
            // Hoặc dùng dictionary mapping trong PrefabReferencesSO
            var prefab = _prefabs; // → cần mapping từ type → prefab, tùy project
            var instance = Object.Instantiate(prefab, root) as T;
            cache[typeof(T)] = instance;
            return instance;
        }
        #endregion
    }
}
```

### Nguyên tắc UI

- UI script **không chứa business logic** — chỉ hiển thị data và fire event.
- Data truyền vào UI qua method `OnShow(data)` — UI không tự đi lấy data.
- Animation/tween dùng DOTween hoặc UniTask delay — không dùng Animation Controller cho UI đơn giản.

---

## 14. Save / Load — Data Persistence

### Quyết định kiến trúc
- **Một file save duy nhất** — `SaveData` class chứa toàn bộ state cần persist.
- Serialize/deserialize bằng `Newtonsoft.Json` (`JsonConvert`).
- Lưu vào `Application.persistentDataPath` — **không** dùng `PlayerPrefs` cho game data.
- `PlayerPrefs` chỉ dùng cho settings đơn giản (âm lượng, ngôn ngữ).
- Auto-save tại các điểm quan trọng (level complete, purchase, ...).

### Package Setup

```
// Package Manager → Add package by name:
com.unity.nuget.newtonsoft-json
```

### Template: SaveData Model

```csharp
// Assets/Scripts/Services/UserData/SaveData.cs
namespace YourGame.Services.UserData
{
    /// <summary>
    /// Toàn bộ data cần persist. Thêm field mới ở đây.
    /// Không bao giờ xóa hoặc rename field đã ship — chỉ thêm mới.
    /// </summary>
    [System.Serializable]
    public class SaveData
    {
        #region Versioning
        /// <summary>Tăng khi thay đổi schema. Dùng để migrate data cũ.</summary>
        public int version = 1;
        #endregion

        #region Player Progress
        public int currentLevel = 1;
        public int totalStars = 0;
        public List<int> completedLevels = new();
        #endregion

        #region Economy
        public int coins = 0;
        public int gems = 0;
        #endregion

        #region Inventory
        public Dictionary<string, int> boosters = new();
        #endregion

        #region Settings
        // Settings đơn giản có thể để ở đây hoặc dùng PlayerPrefs riêng
        public float musicVolume = 0.8f;
        public float sfxVolume = 1f;
        #endregion

        #region Timestamps
        /// <summary>Epoch seconds — dùng server time nếu có, fallback local.</summary>
        public long lastSaveTimestamp;
        #endregion
    }
}
```

### Template: UserDataService

```csharp
// Assets/Scripts/Services/UserData/UserDataService.cs
using Newtonsoft.Json;

namespace YourGame.Services.UserData
{
    public class UserDataService : IInitializable, System.IDisposable
    {
        #region Constants
        private const string SAVE_FILE_NAME = "save.json";
        private const string BACKUP_FILE_NAME = "save.backup.json";
        #endregion

        #region Properties
        /// <summary>Data hiện tại. Đọc/ghi trực tiếp, gọi Save() khi cần persist.</summary>
        public SaveData Data { get; private set; }
        #endregion

        #region IInitializable
        public async UniTask InitializeAsync()
        {
            await LoadAsync();
        }
        #endregion

        #region Public Methods
        /// <summary>Save data xuống disk. Gọi tại các điểm quan trọng.</summary>
        public void Save()
        {
            Data.lastSaveTimestamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds();

            var json = JsonConvert.SerializeObject(Data, Formatting.Indented);
            var path = GetSavePath();

            try
            {
                // Backup file cũ trước khi ghi đè
                if (File.Exists(path))
                    File.Copy(path, GetBackupPath(), overwrite: true);

                File.WriteAllText(path, json);
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[UserDataService] Save failed: {e.Message}");
            }
        }

        /// <summary>Xóa toàn bộ data — dùng cho debug hoặc reset account.</summary>
        public void DeleteAll()
        {
            Data = new SaveData();
            Save();
        }
        #endregion

        #region IDisposable
        public void Dispose()
        {
            Save(); // Auto-save khi game tắt
        }
        #endregion

        #region Private Methods
        private async UniTask LoadAsync()
        {
            var path = GetSavePath();

            if (!File.Exists(path))
            {
                Data = new SaveData();
                return;
            }

            try
            {
                // Đọc file trên background thread để không block main thread
                var json = await UniTask.RunOnThreadPool(() => File.ReadAllText(path));
                Data = JsonConvert.DeserializeObject<SaveData>(json);

                // Migration nếu version cũ
                MigrateIfNeeded();
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[UserDataService] Load failed: {e.Message}. Trying backup...");
                Data = TryLoadBackup() ?? new SaveData();
            }
        }

        private SaveData TryLoadBackup()
        {
            var backupPath = GetBackupPath();
            if (!File.Exists(backupPath)) return null;

            try
            {
                var json = File.ReadAllText(backupPath);
                return JsonConvert.DeserializeObject<SaveData>(json);
            }
            catch
            {
                return null;
            }
        }

        private void MigrateIfNeeded()
        {
            if (Data.version < 2)
            {
                // Ví dụ: version 2 thêm field gems, default = 0
                // Không cần làm gì vì JsonConvert tự dùng default value
                Data.version = 2;
                Save();
            }
            // Thêm migration cho version mới ở đây
        }

        private string GetSavePath()
            => Path.Combine(Application.persistentDataPath, SAVE_FILE_NAME);

        private string GetBackupPath()
            => Path.Combine(Application.persistentDataPath, BACKUP_FILE_NAME);
        #endregion
    }
}

// Cách dùng:
var userData = ServiceLocator.Get<UserDataService>();
userData.Data.coins += 100;
userData.Save();
```

### Schema Migration — Nguyên tắc

- **Không bao giờ xóa field** trong `SaveData` — chỉ thêm mới.
- **Không rename field** — `JsonConvert` sẽ mất data nếu tên thay đổi.
- Nếu bắt buộc phải rename: dùng `[JsonProperty("old_name")]` attribute.
- Khi thêm field mới → tăng `version` → thêm migration block trong `MigrateIfNeeded()`.
- `JsonConvert.DeserializeObject` tự gán default value cho field mới → migration thường chỉ cần set `version`.

### Khi nào gọi Save()

```csharp
// ✅ Gọi Save() tại các milestone
_events.OnLevelComplete += () => _userData.Save();
_events.OnPurchaseComplete += () => _userData.Save();
_events.OnBoosterUsed += () => _userData.Save();

// ❌ KHÔNG gọi Save() mỗi frame hoặc mỗi action nhỏ
// ❌ KHÔNG gọi Save() trong Update()
```

---

## 15. Error Handling & Defensive Coding

### ServiceLocator — Khi nào lỗi và cách debug

**Lỗi hay gặp nhất:** `KeyNotFoundException: Service XxxService chưa được đăng ký.`

**Nguyên nhân phổ biến:**

1. **Gọi `ServiceLocator.Get<T>()` trong `Awake()`** — lúc này `GameContext.Start()` chưa chạy xong.
   → Sửa: chuyển sang `Start()` hoặc sau đó.

2. **Quên đăng ký service** trong `GameContext.RegisterServices()`.
   → Sửa: thêm dòng `ServiceLocator.Add(...)`.

3. **Đăng ký sai thứ tự** — service A phụ thuộc service B nhưng A đăng ký trước B.
   → Sửa: xem [Tham chiếu nhanh — Service Registration Order](#tham-chiếu-nhanh--service-registration-order).

### TryGet — Phiên bản an toàn

```csharp
// Thêm vào ServiceLocator.cs cho trường hợp cần check trước khi dùng
public static bool TryGet<T>(out T service) where T : class
{
    if (_services.TryGetValue(typeof(T), out var obj))
    {
        service = (T)obj;
        return true;
    }
    service = null;
    return false;
}
```

**Khi nào dùng `TryGet` vs `Get`:**
- `Get<T>()` — khi service **bắt buộc phải có** (99% trường hợp). Crash sớm = debug dễ.
- `TryGet<T>()` — khi service là **optional** (ví dụ: DebugService chỉ có trong dev build).

### Nguyên tắc chung

- **Fail fast, fail loud** — để lỗi crash sớm thay vì chạy sai âm thầm.
- **Không bắt exception rồi nuốt** — `catch (Exception) { }` là cấm.
- **Log có context** — `Debug.LogError($"[ClassName] Mô tả lỗi: {detail}")`.
- **Null check chỉ khi có lý do** — không spam `if (x != null)` ở mọi nơi. Nếu x luôn phải non-null, hãy để NullReferenceException crash và fix root cause.

---

## 16. Testing & Debug

### Quyết định kiến trúc
- **Không bắt buộc unit test cho gameplay** — gameplay thay đổi quá nhanh ở giai đoạn prototype.
- **Bắt buộc manual test** theo Acceptance Checklist (xem mỗi PRD).
- **Khuyến khích unit test cho Service** — ServiceLocator pattern giúp mock dễ dàng.

### Debug Tools — Convention

```csharp
// Assets/Scripts/Bootstrap/DebugService.cs
namespace YourGame.Bootstrap
{
    /// <summary>
    /// Service chỉ tồn tại trong Development build.
    /// Cung cấp shortcut để test nhanh.
    /// </summary>
    public class DebugService
    {
        #region Public Methods
        public void AddCoins(int amount)
        {
            var userData = ServiceLocator.Get<UserDataService>();
            userData.Data.coins += amount;
            userData.Save();
            Debug.Log($"[Debug] Added {amount} coins. Total: {userData.Data.coins}");
        }

        public void UnlockAllLevels()
        {
            var userData = ServiceLocator.Get<UserDataService>();
            // ... unlock logic
            Debug.Log("[Debug] All levels unlocked.");
        }

        public void SkipLevel()
        {
            var events = ServiceLocator.Get<GameEventService>();
            events.OnLevelComplete?.Invoke();
            Debug.Log("[Debug] Level skipped.");
        }
        #endregion
    }
}

// Đăng ký có điều kiện trong GameContext:
#if UNITY_EDITOR || DEVELOPMENT_BUILD
    ServiceLocator.Add(new DebugService());
#endif
```

### Mock Service cho Unit Test

```csharp
// Ví dụ: test BoosterService mà không cần game chạy
[Test]
public void MagnetBooster_ShouldExecuteWithoutError()
{
    // Arrange — đăng ký mock service
    ServiceLocator.Add(new GameEventService());
    ServiceLocator.Add(new UserDataService()); // sẽ tạo SaveData mới

    var boosterService = new BoosterService();
    var context = new BoosterContext(/* mock data */);

    // Act
    boosterService.Execute(BoosterType.Magnet, context);

    // Assert
    Assert.Pass();
}
```

### Log Convention

```csharp
// Format: [ClassName] Message cụ thể, có data
Debug.Log($"[AudioService] Playing SFX: {clipName}");
Debug.LogWarning($"[ObstacleRegistry] Unknown type: {type}. Skipping.");
Debug.LogError($"[UserDataService] Save failed: {exception.Message}");

// KHÔNG log quá nhiều trong Update/FixedUpdate — gây lag
// KHÔNG log sensitive data (token, password) — kể cả trong dev build
```

---

## 17. Code Standards

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

## 18. Branch Strategy

```
main              ← production, luôn ổn định
dev               ← active feature development
feature/xxx       ← branch cho từng feature cụ thể (merge vào dev khi xong)
```

**Quy trình:**
- Tạo branch `feature/xxx` từ `dev` cho mỗi feature hoặc task.
- Code, test trên feature branch → tạo Pull Request vào `dev`.
- Code review bắt buộc trước khi merge.
- Khi `dev` ổn định, đủ feature cho bản build → merge vào `main`.
- Hotfix: tạo branch `hotfix/xxx` từ `main`, fix xong merge vào cả `main` và `dev`.

---

## 19. Project Setup Checklist

Dùng checklist này khi bắt đầu một dự án mới. Làm theo thứ tự từ trên xuống.

```markdown
## Phase 1 — Project & Package Setup
- [ ] Tạo Unity project mới (2022.3 LTS, URP)
- [ ] Import UniTask từ GitHub Release (.unitypackage)
- [ ] Import Newtonsoft.Json (`com.unity.nuget.newtonsoft-json`)
- [ ] Tạo cấu trúc thư mục theo Section 3
- [ ] Setup .gitignore cho Unity
- [ ] Init Git repo, tạo branch `main` và `dev`

## Phase 2 — Bootstrap & Core Services
- [ ] Tạo scene `Bootstrap`, đặt index 0 trong Build Settings
- [ ] Implement `GameContext.cs` (singleton, DontDestroyOnLoad)
- [ ] Implement `ServiceLocator.cs` với Add/Get/TryGet/InitializeAllAsync/DisposeAll
- [ ] Implement `IInitializable.cs`
- [ ] Tạo `GameContextSettings` SO
- [ ] Validate: bấm Play → không có error trong Console

## Phase 3 — Event System & Data
- [ ] Implement `GameEventService`
- [ ] Implement `SaveData` model
- [ ] Implement `UserDataService` (JsonConvert, IInitializable, IDisposable)
- [ ] Đăng ký service trong `GameContext.RegisterServices()`
- [ ] Test save/load cycle + backup recovery

## Phase 4 — Config SOs
- [ ] Tạo config SO cho từng service cần config (Audio, Grid, Booster, ...)
- [ ] Tạo `PrefabReferencesSO`
- [ ] Kéo tất cả SO vào `GameContextSettings` Inspector
- [ ] Validate: service nhận đúng config khi init

## Phase 5 — UI Foundation
- [ ] Implement `BaseScreen`, `BasePopup`
- [ ] Implement `UIService`
- [ ] Tạo screen đầu tiên (HomeScreen hoặc GameScreen)
- [ ] Validate: show/hide screen hoạt động đúng

## Phase 6 — Analytics & Booster
- [ ] Implement `AnalyticsService` + `ITrackingEvent`
- [ ] Implement `BoosterService` + `BoosterCommand` base class
- [ ] Tạo event class đầu tiên (LevelCompleteEvent)
- [ ] Validate: track event log ra Console

## Phase 7 — Gameplay
- [ ] Tạo scene `Game`
- [ ] Implement load scene từ `GameContext.BeginInitialization()`
- [ ] Implement gameplay controllers và elements
- [ ] Implement `ObstacleRegistry` nếu game có nhiều loại element

## Phase 8 — Polish & Production
- [ ] Thêm Object Pooling cho high-frequency spawn/destroy
- [ ] Thêm DebugService (chỉ dev build)
- [ ] Audit naming convention + code standards
- [ ] Final test toàn bộ game flow
```

---

## 20. Những thứ KHÔNG làm

| Đừng | Làm thay vào đó |
|---|---|
| Dùng `.instance` trên từng service | `ServiceLocator.Get<T>()` |
| Serialized field trên MonoBehaviour cho config | Dùng ScriptableObject |
| Interface khi chỉ có một implementation | Viết class thẳng, interface sau khi cần |
| Inline string cho event/tracking name | Dùng `const string` |
| Nhiều utility file/class | Một file `Utils.cs`, một class `Utils` |
| Comment giải thích code làm gì | Comment giải thích tại sao |
| Method dài hơn 30 dòng | Tách method |
| Code bị comment out trong commit | Xóa hẳn |
| Sửa enum value đang được dùng trong data | Chỉ thêm mới, không bao giờ sửa/xóa |
| `async void` ở method thường | `async UniTask` — `async void` chỉ ở Unity callback |
| Dùng Coroutine | UniTask — Coroutine bị cấm trong dự án mới |
| `PlayerPrefs` cho game data | `UserDataService` + `JsonConvert` + file |
| Subscribe event bằng lambda | Method riêng — lambda không unsubscribe được |
| `Destroy(obj)` cho object spawn thường xuyên | Object pooling — `pool.Return(obj)` |
| UI script chứa business logic | UI chỉ hiển thị data và fire event |
| `SetActive(true/false)` cho show/hide UI | `CanvasGroup` alpha + interactable |
| `catch (Exception) { }` nuốt lỗi | Log error có context hoặc không catch |
| Xóa/rename field trong SaveData | Chỉ thêm mới, dùng `[JsonProperty]` nếu cần rename |
| Tạo thư mục script ngoài `Assets/Scripts/` | Tất cả code vào `Assets/Scripts/` |
| `Task.Run()` hoặc `System.Threading.Tasks` | Dùng UniTask — chạy trên main thread |

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

*Template này được tổng hợp từ kinh nghiệm thực chiến. Cập nhật version ở bảng đầu document khi có quyết định kiến trúc mới.*
