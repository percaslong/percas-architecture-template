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
15. [Scene Management](#15-scene-management)
16. [Level Data Architecture](#16-level-data-architecture)
17. [Error Handling & Defensive Coding](#17-error-handling--defensive-coding)
18. [Testing & Debug](#18-testing--debug)
19. [Code Standards](#19-code-standards)
20. [Branch Strategy](#20-branch-strategy)
21. [Project Setup Checklist](#21-project-setup-checklist)
22. [Những thứ KHÔNG làm](#22-những-thứ-không-làm)

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
2. Thêm scene vào Build Settings.
3. `GameContext.Start()` sẽ tự động load scene `Game` sau khi init xong (xem template trong [Section 4](#4-bootstrap--service-locator)).
4. `GameContext` sống ở scene `Bootstrap` với `DontDestroyOnLoad` — các scene khác chỉ chứa gameplay.

### Bước 5 — Kiểm tra flow

```
[Play] → Bootstrap scene load
       → GameContext.Awake() — singleton init
       → GameContext.Start() — RegisterServices() → InitializeAllAsync() → LoadSceneAsync("Game")
       → Game scene load
       → Gameplay scripts gọi ServiceLocator.Get<T>() → OK
```

**Nếu gặp lỗi:** xem [Error Handling & Defensive Coding](#17-error-handling--defensive-coding).

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
│  │  │  │PrefabRefs    │ │ UIPrefabs           │  │  │  │
│  │  │  └──────────────┘ └─────────────────────┘  │  │  │
│  │  │  ┌──────────────┐                          │  │  │
│  │  │  │AllLevels     │                          │  │  │
│  │  │  └──────────────┘                          │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │                      │                            │  │
│  │                      ▼                            │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │         ServiceLocator (Static)             │  │  │
│  │  │  ┌────────────┐  ┌──────────────────────┐   │  │  │
│  │  │  │ GameEvent   │  │ UserDataService       │   │  │  │
│  │  │  │ Service     │  │ (JsonConvert+PlayerPrefs)│  │  │  │
│  │  │  └──────┬─────┘  └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ AudioService         │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ SceneService         │   │  │  │
│  │  │         │         └──────────────────────┘   │  │  │
│  │  │         │         ┌──────────────────────┐   │  │  │
│  │  │         ├────────▶│ LevelManager         │   │  │  │
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
│  │  │         └────────▶│ RemoteConfigService  │   │  │  │
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
│   │   ├── UserData/               UserDataService, data classes (ProgressData, EconomyData, ...)
│   │   ├── UI/                     UIService, BaseScreen, BasePopup
│   │   ├── Scene/                  SceneService
│   │   ├── Level/                  LevelManager
│   │   ├── RemoteConfig/           RemoteConfigService
│   │   └── ...
│   ├── GamePlay/                   Code gameplay chính
│   │   ├── Controller/
│   │   ├── Element/
│   │   └── ...
│   ├── Config/                     ScriptableObject config classes (AudioConfigSO, UIPrefabsSO, AllLevelsSO, ...)
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
- Thư mục phải khớp với namespace (xem [Code Standards](#19-code-standards)).
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
using Cysharp.Threading.Tasks;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace YourGame.Bootstrap
{
    public class GameContext : MonoBehaviour
    {
        #region Properties
        public static GameContext Instance { get; private set; }

        /// <summary>Settings SO chứa tất cả config references.</summary>
        [field: SerializeField] public GameContextSettings Settings { get; private set; }
        #endregion

        #region Fields
        [Header("UI Roots — kéo Transform từ Canvas trong Bootstrap scene")]
        [SerializeField] private Transform _screenRoot;
        [SerializeField] private Transform _popupRoot;
        #endregion

        #region Unity Callbacks
        private void Awake()
        {
            if (Instance != null) { Destroy(gameObject); return; }
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }

        private async void Start()
        {
            try
            {
                RegisterServices();
                await ServiceLocator.InitializeAllAsync();
                // Tất cả service đã sẵn sàng — load game scene
                await SceneManager.LoadSceneAsync("Game", LoadSceneMode.Additive).ToUniTask();
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[GameContext] Initialization failed: {e}");
            }
        }

        private void OnApplicationQuit()
        {
            ServiceLocator.DisposeAll();
        }
        #endregion

        #region Private Methods
        private void RegisterServices()
        {
            // Đăng ký theo thứ tự dependency — service nào phụ thuộc service khác thì đăng ký sau
            ServiceLocator.Add(new GameEventService());
            ServiceLocator.Add(new UserDataService());
            ServiceLocator.Add(new AudioService(Settings.AudioConfig));
            ServiceLocator.Add(new SceneService());
            ServiceLocator.Add(new LevelManager(Settings.AllLevels));
            ServiceLocator.Add(new UIService(_screenRoot, _popupRoot, Settings.UIPrefabs));
            ServiceLocator.Add(new BoosterService());
            ServiceLocator.Add(new AnalyticsService());
            ServiceLocator.Add(new RemoteConfigService(Settings));

            #if UNITY_EDITOR || DEVELOPMENT_BUILD
            ServiceLocator.Add(new DebugService());
            #endif
        }
        #endregion
    }
}
```

### Template: ServiceLocator.cs

```csharp
// Assets/Scripts/Bootstrap/ServiceLocator.cs
using System;
using System.Collections.Generic;
using Cysharp.Threading.Tasks;
using UnityEngine;

namespace YourGame.Bootstrap
{
    public static class ServiceLocator
    {
        private static readonly Dictionary<Type, object> _services = new();
        private static readonly List<object> _registrationOrder = new();

        /// <summary>Đăng ký service. Chỉ gọi từ GameContext.RegisterServices().</summary>
        public static void Add<T>(T service) where T : class
        {
            _services[typeof(T)] = service;
            _registrationOrder.Add(service);
        }

        /// <summary>Lấy service. Throws nếu chưa đăng ký.</summary>
        public static T Get<T>() where T : class
        {
            if (_services.TryGetValue(typeof(T), out var service))
                return (T)service;
            throw new KeyNotFoundException($"Service {typeof(T).Name} chưa được đăng ký.");
        }

        /// <summary>Lấy service an toàn — dùng cho service optional (DebugService, ...).</summary>
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
                if (_registrationOrder[i] is IDisposable disposable)
                {
                    try { disposable.Dispose(); }
                    catch (Exception e) { Debug.LogError($"Dispose error: {e}"); }
                }
            }
            _services.Clear();
            _registrationOrder.Clear();
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
using Cysharp.Threading.Tasks;

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

### Ví dụ: Service dùng IInitializable và IDisposable

```csharp
public class UserDataService : System.IDisposable
{
    public UserDataService()
    {
        // Load từ PlayerPrefs — synchronous, không cần IInitializable
        LoadAll();
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
- SO chỉ chứa **giá trị mặc định** — sạch, đơn giản, không có remote key.
- Remote Config override được xử lý tập trung tại `RemoteConfigService` — SO không biết gì về remote.
- `GameContextSettings` là SO trung tâm, chứa references đến tất cả config SO khác.

### Pattern SO — chỉ chứa giá trị mặc định

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
        #endregion
    }
}

// Assets/Scripts/Config/EconomyConfigSO.cs
namespace YourGame.Config
{
    [CreateAssetMenu(menuName = "Config/Economy")]
    public class EconomyConfigSO : ScriptableObject
    {
        #region Fields
        [Header("Revive Cost")]
        public int reviveCost1 = 900;
        public int reviveCost2 = 1900;
        public int reviveCost3 = 2900;

        [Header("Rewards")]
        public int coinsPerLevel = 50;
        public int coinsPerAd = 100;
        #endregion
    }
}
```

### Remote Config — Override tập trung

```csharp
// Assets/Scripts/Services/RemoteConfig/RemoteConfigService.cs
using Cysharp.Threading.Tasks;

namespace YourGame.Services.RemoteConfig
{
    /// <summary>
    /// Fetch remote values và ghi đè lên SO.
    /// SO giữ giá trị mặc định — remote chỉ override nếu key tồn tại.
    /// Chạy một lần sau khi fetch xong, trước khi game bắt đầu.
    /// </summary>
    public class RemoteConfigService : IInitializable
    {
        #region Fields
        private readonly GameContextSettings _settings;
        #endregion

        #region Constructor
        public RemoteConfigService(GameContextSettings settings)
        {
            _settings = settings;
        }
        #endregion

        #region IInitializable
        public async UniTask InitializeAsync()
        {
            await FetchRemoteValues();
            ApplyOverrides();
        }
        #endregion

        #region Private Methods
        private async UniTask FetchRemoteValues()
        {
            // TODO: gọi SDK fetch, chờ response hoặc timeout
            // Ví dụ: await Firebase.RemoteConfig.FetchAsync(TimeSpan.FromSeconds(10));
            await UniTask.CompletedTask; // placeholder
        }

        private void ApplyOverrides()
        {
            var audio = _settings.AudioConfig;
            var economy = _settings.EconomyConfig;

            // Audio
            audio.defaultMusicVolume = GetFloat("default_music_volume", audio.defaultMusicVolume);
            audio.defaultSfxVolume   = GetFloat("default_sfx_volume",   audio.defaultSfxVolume);

            // Economy
            economy.reviveCost1   = GetInt("revive_cost_1",   economy.reviveCost1);
            economy.reviveCost2   = GetInt("revive_cost_2",   economy.reviveCost2);
            economy.coinsPerLevel = GetInt("coins_per_level", economy.coinsPerLevel);

            // Thêm override cho config mới ở đây — một chỗ duy nhất
        }

        /// <summary>Lấy giá trị remote. Nếu key không tồn tại, trả về defaultValue.</summary>
        private float GetFloat(string key, float defaultValue)
        {
            // if (RemoteSDK.HasKey(key)) return RemoteSDK.GetFloat(key);
            return defaultValue;
        }

        private int GetInt(string key, int defaultValue)
        {
            // if (RemoteSDK.HasKey(key)) return RemoteSDK.GetInt(key);
            return defaultValue;
        }
        #endregion
    }
}
```

**Tại sao làm thế này:**
- SO giữ nguyên vai trò: **giá trị mặc định, tunable trong Editor**. Không bị phình thêm field remote key.
- Khi thêm 50 biến config, chỉ cần thêm 50 dòng `TryOverride` tại **một file duy nhất** — không phải rải remote key khắp các SO.
- Nếu không dùng remote config → xóa `RemoteConfigService`, SO vẫn hoạt động bình thường.
- Debug dễ: mở SO trong Inspector thấy giá trị hiện tại (đã bị override hay chưa).

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
        [field: SerializeField] public EconomyConfigSO  EconomyConfig  { get; private set; }
        [field: SerializeField] public BoosterConfigSO  BoosterConfig  { get; private set; }
        [field: SerializeField] public PrefabReferencesSO Prefabs      { get; private set; }
        [field: SerializeField] public UIPrefabsSO     UIPrefabs    { get; private set; }
        [field: SerializeField] public AllLevelsSO     AllLevels    { get; private set; }
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

### Template: UIPrefabsSO

```csharp
// Assets/Scripts/Config/UIPrefabsSO.cs
using UnityEngine;

namespace YourGame.Config
{
    /// <summary>
    /// SO chứa references trực tiếp đến tất cả UI prefab.
    /// Kéo prefab vào Inspector — slot trống = lỗi thấy ngay, không đợi runtime.
    /// </summary>
    [CreateAssetMenu(menuName = "Config/UIPrefabs")]
    public class UIPrefabsSO : ScriptableObject
    {
        #region Screens
        [Header("Screens")]
        public HomeScreen homeScreen;
        public GameScreen gameScreen;
        #endregion

        #region Popups
        [Header("Popups")]
        public ResultPopup resultPopup;
        public PurchasePopup purchasePopup;
        public SettingsPopup settingsPopup;
        // Thêm screen/popup mới → thêm 1 field ở đây
        #endregion
    }
}
```

### Template: UIService

```csharp
// Assets/Scripts/Services/UI/UIService.cs
using System;
using System.Collections.Generic;
using UnityEngine;

namespace YourGame.Services.UI
{
    public class UIService
    {
        #region Fields
        private BaseScreen _currentScreen;
        private readonly Stack<BasePopup> _popupStack = new();
        private readonly Dictionary<Type, MonoBehaviour> _instanceCache = new();
        private readonly Dictionary<Type, MonoBehaviour> _prefabMap;
        private readonly Transform _screenRoot;
        private readonly Transform _popupRoot;
        #endregion

        #region Constructor
        public UIService(Transform screenRoot, Transform popupRoot, UIPrefabsSO prefabs)
        {
            _screenRoot = screenRoot;
            _popupRoot = popupRoot;
            _prefabMap = BuildPrefabMap(prefabs);
        }
        #endregion

        #region Public Methods
        /// <summary>Chuyển screen. Hide screen cũ, show screen mới.</summary>
        public T ShowScreen<T>() where T : BaseScreen
        {
            _currentScreen?.OnHide();
            var screen = GetOrCreate<T>(_screenRoot);
            screen.OnShow();
            _currentScreen = screen;
            return screen;
        }

        /// <summary>Show popup, push vào stack.</summary>
        public T ShowPopup<T>() where T : BasePopup
        {
            var popup = GetOrCreate<T>(_popupRoot);
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
        private T GetOrCreate<T>(Transform root) where T : MonoBehaviour
        {
            if (_instanceCache.TryGetValue(typeof(T), out var existing))
                return (T)existing;

            if (!_prefabMap.TryGetValue(typeof(T), out var prefab) || prefab == null)
            {
                Debug.LogError($"[UIService] Prefab for '{typeof(T).Name}' not assigned in UIPrefabsSO.");
                return null;
            }

            var instance = UnityEngine.Object.Instantiate((T)prefab, root);
            _instanceCache[typeof(T)] = instance;
            return instance;
        }

        /// <summary>Build mapping type → prefab từ SO. Chạy 1 lần trong constructor.</summary>
        private Dictionary<Type, MonoBehaviour> BuildPrefabMap(UIPrefabsSO prefabs)
        {
            return new Dictionary<Type, MonoBehaviour>
            {
                { typeof(HomeScreen),     prefabs.homeScreen },
                { typeof(GameScreen),     prefabs.gameScreen },
                { typeof(ResultPopup),    prefabs.resultPopup },
                { typeof(PurchasePopup),  prefabs.purchasePopup },
                { typeof(SettingsPopup),  prefabs.settingsPopup },
                // Thêm mapping cho screen/popup mới ở đây
            };
        }
        #endregion
    }
}
```

### Thêm Screen / Popup mới — Checklist

1. Tạo class mới kế thừa `BaseScreen` hoặc `BasePopup`.
2. Tạo prefab, attach script vào.
3. Thêm **1 field** vào `UIPrefabsSO`.
4. Thêm **1 dòng** vào `BuildPrefabMap()` trong `UIService`.
5. Kéo prefab vào slot trong `UIPrefabsSO` Inspector.

**Xong. Nếu quên bước 5, Inspector sẽ hiện slot trống — thấy ngay, không đợi runtime.**

### Nguyên tắc UI

- UI script **không chứa business logic** — chỉ hiển thị data và fire event.
- Data truyền vào UI qua method `OnShow(data)` — UI không tự đi lấy data.
- Animation/tween dùng DOTween hoặc UniTask delay — không dùng Animation Controller cho UI đơn giản.

### Ví dụ: Truyền data vào UI qua OnShow(data)

```csharp
// ===== DATA MODEL — struct chứa đúng những gì UI cần hiển thị =====

// Assets/Scripts/Services/UI/Screens/ResultScreenData.cs
namespace YourGame.Services.UI
{
    public struct ResultScreenData
    {
        public int level;
        public int stars;
        public int coinsEarned;
        public bool isNewHighScore;
    }
}

// ===== SCREEN — chỉ nhận data và hiển thị, không tự query =====

// Assets/Scripts/Services/UI/Screens/ResultScreen.cs
namespace YourGame.Services.UI
{
    public class ResultScreen : BaseScreen
    {
        #region Fields
        [SerializeField] private TMP_Text _levelText;
        [SerializeField] private TMP_Text _coinsText;
        [SerializeField] private GameObject _highScoreBadge;
        [SerializeField] private Image[] _starImages;
        #endregion

        #region Public Methods
        /// <summary>Show screen với data cụ thể. Caller chuẩn bị data, screen chỉ hiển thị.</summary>
        public void OnShow(ResultScreenData data)
        {
            base.OnShow(); // CanvasGroup alpha = 1

            _levelText.text = $"Level {data.level}";
            _coinsText.text = $"+{data.coinsEarned}";
            _highScoreBadge.SetActive(data.isNewHighScore);

            for (int i = 0; i < _starImages.Length; i++)
                _starImages[i].enabled = i < data.stars;
        }
        #endregion
    }
}

// ===== CALLER — chuẩn bị data rồi truyền vào UI =====

// Trong GameController hoặc bất kỳ đâu:
var resultScreen = ServiceLocator.Get<UIService>().ShowScreen<ResultScreen>();
resultScreen.OnShow(new ResultScreenData
{
    level = 5,
    stars = 3,
    coinsEarned = 150,
    isNewHighScore = true
});
```

```csharp
// ===== POPUP VỚI DATA — ví dụ popup xác nhận mua booster =====

// Assets/Scripts/Services/UI/Popups/PurchasePopupData.cs
namespace YourGame.Services.UI
{
    public struct PurchasePopupData
    {
        public string itemName;
        public int cost;
        public int currentCoins;
        public System.Action onConfirm; // callback khi user bấm Mua
    }
}

// Assets/Scripts/Services/UI/Popups/PurchasePopup.cs
namespace YourGame.Services.UI
{
    public class PurchasePopup : BasePopup
    {
        #region Fields
        [SerializeField] private TMP_Text _itemNameText;
        [SerializeField] private TMP_Text _costText;
        [SerializeField] private Button _buyButton;
        [SerializeField] private TMP_Text _buyButtonText;

        private System.Action _onConfirm;
        #endregion

        #region Public Methods
        public void OnShow(PurchasePopupData data)
        {
            base.OnShow();

            _itemNameText.text = data.itemName;
            _costText.text = $"{data.cost} coins";
            _onConfirm = data.onConfirm;

            // Disable nút Mua nếu không đủ tiền
            bool canAfford = data.currentCoins >= data.cost;
            _buyButton.interactable = canAfford;
            _buyButtonText.text = canAfford ? "Mua" : "Không đủ";
        }
        #endregion

        #region UI Callbacks — gắn trong Inspector
        public void OnBuyButtonClicked()
        {
            _onConfirm?.Invoke();
            Close();
        }
        #endregion
    }
}

// Caller:
var userData = ServiceLocator.Get<UserDataService>();
var popup = ServiceLocator.Get<UIService>().ShowPopup<PurchasePopup>();
popup.OnShow(new PurchasePopupData
{
    itemName = "Magnet Booster",
    cost = 500,
    currentCoins = userData.Economy.coins,
    onConfirm = () =>
    {
        userData.Economy.coins -= 500;
        userData.SaveEconomy();
    }
});
```

**Tại sao truyền data qua OnShow thay vì UI tự lấy:**
- UI không phụ thuộc vào service nào — dễ test, dễ reuse.
- Caller kiểm soát data gì được hiển thị — cùng 1 popup có thể show với data khác nhau.
- Khi đọc code, nhìn vào caller là biết ngay popup sẽ hiển thị gì — không cần mở popup script ra tìm.

---

## 14. Save / Load — Data Persistence

### Quyết định kiến trúc
- Serialize/deserialize bằng `Newtonsoft.Json` (`JsonConvert`), lưu vào `PlayerPrefs`.
- **Mỗi feature có data class riêng** — `ProgressData`, `EconomyData`, `LivesData`, ... Không gom tất cả vào một class.
- Mỗi data class được lưu dưới một `PlayerPrefs` key riêng biệt.
- `UserDataService` là entry point duy nhất để đọc/ghi tất cả data — không class nào khác gọi `PlayerPrefs` trực tiếp.
- Auto-save tại các điểm quan trọng (level complete, purchase, ...).

### Package Setup

```
// Package Manager → Add package by name:
com.unity.nuget.newtonsoft-json
```

### Template: Data class theo feature

```csharp
// Assets/Scripts/Services/UserData/ProgressData.cs
namespace YourGame.Services.UserData
{
    /// <summary>Data tiến trình chơi game.</summary>
    [System.Serializable]
    public class ProgressData
    {
        public int currentLevel = 1;
        public int totalStars = 0;
        public List<int> completedLevels = new();
    }
}

// Assets/Scripts/Services/UserData/EconomyData.cs
namespace YourGame.Services.UserData
{
    /// <summary>Data tiền tệ và inventory.</summary>
    [System.Serializable]
    public class EconomyData
    {
        public int coins = 0;
        public int gems = 0;
        public Dictionary<string, int> boosters = new();
    }
}

// Assets/Scripts/Services/UserData/LivesData.cs
namespace YourGame.Services.UserData
{
    /// <summary>Data hệ thống mạng.</summary>
    [System.Serializable]
    public class LivesData
    {
        public int currentLives = 5;
        public long lastRegenTimestamp; // epoch seconds (UTC)
        public long infiniteLivesExpiry; // epoch seconds, 0 = không active
    }
}

// Assets/Scripts/Services/UserData/SettingsData.cs
namespace YourGame.Services.UserData
{
    /// <summary>Data cài đặt người dùng.</summary>
    [System.Serializable]
    public class SettingsData
    {
        public float musicVolume = 0.8f;
        public float sfxVolume = 1f;
        public bool hapticEnabled = true;
    }
}
```

### Template: UserDataService

```csharp
// Assets/Scripts/Services/UserData/UserDataService.cs
using System;
using System.Collections.Generic;
using Newtonsoft.Json;
using UnityEngine;

namespace YourGame.Services.UserData
{
    public class UserDataService : IDisposable
    {
        #region Constants
        private const string KEY_PROGRESS = "data_progress";
        private const string KEY_ECONOMY  = "data_economy";
        private const string KEY_LIVES    = "data_lives";
        private const string KEY_SETTINGS = "data_settings";
        // Thêm key mới cho mỗi feature data class mới
        #endregion

        #region Properties
        public ProgressData Progress { get; private set; }
        public EconomyData Economy   { get; private set; }
        public LivesData Lives       { get; private set; }
        public SettingsData Settings  { get; private set; }
        #endregion

        #region Constructor
        public UserDataService()
        {
            LoadAll();
        }
        #endregion

        #region Public Methods
        /// <summary>Save tất cả data. Gọi tại các điểm quan trọng.</summary>
        public void Save()
        {
            SaveEntry(KEY_PROGRESS, Progress);
            SaveEntry(KEY_ECONOMY, Economy);
            SaveEntry(KEY_LIVES, Lives);
            SaveEntry(KEY_SETTINGS, Settings);
            PlayerPrefs.Save();
        }

        /// <summary>Save một feature data cụ thể — dùng khi chỉ thay đổi 1 feature.</summary>
        public void SaveProgress() { SaveEntry(KEY_PROGRESS, Progress); PlayerPrefs.Save(); }
        public void SaveEconomy()  { SaveEntry(KEY_ECONOMY, Economy);   PlayerPrefs.Save(); }
        public void SaveLives()    { SaveEntry(KEY_LIVES, Lives);       PlayerPrefs.Save(); }
        public void SaveSettings() { SaveEntry(KEY_SETTINGS, Settings); PlayerPrefs.Save(); }

        /// <summary>Xóa toàn bộ data — dùng cho debug hoặc reset account.</summary>
        public void DeleteAll()
        {
            PlayerPrefs.DeleteAll();
            LoadAll(); // Reload default values
        }

        /// <summary>Export toàn bộ data ra JSON — dùng để debug trong Console.</summary>
        public string ExportAllToJson()
        {
            var snapshot = new Dictionary<string, object>
            {
                { KEY_PROGRESS, Progress },
                { KEY_ECONOMY, Economy },
                { KEY_LIVES, Lives },
                { KEY_SETTINGS, Settings }
            };
            return JsonConvert.SerializeObject(snapshot, Formatting.Indented);
        }
        #endregion

        #region IDisposable
        public void Dispose()
        {
            Save(); // Auto-save khi game tắt
        }
        #endregion

        #region Private Methods
        private void LoadAll()
        {
            Progress = LoadEntry<ProgressData>(KEY_PROGRESS);
            Economy  = LoadEntry<EconomyData>(KEY_ECONOMY);
            Lives    = LoadEntry<LivesData>(KEY_LIVES);
            Settings = LoadEntry<SettingsData>(KEY_SETTINGS);
        }

        private T LoadEntry<T>(string key) where T : new()
        {
            if (!PlayerPrefs.HasKey(key))
                return new T();

            try
            {
                var json = PlayerPrefs.GetString(key);
                return JsonConvert.DeserializeObject<T>(json) ?? new T();
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[UserDataService] Load '{key}' failed: {e.Message}. Using default.");
                return new T();
            }
        }

        private void SaveEntry<T>(string key, T data)
        {
            try
            {
                var json = JsonConvert.SerializeObject(data);
                PlayerPrefs.SetString(key, json);
            }
            catch (System.Exception e)
            {
                Debug.LogError($"[UserDataService] Save '{key}' failed: {e.Message}");
            }
        }
        #endregion
    }
}
```

### Cách dùng

```csharp
// Lấy service
var userData = ServiceLocator.Get<UserDataService>();

// Đọc/ghi data
userData.Economy.coins += 100;
userData.SaveEconomy(); // Save chỉ economy — nhanh, gọn

// Hoặc save tất cả
userData.Progress.currentLevel = 5;
userData.Economy.gems += 10;
userData.Save(); // Save toàn bộ

// Debug — in toàn bộ data ra Console
Debug.Log(userData.ExportAllToJson());
```

### Thêm feature data mới — Checklist

1. Tạo data class mới: `XxxData.cs` trong `Assets/Scripts/Services/UserData/`.
2. Thêm `const string KEY_XXX` vào `UserDataService`.
3. Thêm property `public XxxData Xxx { get; private set; }`.
4. Thêm `LoadEntry` trong `LoadAll()`, thêm `SaveEntry` trong `Save()`.
5. Thêm method `SaveXxx()` cho save riêng lẻ.
6. Thêm vào `ExportAllToJson()`.

**Xong. Không class nào khác bị ảnh hưởng.**

### Schema — Nguyên tắc

- **Không bao giờ xóa field** trong data class đã ship — chỉ thêm mới.
- **Không rename field** — `JsonConvert` sẽ mất data nếu tên thay đổi.
- Nếu bắt buộc phải rename: dùng `[JsonProperty("old_name")]` attribute.
- `JsonConvert.DeserializeObject` tự gán default value cho field mới → thêm field mới không cần migration.
- **Không đổi PlayerPrefs key** — key là identity của data block.

### Khi nào gọi Save()

```csharp
// ✅ Gọi Save hoặc SaveXxx() tại các milestone — dùng method riêng, không dùng lambda
private void Start()
{
    _events = ServiceLocator.Get<GameEventService>();
    _userData = ServiceLocator.Get<UserDataService>();

    _events.OnLevelComplete += HandleLevelComplete;
    _events.OnPurchaseComplete += HandlePurchaseComplete;
    _events.OnBoosterUsed += HandleBoosterUsed;
    _events.OnSettingsChanged += HandleSettingsChanged;
}

private void OnDestroy()
{
    _events.OnLevelComplete -= HandleLevelComplete;
    _events.OnPurchaseComplete -= HandlePurchaseComplete;
    _events.OnBoosterUsed -= HandleBoosterUsed;
    _events.OnSettingsChanged -= HandleSettingsChanged;
}

private void HandleLevelComplete() => _userData.SaveProgress();
private void HandlePurchaseComplete() => _userData.SaveEconomy();
private void HandleBoosterUsed() => _userData.SaveEconomy();
private void HandleSettingsChanged() => _userData.SaveSettings();

// ❌ KHÔNG gọi Save() mỗi frame hoặc mỗi action nhỏ
// ❌ KHÔNG gọi Save() trong Update()
```

---

## 15. Scene Management

### Quyết định kiến trúc
- **Bootstrap scene** (index 0) chứa `GameContext` — luôn load đầu tiên, không bao giờ unload.
- Các scene khác load bằng **Additive mode** — `GameContext` vẫn sống xuyên suốt.
- `SceneService` quản lý load/unload — không class nào khác gọi `SceneManager` trực tiếp.
- Transition (fade in/out) xử lý bởi `SceneService` — gameplay code không biết gì về transition.

### Scene Flow cho Hybrid Puzzle

```
Bootstrap (always loaded, DontDestroyOnLoad)
    │
    ├──▶ Home (Additive) ──── Main menu, level select
    │         │
    │         └──▶ Unload Home → Load Game (Additive) ──── Gameplay
    │                                │
    │                                ├── Win  → Show ResultPopup (overlay) → Next level hoặc Home
    │                                └── Lose → Show ResultPopup (overlay) → Retry hoặc Home
    │                                               │
    │                                               ├── Next/Retry → giữ Game scene, load level mới
    │                                               └── Home → Unload Game → Load Home
```

**Lưu ý:** Result là **popup** (xem [Section 13 — UI Architecture](#13-ui-architecture)), không phải scene riêng. Popup overlay lên Game scene — player vẫn thấy board phía sau.

### Template: SceneService

```csharp
// Assets/Scripts/Services/Scene/SceneService.cs
using Cysharp.Threading.Tasks;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace YourGame.Services.Scene
{
    public class SceneService
    {
        #region Fields
        private string _currentScene;
        #endregion

        #region Public Methods
        /// <summary>Load scene mới, unload scene cũ. Có transition fade.</summary>
        public async UniTask LoadScene(string sceneName, CancellationToken ct = default)
        {
            // Fade out
            await FadeOut(ct);

            // Unload scene cũ nếu có
            if (!string.IsNullOrEmpty(_currentScene))
                await SceneManager.UnloadSceneAsync(_currentScene).ToUniTask(cancellationToken: ct);

            // Load scene mới
            await SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive).ToUniTask(cancellationToken: ct);
            _currentScene = sceneName;

            // Fade in
            await FadeIn(ct);
        }
        #endregion

        #region Private Methods
        private async UniTask FadeOut(CancellationToken ct)
        {
            // TODO: CanvasGroup fade hoặc Animator
            await UniTask.Delay(300, cancellationToken: ct);
        }

        private async UniTask FadeIn(CancellationToken ct)
        {
            await UniTask.Delay(300, cancellationToken: ct);
        }
        #endregion
    }
}

// Cách dùng:
var sceneService = ServiceLocator.Get<SceneService>();
await sceneService.LoadScene("Game");
```

### Nguyên tắc

- **Không gọi `SceneManager` trực tiếp** từ gameplay code — luôn qua `SceneService`.
- **Không dùng `LoadSceneMode.Single`** — sẽ destroy `GameContext`.
- Mỗi scene là self-contained: có Canvas riêng, Camera riêng (nếu cần), EventSystem chỉ ở Bootstrap.
- Scene name dùng `const string` — không inline literal.

---

## 16. Level Data Architecture

### Quyết định kiến trúc
- Mỗi level là một **ScriptableObject riêng** (`LevelDataSO`) — edit độc lập trong Inspector, Git-friendly.
- **`AllLevelsSO`** chứa `List<LevelDataSO>` — list references đến tất cả level SO, truy cập O(1) bằng index.
- `LevelManager` là service, đăng ký trong `ServiceLocator`, nhận `AllLevelsSO` qua constructor.
- Level data chỉ chứa **input** (cấu hình level) — không chứa runtime state (progress, score).
- Runtime state thuộc về controller — **không ghi ngược vào SO**.

### Template: LevelDataSO

```csharp
// Assets/Scripts/Config/LevelDataSO.cs
using UnityEngine;

namespace YourGame.Config
{
    /// <summary>Cấu hình 1 level. Mỗi level là 1 asset SO riêng.</summary>
    [CreateAssetMenu(menuName = "Levels/LevelData")]
    public class LevelDataSO : ScriptableObject
    {
        #region Fields
        [Header("Basic Info")]
        public int levelNumber;
        public float targetTime; // seconds, 0 = no time limit

        [Header("Grid")]
        public int gridWidth = 5;
        public int gridHeight = 5;
        public CellData[] cells; // serialize grid layout

        [Header("Win Condition")]
        public WinConditionType winCondition;
        public int winTarget; // ví dụ: số cát cần đổ, số move tối đa

        [Header("Boosters")]
        public BoosterType[] availableBoosters;
        #endregion
    }

    [System.Serializable]
    public struct CellData
    {
        public int x;
        public int y;
        public ElementType elementType;
        public int rotation; // 0, 90, 180, 270
    }
}
```

### Template: AllLevelsSO — List references

```csharp
// Assets/Scripts/Config/AllLevelsSO.cs
using System.Collections.Generic;
using UnityEngine;

namespace YourGame.Config
{
    /// <summary>
    /// SO trung tâm chứa references đến tất cả LevelDataSO.
    /// Thứ tự trong list = thứ tự level (index 0 = level 1).
    /// </summary>
    [CreateAssetMenu(menuName = "Config/AllLevels")]
    public class AllLevelsSO : ScriptableObject
    {
        #region Fields
        [SerializeField] private List<LevelDataSO> levels = new();
        #endregion

        #region Properties
        public int TotalLevels => levels.Count;
        #endregion

        #region Public Methods
        /// <summary>Lấy level theo number. O(1) — truy cập list bằng index.</summary>
        public LevelDataSO GetLevel(int levelNumber)
        {
            // Convention: levelNumber = index + 1 (level 1 ở index 0)
            int index = levelNumber - 1;
            if (index < 0 || index >= levels.Count)
                return null;
            return levels[index];
        }

        public bool HasLevel(int levelNumber)
        {
            return levelNumber >= 1 && levelNumber <= levels.Count;
        }
        #endregion
    }
}
```

### Template: LevelManager

```csharp
// Assets/Scripts/Services/Level/LevelManager.cs
using UnityEngine;

namespace YourGame.Services.Level
{
    public class LevelManager
    {
        #region Fields
        private readonly AllLevelsSO _allLevels;
        #endregion

        #region Properties
        public LevelDataSO CurrentLevel { get; private set; }
        public int TotalLevels => _allLevels.TotalLevels;
        #endregion

        #region Constructor
        public LevelManager(AllLevelsSO allLevels)
        {
            _allLevels = allLevels;
        }
        #endregion

        #region Public Methods
        /// <summary>Load level theo number. Truy cập trực tiếp từ List — rất nhanh.</summary>
        public LevelDataSO LoadLevel(int levelNumber)
        {
            CurrentLevel = _allLevels.GetLevel(levelNumber);
            if (CurrentLevel == null)
                Debug.LogError($"[LevelManager] Level {levelNumber} not found. Total: {TotalLevels}");
            return CurrentLevel;
        }

        /// <summary>Load level tiếp theo dựa trên progress hiện tại.</summary>
        public LevelDataSO LoadNextLevel()
        {
            var userData = ServiceLocator.Get<UserDataService>();
            return LoadLevel(userData.Progress.currentLevel);
        }

        /// <summary>Kiểm tra level có tồn tại không.</summary>
        public bool HasLevel(int levelNumber) => _allLevels.HasLevel(levelNumber);
        #endregion
    }
}

// Đăng ký trong GameContext.RegisterServices():
// ServiceLocator.Add(new LevelManager(Settings.AllLevels));
```

### Kết nối với GameContextSettings

```csharp
// Thêm vào GameContextSettings
[field: SerializeField] public AllLevelsSO AllLevels { get; private set; }
```

### Quy ước tổ chức Level SO

```
Assets/
└── ScriptableObjects/
    └── Levels/
        ├── Level_00001.asset      ← mỗi level 1 file SO riêng
        ├── Level_00002.asset
        ├── Level_00003.asset
        ├── ...
        └── AllLevels.asset        ← SO chứa list references đến tất cả level trên
```

- Tên file: `Level_XXXXX.asset` — padding **5 chữ số** (hỗ trợ đến 99,999 levels).
- Tạo level mới: Project window → Create → Levels → LevelData → edit trong Inspector.
- Sau khi tạo, **kéo SO vào list `AllLevels`** trong Inspector — đúng vị trí theo thứ tự level.

### Tại sao dùng SO riêng + AllLevelsSO references

| Tiêu chí | SO riêng + AllLevelsSO | Plain class trong 1 SO | File JSON riêng lẻ |
|---|---|---|---|
| Edit trong Inspector | ✅ Mỗi level edit độc lập | ❌ List quá dài, khó navigate | ❌ Không dùng Inspector |
| Git conflict | ✅ Mỗi file riêng, ít conflict | ❌ 1 file, dễ conflict | ✅ Mỗi file riêng |
| Tốc độ truy cập | ✅ O(1) qua AllLevelsSO list | ✅ O(1) | ❌ I/O mỗi lần load |
| Tạo level mới | ✅ Create SO → kéo vào list | Thêm entry vào list dài | Tạo file JSON |
| Preview nhanh | ✅ Click SO → thấy ngay config | Cuộn list tìm entry | Mở file text |

### Level Editor Tool — Khuyến nghị

Khi game có hàng nghìn levels, nên build custom Editor tool hỗ trợ:

1. **Auto-populate**: scan thư mục `Levels/` → tự thêm SO mới vào `AllLevelsSO` list đúng thứ tự.
2. **Batch create**: tạo nhiều level SO cùng lúc từ template.
3. **Visual Editor**: UI trực quan cho designer đặt element trên grid.

Tool này nằm trong `Assets/Editor/` — không ship cùng game.

---

## 17. Error Handling & Defensive Coding

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

`ServiceLocator.TryGet<T>()` đã có sẵn trong template (xem [Section 4](#4-bootstrap--service-locator)).

**Khi nào dùng `TryGet` vs `Get`:**
- `Get<T>()` — khi service **bắt buộc phải có** (99% trường hợp). Crash sớm = debug dễ.
- `TryGet<T>()` — khi service là **optional** (ví dụ: DebugService chỉ có trong dev build).

### Nguyên tắc chung

- **Fail fast, fail loud** — để lỗi crash sớm thay vì chạy sai âm thầm.
- **Không bắt exception rồi nuốt** — `catch (Exception) { }` là cấm.
- **Log có context** — `Debug.LogError($"[ClassName] Mô tả lỗi: {detail}")`.
- **Null check chỉ khi có lý do** — không spam `if (x != null)` ở mọi nơi. Nếu x luôn phải non-null, hãy để NullReferenceException crash và fix root cause.

---

## 18. Testing & Debug

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
            userData.Economy.coins += amount;
            userData.SaveEconomy();
            Debug.Log($"[Debug] Added {amount} coins. Total: {userData.Economy.coins}");
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
    ServiceLocator.Add(new UserDataService()); // sẽ tạo data mới từ default

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

## 19. Code Standards

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
// Chờ 1 frame để đảm bảo tất cả Awake() đã chạy xong trước khi đăng ký event
await UniTask.Yield();

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

### Update() vs UniTask — Khi nào dùng gì

> **Nguyên tắc:** Hạn chế `Update()` — chỉ dùng khi logic **thực sự cần chạy mỗi frame liên tục và vô điều kiện**. Phần lớn code trông giống Update thực ra là "chờ điều kiện" hoặc "chạy sequence" — dùng UniTask cho những trường hợp đó.

| Tình huống | Dùng | Lý do |
|---|---|---|
| Chờ điều kiện (animation xong, data sẵn sàng, ...) | `await UniTask.WaitUntil()` | Rõ ràng, tự cleanup khi destroy |
| Timer / countdown / delay | `await UniTask.Delay()` | Không cần tích lũy deltaTime thủ công |
| Sequence (spawn → chờ → fade → done) | `async UniTask` method | Code sequential, dễ đọc, dễ debug |
| One-shot reaction (chờ xong → làm 1 việc) | `await UniTask.WaitUntil()` | Không cần biến state, không cần loop |
| Di chuyển / lerp liên tục mỗi frame | `Update()` | Đúng bản chất — chạy mỗi frame vô điều kiện |
| Input liên tục (drag, swipe, multi-touch) | `Update()` | Cần đọc input state mỗi frame |
| Camera follow | `LateUpdate()` | Unity convention — chạy sau tất cả Update |
| Physics (lực, velocity, raycast liên tục) | `FixedUpdate()` | Unity physics step |

**Ví dụ — sai vs đúng:**

```csharp
// ❌ SAI — dùng Update để chờ điều kiện
private bool _isReady;
private void Update()
{
    if (_isReady)
    {
        DoSomething();
        _isReady = false;
    }
}

// ✅ ĐÚNG — dùng UniTask
private async UniTask WaitAndDoAsync(CancellationToken ct)
{
    await UniTask.WaitUntil(() => _isReady, cancellationToken: ct);
    DoSomething();
}
```

```csharp
// ❌ SAI — dùng Update cho sequence với state machine thủ công
private int _state;
private float _timer;
private void Update()
{
    switch (_state)
    {
        case 0: Spawn(); _state = 1; _timer = 0.5f; break;
        case 1: _timer -= Time.deltaTime; if (_timer <= 0) { FadeIn(); _state = 2; } break;
        case 2: if (Input.GetMouseButtonDown(0)) { Finish(); _state = 3; } break;
    }
}

// ✅ ĐÚNG — dùng UniTask, code đọc như kịch bản
private async UniTask PlaySequenceAsync(CancellationToken ct)
{
    Spawn();
    await UniTask.Delay(500, cancellationToken: ct);
    FadeIn();
    await UniTask.WaitUntil(() => Input.GetMouseButtonDown(0), cancellationToken: ct);
    Finish();
}
```

```csharp
// ✅ ĐÚNG — Update cho di chuyển liên tục mỗi frame (không thay thế bằng UniTask)
private void Update()
{
    transform.position = Vector3.Lerp(transform.position, _target, Time.deltaTime * _speed);
}
```

**Code review rule:** Khi thấy `Update()`, `LateUpdate()`, hoặc `FixedUpdate()` trong PR — reviewer kiểm tra:
1. Logic này có **thực sự cần chạy mỗi frame** không?
2. Có thể chuyển sang UniTask (WaitUntil, Delay, sequence) không?
3. Nếu giữ Update → method có dưới 30 dòng không?

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

## 20. Branch Strategy

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

## 21. Project Setup Checklist

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
- [ ] Tạo data class đầu tiên: `ProgressData`, `EconomyData`
- [ ] Implement `UserDataService` (JsonConvert + PlayerPrefs, IDisposable)
- [ ] Đăng ký service trong `GameContext.RegisterServices()`
- [ ] Test save/load cycle: ghi data → tắt Play → bật Play → data còn nguyên

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

## Phase 7 — Scene & Level
- [ ] Implement `SceneService`
- [ ] Tạo scene `Home`, scene `Game`
- [ ] Thêm tất cả scene vào Build Settings
- [ ] Test flow: Bootstrap → Home → Game → ResultPopup → Home
- [ ] Implement `LevelManager`
- [ ] Tạo `AllLevelsSO`, thêm vào `GameContextSettings`
- [ ] Tạo `LevelDataSO` đầu tiên, kéo vào `AllLevelsSO` list
- [ ] Test: load level → gameplay nhận đúng data

## Phase 8 — Gameplay
- [ ] Implement gameplay controllers và elements
- [ ] Implement `ObstacleRegistry` nếu game có nhiều loại element
- [ ] Kết nối gameplay events (OnLevelComplete, OnLevelFail) với UI + save

## Phase 9 — Polish & Production
- [ ] Thêm Object Pooling cho high-frequency spawn/destroy
- [ ] Thêm `RemoteConfigService` khi cần A/B test
- [ ] Thêm DebugService (chỉ dev build)
- [ ] Audit naming convention + code standards
- [ ] Final test toàn bộ game flow
```

---

## 22. Những thứ KHÔNG làm

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
| Gọi `PlayerPrefs` trực tiếp từ code gameplay | Chỉ `UserDataService` được gọi `PlayerPrefs` |
| Gom tất cả data vào một class duy nhất | Chia class data riêng theo feature |
| Subscribe event bằng lambda | Method riêng — lambda không unsubscribe được |
| `Destroy(obj)` cho object spawn thường xuyên | Object pooling — `pool.Return(obj)` |
| UI script chứa business logic | UI chỉ hiển thị data và fire event |
| `SetActive(true/false)` cho show/hide UI | `CanvasGroup` alpha + interactable |
| `catch (Exception) { }` nuốt lỗi | Log error có context hoặc không catch |
| Xóa/rename field trong data class | Chỉ thêm mới, dùng `[JsonProperty]` nếu cần rename |
| Tạo thư mục script ngoài `Assets/Scripts/` | Tất cả code vào `Assets/Scripts/` |
| `Task.Run()` hoặc `System.Threading.Tasks` | Dùng UniTask — chạy trên main thread |
| `Update()` để chờ điều kiện hoặc chạy sequence | `UniTask.WaitUntil()`, `UniTask.Delay()`, async method |
| Gọi `SceneManager` trực tiếp từ gameplay code | Chỉ `SceneService` được gọi `SceneManager` |
| `LoadSceneMode.Single` | `LoadSceneMode.Additive` — Single sẽ destroy GameContext |
| Ghi runtime state vào LevelDataSO | Level SO chỉ chứa input — runtime state ở controller |
| Load level data mỗi lần chuyển level | Dùng `AllLevelsSO` — load 1 lần, truy cập O(1) |

---

## Tham chiếu nhanh — Service Registration Order

```csharp
// Thứ tự khuyến nghị: service nào không phụ thuộc gì → đăng ký trước
ServiceLocator.Add(new GameEventService());          // 1. Event bus — không phụ thuộc gì
ServiceLocator.Add(new UserDataService());           // 2. Data — không phụ thuộc gì
ServiceLocator.Add(new AudioService(config));        // 3. Audio — có thể phụ thuộc config
ServiceLocator.Add(new SceneService());              // 4. Scene — không phụ thuộc gì
ServiceLocator.Add(new LevelManager(Settings.AllLevels));// 5. Level — truy cập AllLevelsSO
ServiceLocator.Add(new UIService(screenRoot, popupRoot, Settings.UIPrefabs)); // 6. UI — prefab từ UIPrefabsSO
ServiceLocator.Add(new BoosterService());            // 7. Booster — dùng events và user data
ServiceLocator.Add(new AnalyticsService());          // 8. Analytics — dùng sau các service khác
ServiceLocator.Add(new RemoteConfigService(config)); // 9. Remote — IInitializable, override SO values
```

---

*Template này được tổng hợp từ kinh nghiệm thực chiến. Cập nhật version ở bảng đầu document khi có quyết định kiến trúc mới.*
