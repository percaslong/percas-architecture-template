# Unity Project Architecture Template
> The standard architecture for every new Unity project at Percas Studio.
> This is the single source of truth for structure, conventions, and architectural decisions.
> All new projects **must** follow this template from day one.

Unity Version Target: 2022.3.62f2

| Version | Date | Changes |
|---|---|---|
| v3.0 | 2026-04-13 | Restructured for new projects (removed legacy/refactor framing). UniTask installed via GitHub Release. |
| v2.0 | 2026-04-13 | Added Quick Start, Dependency Diagram, Service Lifecycle, UniTask Convention, Object Pooling, UI Architecture, Save/Load (JsonConvert), Error Handling, Testing & Debug. |
| v1.0 | — | Initial version. |
---

## Table of Contents

1. [Quick Start — From Zero to Running Game](#1-quick-start--from-zero-to-running-game)
2. [Dependency Diagram](#2-dependency-diagram)
3. [Folder Structure](#3-folder-structure)
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
22. [Things NOT to Do](#22-things-not-to-do)

---

## 1. Quick Start — From Zero to Running Game

> For new team members or when starting a new project. Follow these 5 steps, press Play, and the game runs.

### Step 1 — Create the `Bootstrap` Scene

1. File → New Scene → Save as `Bootstrap.scene`.
2. Set this scene as the **first** in Build Settings (index 0).
3. Create an empty GameObject named `GameContext`, attach the `GameContext.cs` script.

### Step 2 — Create Config SOs

1. Project window → Create → Config → `GameContextSettings`.
2. Create child SOs: `AudioConfigSO`, `GridConfigSO`, `BoosterConfigSO`, `PrefabReferencesSO`, ...
3. Drag child SOs into their corresponding slots on `GameContextSettings`.
4. Drag `GameContextSettings` into the `Settings` slot on the `GameContext` Inspector.

### Step 3 — Verify ServiceLocator

1. Press Play.
2. Open Console — no `KeyNotFoundException` = OK.
3. Try calling `ServiceLocator.Get<GameEventService>()` from any script → get an instance = OK.

### Step 4 — Create the `Game` Scene

1. Create the main gameplay scene.
2. Add the scene to Build Settings.
3. `GameContext.Start()` will automatically load the `Game` scene after initialization (see template in [Section 4](#4-bootstrap--service-locator)).
4. `GameContext` lives in the `Bootstrap` scene with `DontDestroyOnLoad` — other scenes contain gameplay only.

### Step 5 — Verify the flow

```
[Play] → Bootstrap scene load
       → GameContext.Awake() — singleton init
       → GameContext.Start() — RegisterServices() → InitializeAllAsync() → LoadSceneAsync("Game")
       → Game scene load
       → Gameplay scripts call ServiceLocator.Get<T>() → OK
```

**If you encounter errors:** see [Error Handling & Defensive Coding](#17-error-handling--defensive-coding).

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
│  → All call ServiceLocator.Get<T>() to access services    │
└─────────────────────────────────────────────────────────┘
```

**Arrow `──▶` = "subscribes to / uses events from"**. GameEventService is the hub; other services subscribe to events from it.

---

## 3. Folder Structure

```
Assets/
├── Scripts/                        ← ALL CODE GOES HERE
│   ├── Bootstrap/                  GameContext, ServiceLocator, IInitializable
│   ├── Services/                   Services (Audio, UserData, UI, ...)
│   │   ├── Audio/
│   │   ├── UserData/               UserDataService, data classes (ProgressData, EconomyData, ...)
│   │   ├── UI/                     UIService, BaseScreen, BasePopup
│   │   ├── Scene/                  SceneService
│   │   ├── Level/                  LevelManager
│   │   ├── RemoteConfig/           RemoteConfigService
│   │   └── ...
│   ├── GamePlay/                   Main gameplay code
│   │   ├── Controller/
│   │   ├── Element/
│   │   └── ...
│   ├── Config/                     ScriptableObject config classes (AudioConfigSO, UIPrefabsSO, AllLevelsSO, ...)
│   ├── Events/                     GameEventService
│   ├── Analytics/                  AnalyticsService, ITrackingEvent, event classes
│   ├── Booster/                    BoosterCommand, BoosterService
│   ├── Obstacles/                  IObstacleElement, ObstacleRegistry, ObstacleDefinition
│   ├── Pool/                       ObjectPool<T>, PoolManager
│   └── Utils.cs                    ← ALL utility/helper functions go here (single file)
│
├── Plugins/                        ← Imported .unitypackage packages (UniTask, DOTween, ...)
│
└── Resources/                      ← Only for assets that need dynamic loading via Resources.Load (minimize usage)
```

**Principles:**
- All code lives in `Assets/Scripts/` — do not create script folders elsewhere.
- Folders must match namespaces (see [Code Standards](#19-code-standards)).
- Do not create empty folders — only create when there is a file to put in.

---

## 4. Bootstrap & Service Locator

### Architectural Decisions
- **Only one singleton:** `GameContext` — MonoBehaviour, `DontDestroyOnLoad`.
- All services are registered through a ServiceLocator.
- Do not use `.instance` on individual services.
- Access services anywhere: `ServiceLocator.Get<AudioService>()`.

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

        /// <summary>Settings SO containing all config references.</summary>
        [field: SerializeField] public GameContextSettings Settings { get; private set; }
        #endregion

        #region Fields
        [Header("UI Roots — drag Transform from Canvas in Bootstrap scene")]
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
                // All services ready — load game scene
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
            // Register in dependency order — services that depend on others register later
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

        /// <summary>Register service. Only call from GameContext.RegisterServices().</summary>
        public static void Add<T>(T service) where T : class
        {
            _services[typeof(T)] = service;
            _registrationOrder.Add(service);
        }

        /// <summary>Get service. Throws if not registered.</summary>
        public static T Get<T>() where T : class
        {
            if (_services.TryGetValue(typeof(T), out var service))
                return (T)service;
            throw new KeyNotFoundException($"Service {typeof(T).Name} chưa được đăng ký.");
        }

        /// <summary>Get service safely — for optional services (DebugService, ...). </summary>
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

        /// <summary>Call InitializeAsync on all IInitializable services, in registration order.</summary>
        public static async UniTask InitializeAllAsync()
        {
            foreach (var service in _registrationOrder)
            {
                if (service is IInitializable initializable)
                    await initializable.InitializeAsync();
            }
        }

        /// <summary>Call Dispose on all IDisposable services, in reverse order.</summary>
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

### Accessing services in game code

```csharp
// Access services from any MonoBehaviour
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

### Architectural Decisions
- Services that need async init (load data, wait for SDK) → implement `IInitializable`.
- Services that need cleanup (save data, flush analytics) → implement `System.IDisposable`.
- `GameContext` calls init in dependency order, calls dispose on `OnApplicationQuit`.

### Template: IInitializable.cs

```csharp
// Assets/Scripts/Bootstrap/IInitializable.cs
using Cysharp.Threading.Tasks;

namespace YourGame.Bootstrap
{
    /// <summary>Services needing async init implement this interface.</summary>
    public interface IInitializable
    {
        /// <summary>Async init. GameContext calls in registration order.</summary>
        UniTask InitializeAsync();
    }
}
```

### Example: Service using IInitializable and IDisposable

```csharp
public class UserDataService : System.IDisposable
{
    public UserDataService()
    {
        // Load from PlayerPrefs — synchronous, no need for IInitializable
        LoadAll();
    }

    public void Dispose()
    {
        // Save data before game quits
        Save();
    }
}
```

**Principles:**
- Not every service needs to implement these — only those that **need** them.
- Init runs **sequentially** in registration order — ensures dependency order.
- Dispose runs in **reverse** — services registered last (most dependencies) clean up first.

---

## 6. Async Convention — UniTask

### Architectural Decisions
- **Use UniTask for all async** — no Coroutine, no `System.Threading.Tasks.Task`.
- **No Coroutine** — even for simple logic. UniTask covers all use cases.
- **Never** use `async void` — except for Unity callbacks (`Start`, `OnDestroy`, ...).

### Package Setup

1. Go to [https://github.com/Cysharp/UniTask/releases](https://github.com/Cysharp/UniTask/releases).
2. Download the `.unitypackage` of the **latest version**.
3. In Unity: Assets → Import Package → Custom Package → select the downloaded file.
4. Import all → UniTask will be in `Assets/Plugins/UniTask/`.

> **Note:** Do not use `Add package by name` since UniTask is not on the official Unity Registry.
> To update: download the latest `.unitypackage` from GitHub and import over the existing one.

### Specific Rules

| Scenario | Approach | Example |
|---|---|---|
| Async method returning a value | `UniTask<T>` | `async UniTask<int> LoadLevel()` |
| Async method returning nothing | `UniTask` | `async UniTask ShowPopup()` |
| Unity callback needing async | `async void` + try/catch | `async void Start()` |
| Wait one frame | `await UniTask.Yield()` | Replaces `yield return null` |
| Wait for time | `await UniTask.Delay(ms)` | Replaces `yield return new WaitForSeconds` |
| Wait for condition | `await UniTask.WaitUntil(() => condition)` | Replaces `yield return new WaitUntil` |
| Cancel task on destroy | `CancellationToken` from `this.GetCancellationTokenOnDestroy()` | See template below |
| Load scene | `.ToUniTask()` | `SceneManager.LoadSceneAsync("X").ToUniTask()` |

### Template: Async in MonoBehaviour

```csharp
public class LevelLoader : MonoBehaviour
{
    #region Unity Callbacks
    private async void Start()
    {
        // async void ONLY in Unity callbacks — always wrap with try/catch
        try
        {
            await LoadLevelAsync(this.GetCancellationTokenOnDestroy());
        }
        catch (OperationCanceledException)
        {
            // GameObject was destroyed mid-operation — normal, don't log error
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

### Template: Async in Service (non-MonoBehaviour)

```csharp
public class RemoteConfigService : IInitializable
{
    #region Fields
    private Dictionary<string, string> _config;
    #endregion

    #region IInitializable
    public async UniTask InitializeAsync()
    {
        // Service doesn't have GetCancellationTokenOnDestroy()
        // Use timeout instead of CancellationToken if needed
        var (isCanceled, result) = await FetchRemoteConfig()
            .TimeoutWithoutException(TimeSpan.FromSeconds(10));

        _config = isCanceled
            ? LoadDefaultConfig()   // Timeout → use default config
            : result;
    }
    #endregion
}
```

### Things NOT to do with async

| Don't | Do this instead |
|---|---|
| `async void` in regular methods | `async UniTask` |
| `StartCoroutine(DoSomething())` | `DoSomethingAsync().Forget()` — Coroutine is banned |
| `yield return new WaitForSeconds(1f)` | `await UniTask.Delay(1000, cancellationToken: ct)` — no yield |
| Forgetting CancellationToken in MonoBehaviour | Always use `this.GetCancellationTokenOnDestroy()` |
| `Task.Run(() => ...)` | UniTask already runs on main thread, no need for thread pool |
| Uncontrolled fire-and-forget | Use `.Forget()` intentionally, or `try/catch` in `async void` |

---

## 7. ScriptableObject Configs

### Architectural Decisions
- Each service has its own config SO.
- **Do not** use serialized fields on MonoBehaviour — all config belongs in SOs.
- SOs only contain **default values** — clean, simple, no remote keys.
- Remote Config overrides are handled centrally in `RemoteConfigService` — SOs know nothing about remote.
- `GameContextSettings` is the central SO, containing references to all other config SOs.

### SO Pattern — default values only

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

### Remote Config — Centralized Override

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

            // Add overrides for new configs here — one single place
        }

        /// <summary>Get remote value. If key doesn't exist, returns defaultValue.</summary>
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

**Why this approach:**
- SOs keep their role: **default values, tunable in Editor**. No bloat from remote key fields.
- When adding 50 config variables, just add 50 override lines in **one single file** — no scattering remote keys across SOs.
- If remote config isn't needed → delete `RemoteConfigService`, SOs still work normally.
- Easy debugging: open SO in Inspector to see current values (whether overridden or not).

### GameContextSettings — Central SO

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
        // ... add other SOs
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
        // PascalCase naming, use [field: SerializeField]
        [field: SerializeField] public GameObject BasketElement  { get; private set; }
        [field: SerializeField] public GameObject WallElement    { get; private set; }
        // ... continue
        #endregion
    }
}

// Access:
// GameContext.Instance.Settings.Prefabs.BasketElement
```

---

## 8. Event System

### Architectural Decisions
- `GameEventService` is a plain class, no interface.
- All delegates become public properties (PascalCase, action-oriented).
- No `static` event bus.
- `GameStateManager` is a separate state machine — do not merge into event system.

### Template: GameEventService.cs

```csharp
// Assets/Scripts/Events/GameEventService.cs
namespace YourGame.Events
{
    public class GameEventService
    {
        #region Gameplay Events
        /// <summary>Fired when player selects a cell on the grid.</summary>
        public Action<SlotOnElement> OnCellChoose { get; set; }

        /// <summary>Fired when level is completed.</summary>
        public Action OnLevelComplete { get; set; }

        /// <summary>Fired when player loses.</summary>
        public Action OnLevelFail { get; set; }
        #endregion

        #region UI Events
        public Action<int> OnCoinChanged  { get; set; }
        public Action<int> OnLivesChanged { get; set; }
        #endregion
    }
}
```

### Usage

```csharp
// Subscribe
_events.OnLevelComplete += HandleLevelComplete;

// Unsubscribe (always unsubscribe on destroy)
private void OnDestroy()
{
    _events.OnLevelComplete -= HandleLevelComplete;
}

// Fire
_events.OnLevelComplete?.Invoke();
```

### ⚠️ Event Safety — Avoiding Memory Leaks

Delegate-based event systems are the **most common** source of memory leaks. Follow this checklist:

**Mandatory checklist when subscribing to events:**

1. **Subscribe in `Start()` or `OnEnable()`** — do not subscribe in `Awake()` (services may not be ready).
2. **Unsubscribe in `OnDestroy()`** — mandatory, no exceptions.
3. **Handler must be a named method, not a lambda** — lambdas cannot be unsubscribed.
4. **Code review must check:** every `+=` must have a corresponding `-=`.

```csharp
// ✅ CORRECT — easy to unsubscribe
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

// ❌ WRONG — cannot unsubscribe lambda
_events.OnLevelComplete += () => Debug.Log("Done");
```

---

## 9. Booster / Command Pattern

### Architectural Decisions
- All booster logic lives in `BoosterService` + individual `BoosterCommand` subclasses.
- Controllers only call `BoosterService.Execute(type, context)` — no knowledge of internal logic.
- Adding a new booster = adding a new `BoosterCommand` subclass, no existing code modified.

### Template

```csharp
// Assets/Scripts/Booster/BoosterCommand.cs
namespace YourGame.Booster
{
    public abstract class BoosterCommand
    {
        /// <summary>Execute booster. Override in each subclass.</summary>
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
            // Sand magnet logic
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
        /// <summary>Execute booster by type.</summary>
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

### Architectural Decisions
- Only `AnalyticsService` calls tracking SDKs directly.
- All other classes only call `AnalyticsService.Track(ITrackingEvent)`.
- Event names are constants, no inline string literals.

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
        /// <summary>Send tracking event. This is the only point that calls the SDK.</summary>
        public void Track(ITrackingEvent trackingEvent)
        {
            // YourSDK.TrackEvent(trackingEvent.EventName, trackingEvent.Properties);
        }
        #endregion
    }
}

// Usage from anywhere:
_analytics.Track(new LevelCompleteEvent(level: 5, moves: 30, duration: 120f));
```

---

## 11. Obstacle / Registry Pattern

Use when the game has many object/element types that spawn dynamically and you want to add new types **without modifying existing code**.

### Template: Interface

```csharp
// Assets/Scripts/Obstacles/IObstacleElement.cs
namespace YourGame.Obstacles
{
    public interface IObstacleElement
    {
        /// <summary>Initial setup from cell data.</summary>
        void SpawnFromCellData(CellData cell, GridCell gridCell);

        /// <summary>Called after the entire grid has initialized — used for cross-referencing.</summary>
        void OnPostInit(SlotsController controller);

        /// <summary>True if the obstacle occupies a cell slot on the grid.</summary>
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

### Checklist for adding a new obstacle (after the system is live)

1. Add a value to the `ElementType` enum — **never** reuse or renumber existing values.
2. Create `[Name]Element.cs` implementing `IObstacleElement`.
3. Create a prefab in Unity.
4. Create an `ObstacleDefinition` asset, fill in type / prefab / icon.
5. Add the definition to the `ObstacleRegistry` asset in the Inspector.

**Done. No other files are modified.**

---

## 12. Object Pooling

### Architectural Decisions
- Any object spawned/destroyed **more than 10 times per second** → use pool.
- Use generic `ObjectPool<T>` — one pool per prefab type.
- `PoolManager` is a service, registered in `ServiceLocator`.
- Pooled objects **must** implement `IPoolable` to reset state when returned to pool.

### Template: IPoolable

```csharp
// Assets/Scripts/Pool/IPoolable.cs
namespace YourGame.Pool
{
    public interface IPoolable
    {
        /// <summary>Reset state to default when returned to pool.</summary>
        void OnReturnToPool();

        /// <summary>Setup when taken from pool.</summary>
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

### Usage

```csharp
// Create pool
var sandPool = new ObjectPool<SandElement>(sandPrefab, poolParent, prewarmCount: 20);

// Get from pool
var sand = sandPool.Get(spawnPos, Quaternion.identity);

// Return to pool (instead of Destroy)
sandPool.Return(sand);
```

### When NOT to use pooling
- Objects created only once (UI popup, singleton).
- Objects with complex lifecycles that need individual tracking.
- Prototype phase — only add pooling when profiler confirms GC spikes.

---

## 13. UI Architecture

### Architectural Decisions
- `UIService` manages all Screens and Popups — the only entry point to show/hide UI.
- **Screen** = full screen, only 1 active at a time (Home, Game, Result).
- **Popup** = overlay, can stack multiple popups simultaneously.
- Show/Hide uses `CanvasGroup` (fade alpha) — **do not** use `SetActive` directly on root.
- Each screen/popup is a separate prefab, loaded on-demand.

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
        /// <summary>Called when screen is shown. Override to populate data.</summary>
        public virtual void OnShow()
        {
            _canvasGroup.alpha = 1f;
            _canvasGroup.interactable = true;
            _canvasGroup.blocksRaycasts = true;
        }

        /// <summary>Called when screen is hidden. Override to cleanup.</summary>
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

        /// <summary>Called when user taps Close or back button.</summary>
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
        // Add new screen/popup → add 1 field here
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
        /// <summary>Switch screen. Hide old screen, show new one.</summary>
        public T ShowScreen<T>() where T : BaseScreen
        {
            _currentScreen?.OnHide();
            var screen = GetOrCreate<T>(_screenRoot);
            screen.OnShow();
            _currentScreen = screen;
            return screen;
        }

        /// <summary>Show popup, push onto stack.</summary>
        public T ShowPopup<T>() where T : BasePopup
        {
            var popup = GetOrCreate<T>(_popupRoot);
            popup.OnShow();
            _popupStack.Push(popup);
            return popup;
        }

        /// <summary>Close a specific popup.</summary>
        public void ClosePopup(BasePopup popup)
        {
            popup.OnHide();
            // Rebuild stack if popup is not top — but usually is
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

        /// <summary>Build type → prefab mapping from SO. Runs once in constructor.</summary>
        private Dictionary<Type, MonoBehaviour> BuildPrefabMap(UIPrefabsSO prefabs)
        {
            return new Dictionary<Type, MonoBehaviour>
            {
                { typeof(HomeScreen),     prefabs.homeScreen },
                { typeof(GameScreen),     prefabs.gameScreen },
                { typeof(ResultPopup),    prefabs.resultPopup },
                { typeof(PurchasePopup),  prefabs.purchasePopup },
                { typeof(SettingsPopup),  prefabs.settingsPopup },
                // Add mapping for new screen/popup here
            };
        }
        #endregion
    }
}
```

### Adding a new Screen / Popup — Checklist

1. Create a new class inheriting from `BaseScreen` or `BasePopup`.
2. Create a prefab, attach the script.
3. Add **1 field** to `UIPrefabsSO`.
4. Add **1 line** to `BuildPrefabMap()` in `UIService`.
5. Drag the prefab into the slot in `UIPrefabsSO` Inspector.

**Done. If step 5 is missed, Inspector will show an empty slot — visible immediately, no runtime surprises.**

### UI Principles

- UI scripts **contain no business logic** — only display data and fire events.
- Data is passed to UI via `OnShow(data)` method — UI does not fetch its own data.
- Animation/tween uses DOTween or UniTask delay — do not use Animation Controller for simple UI.

### Example: Passing data to UI via OnShow(data)

```csharp
// ===== DATA MODEL — struct containing exactly what the UI needs to display =====

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

// ===== SCREEN — only receives data and displays, never queries on its own =====

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
        /// <summary>Show screen with specific data. Caller prepares data, screen only displays.</summary>
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

// ===== CALLER — prepares data then passes to UI =====

// In GameController or anywhere:
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
// ===== POPUP WITH DATA — example: booster purchase confirmation popup =====

// Assets/Scripts/Services/UI/Popups/PurchasePopupData.cs
namespace YourGame.Services.UI
{
    public struct PurchasePopupData
    {
        public string itemName;
        public int cost;
        public int currentCoins;
        public System.Action onConfirm; // callback when user taps Buy
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

            // Disable Buy button if not enough coins
            bool canAfford = data.currentCoins >= data.cost;
            _buyButton.interactable = canAfford;
            _buyButtonText.text = canAfford ? "Mua" : "Không đủ";
        }
        #endregion

        #region UI Callbacks — assigned in Inspector
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

**Why pass data via OnShow instead of UI fetching its own:**
- UI has no service dependencies — easy to test, easy to reuse.
- Caller controls what data is displayed — the same popup can show different data.
- When reading code, the caller shows exactly what the popup will display — no need to open the popup script.

---

## 14. Save / Load — Data Persistence

### Architectural Decisions
- Serialize/deserialize using `Newtonsoft.Json` (`JsonConvert`), stored in `PlayerPrefs`.
- **Each feature has its own data class** — `ProgressData`, `EconomyData`, `LivesData`, ... Do not combine everything into one class.
- Each data class is stored under a separate `PlayerPrefs` key.
- `UserDataService` is the only entry point for reading/writing all data — no other class calls `PlayerPrefs` directly.
- Auto-save at important milestones (level complete, purchase, ...).

### Package Setup

```
// Package Manager → Add package by name:
com.unity.nuget.newtonsoft-json
```

### Template: Data class per feature

```csharp
// Assets/Scripts/Services/UserData/ProgressData.cs
namespace YourGame.Services.UserData
{
    /// <summary>Player progress data.</summary>
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
    /// <summary>Currency and inventory data.</summary>
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
    /// <summary>Lives system data.</summary>
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
    /// <summary>User settings data.</summary>
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
        // Add new key for each new feature data class
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
        /// <summary>Save all data. Call at important milestones.</summary>
        public void Save()
        {
            SaveEntry(KEY_PROGRESS, Progress);
            SaveEntry(KEY_ECONOMY, Economy);
            SaveEntry(KEY_LIVES, Lives);
            SaveEntry(KEY_SETTINGS, Settings);
            PlayerPrefs.Save();
        }

        /// <summary>Save a specific feature data — use when only 1 feature changed.</summary>
        public void SaveProgress() { SaveEntry(KEY_PROGRESS, Progress); PlayerPrefs.Save(); }
        public void SaveEconomy()  { SaveEntry(KEY_ECONOMY, Economy);   PlayerPrefs.Save(); }
        public void SaveLives()    { SaveEntry(KEY_LIVES, Lives);       PlayerPrefs.Save(); }
        public void SaveSettings() { SaveEntry(KEY_SETTINGS, Settings); PlayerPrefs.Save(); }

        /// <summary>Delete all data — for debug or account reset.</summary>
        public void DeleteAll()
        {
            PlayerPrefs.DeleteAll();
            LoadAll(); // Reload default values
        }

        /// <summary>Export all data to JSON — for debugging in Console.</summary>
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
            Save(); // Auto-save when game quits
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

### Usage

```csharp
// Get service
var userData = ServiceLocator.Get<UserDataService>();

// Read/write data
userData.Economy.coins += 100;
userData.SaveEconomy(); // Save only economy — fast, clean

// Or save everything
userData.Progress.currentLevel = 5;
userData.Economy.gems += 10;
userData.Save(); // Save all

// Debug — print all data to Console
Debug.Log(userData.ExportAllToJson());
```

### Adding new feature data — Checklist

1. Create new data class: `XxxData.cs` in `Assets/Scripts/Services/UserData/`.
2. Add `const string KEY_XXX` to `UserDataService`.
3. Add property `public XxxData Xxx { get; private set; }`.
4. Add `LoadEntry` in `LoadAll()`, add `SaveEntry` in `Save()`.
5. Add method `SaveXxx()` for individual save.
6. Add to `ExportAllToJson()`.

**Done. No other classes are affected.**

### Schema — Principles

- **Never delete fields** in shipped data classes — only add new ones.
- **Do not rename fields** — `JsonConvert` will lose data if names change.
- If renaming is unavoidable: use `[JsonProperty("old_name")]` attribute.
- `JsonConvert.DeserializeObject` automatically assigns default values for new fields → adding new fields needs no migration.
- **Do not change PlayerPrefs keys** — the key is the identity of the data block.

### When to call Save()

```csharp
// ✅ Call Save or SaveXxx() at milestones — use named methods, not lambdas
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

// ❌ Do NOT call Save() every frame or on every minor action
// ❌ Do NOT call Save() in Update()
```

---

## 15. Scene Management

### Architectural Decisions
- **Bootstrap scene** (index 0) contains `GameContext` — always loaded first, never unloaded.
- Other scenes load in **Additive mode** — `GameContext` persists throughout.
- `SceneService` manages load/unload — no other class calls `SceneManager` directly.
- Transitions (fade in/out) are handled by `SceneService` — gameplay code knows nothing about transitions.

### Scene Flow for Hybrid Puzzle

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

**Note:** Result is a **popup** (see [Section 13 — UI Architecture](#13-ui-architecture)), not a separate scene. The popup overlays the Game scene — players can still see the board behind it.

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
        /// <summary>Load new scene, unload old scene. Includes fade transition.</summary>
        public async UniTask LoadScene(string sceneName, CancellationToken ct = default)
        {
            // Fade out
            await FadeOut(ct);

            // Unload old scene if exists
            if (!string.IsNullOrEmpty(_currentScene))
                await SceneManager.UnloadSceneAsync(_currentScene).ToUniTask(cancellationToken: ct);

            // Load new scene
            await SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive).ToUniTask(cancellationToken: ct);
            _currentScene = sceneName;

            // Fade in
            await FadeIn(ct);
        }
        #endregion

        #region Private Methods
        private async UniTask FadeOut(CancellationToken ct)
        {
            // TODO: CanvasGroup fade or Animator
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

### Principles

- **Do not call `SceneManager` directly** from gameplay code — always go through `SceneService`.
- **Do not use `LoadSceneMode.Single`** — it will destroy `GameContext`.
- Each scene is self-contained: has its own Canvas, its own Camera (if needed), EventSystem only in Bootstrap.
- Scene names use `const string` — no inline literals.

---

## 16. Level Data Architecture

### Architectural Decisions
- Each level is a **separate ScriptableObject** (`LevelDataSO`) — edited independently in Inspector, Git-friendly.
- **`AllLevelsSO`** contains `List<LevelDataSO>` — a list of references to all level SOs, O(1) access by index.
- `LevelManager` is a service, registered in `ServiceLocator`, receives `AllLevelsSO` via constructor.
- Level data only contains **input** (level configuration) — no runtime state (progress, score).
- Runtime state belongs to controllers — **never write back to SO**.

### Template: LevelDataSO

```csharp
// Assets/Scripts/Config/LevelDataSO.cs
using UnityEngine;

namespace YourGame.Config
{
    /// <summary>Configuration for 1 level. Each level is a separate SO asset.</summary>
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

### Template: AllLevelsSO — List of References

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
        /// <summary>Get level by number. O(1) — list access by index.</summary>
        public LevelDataSO GetLevel(int levelNumber)
        {
            // Convention: levelNumber = index + 1 (level 1 at index 0)
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
        /// <summary>Load level by number. Direct list access — very fast.</summary>
        public LevelDataSO LoadLevel(int levelNumber)
        {
            CurrentLevel = _allLevels.GetLevel(levelNumber);
            if (CurrentLevel == null)
                Debug.LogError($"[LevelManager] Level {levelNumber} not found. Total: {TotalLevels}");
            return CurrentLevel;
        }

        /// <summary>Load next level based on current progress.</summary>
        public LevelDataSO LoadNextLevel()
        {
            var userData = ServiceLocator.Get<UserDataService>();
            return LoadLevel(userData.Progress.currentLevel);
        }

        /// <summary>Check if level exists.</summary>
        public bool HasLevel(int levelNumber) => _allLevels.HasLevel(levelNumber);
        #endregion
    }
}

// Register in GameContext.RegisterServices():
// ServiceLocator.Add(new LevelManager(Settings.AllLevels));
```

### Connect to GameContextSettings

```csharp
// Add to GameContextSettings
[field: SerializeField] public AllLevelsSO AllLevels { get; private set; }
```

### Level SO Organization Convention

```
Assets/
└── ScriptableObjects/
    └── Levels/
        ├── Level_00001.asset      ← each level is a separate SO file
        ├── Level_00002.asset
        ├── Level_00003.asset
        ├── ...
        └── AllLevels.asset        ← SO containing references to all levels above
```

- File name: `Level_XXXXX.asset` — **5-digit** padding (supports up to 99,999 levels).
- Create new level: Project window → Create → Levels → LevelData → edit in Inspector.
- After creation, **drag SO into the `AllLevels` list** in Inspector — at the correct position by level order.

### Why use separate SOs + AllLevelsSO references

| Criteria | Separate SO + AllLevelsSO | Plain class in 1 SO | Separate JSON files |
|---|---|---|---|
| Edit in Inspector | ✅ Each level editable independently | ❌ List too long, hard to navigate | ❌ No Inspector |
| Git conflict | ✅ Separate files, less conflict | ❌ 1 file, easy conflict | ✅ Separate files |
| Access speed | ✅ O(1) via AllLevelsSO list | ✅ O(1) | ❌ I/O per load |
| Create new level | ✅ Create SO → drag into list | Add entry to long list | Create JSON file |
| Quick preview | ✅ Click SO → see config immediately | Scroll list to find entry | Open text file |

### Level Editor Tool — Recommended

When the game has thousands of levels, build a custom Editor tool to support:

1. **Auto-populate**: scan the `Levels/` folder → automatically add new SOs to the `AllLevelsSO` list in correct order.
2. **Batch create**: create multiple level SOs at once from a template.
3. **Visual Editor**: visual UI for designers to place elements on the grid.

This tool lives in `Assets/Editor/` — not shipped with the game.

---

## 17. Error Handling & Defensive Coding

### ServiceLocator — When errors occur and how to debug

**Most common error:** `KeyNotFoundException: Service XxxService chưa được đăng ký.`

**Common causes:**

1. **Calling `ServiceLocator.Get<T>()` in `Awake()`** — at this point `GameContext.Start()` hasn't finished.
   → Fix: move to `Start()` or later.

2. **Forgot to register service** in `GameContext.RegisterServices()`.
   → Fix: add the `ServiceLocator.Add(...)` line.

3. **Wrong registration order** — service A depends on service B but A is registered before B.
   → Fix: see [Quick Reference — Service Registration Order](#quick-reference--service-registration-order).

### TryGet — Safe Version

`ServiceLocator.TryGet<T>()` is already included in the template (see [Section 4](#4-bootstrap--service-locator)).

**When to use `TryGet` vs `Get`:**
- `Get<T>()` — when the service **must exist** (99% of cases). Crash early = easy debug.
- `TryGet<T>()` — when the service is **optional** (e.g., DebugService only in dev builds).

### Principles chung

- **Fail fast, fail loud** — let errors crash early instead of running silently wrong.
- **Do not catch and swallow exceptions** — `catch (Exception) { }` is banned.
- **Log with context** — `Debug.LogError($"[ClassName] Error description: {detail}")`.
- **Null check only when justified** — don't spam `if (x != null)` everywhere. If x must always be non-null, let the NullReferenceException crash and fix the root cause.

---

## 18. Testing & Debug

### Architectural Decisions
- **Unit tests are not mandatory for gameplay** — gameplay changes too fast during prototype phase.
- **Manual testing is mandatory** per Acceptance Checklist (see each PRD).
- **Unit tests are encouraged for Services** — ServiceLocator pattern makes mocking easy.

### Debug Tools — Convention

```csharp
// Assets/Scripts/Bootstrap/DebugService.cs
namespace YourGame.Bootstrap
{
    /// <summary>
    /// Service that only exists in Development builds.
    /// Provides shortcuts for quick testing.
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

// Conditional registration in GameContext:
#if UNITY_EDITOR || DEVELOPMENT_BUILD
    ServiceLocator.Add(new DebugService());
#endif
```

### Mocking Services for Unit Tests

```csharp
// Example: test BoosterService without running the game
[Test]
public void MagnetBooster_ShouldExecuteWithoutError()
{
    // Arrange — register mock services
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

// Do NOT log excessively in Update/FixedUpdate — causes lag
// Do NOT log sensitive data (token, password) — even in dev builds
```

---

## 19. Code Standards

### Namespaces — mandatory

```csharp
// File: Assets/Scripts/Services/Audio/AudioService.cs
namespace YourGame.Services.Audio { ... }

// File: Assets/Scripts/GamePlay/Element/Basket/BasketElement.cs
namespace YourGame.GamePlay.Element.Basket { ... }
```

Namespaces must match the folder structure.

### Regions — mandatory

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

Only include regions that have content.

### Naming Convention

| Type | Convention | Example |
|---|---|---|
| Class / Interface / SO | PascalCase | `AudioService`, `ITrackingEvent` |
| Public property | PascalCase | `public float Volume { get; set; }` |
| Private field | _camelCase | `private float _volume;` |
| Constant | UPPER_SNAKE | `private const string EVENT_NAME = "level_complete";` |
| Method | PascalCase | `public void PlaySound()` |
| Event property | PascalCase, action-oriented | `OnLevelComplete`, `OnCellChoose` |

### Comments

```csharp
// ✅ Comments explain WHY, not WHAT
// Wait 1 frame to ensure all Awake() have finished before subscribing to events
await UniTask.Yield();

// ❌ Redundant comment, just restates the code
// Assign volume value to _volume variable
_volume = value;
```

- All comments in **English**.
- All public methods, properties, and important fields need XML doc comments (`/// <summary>`).
- Do not commit commented-out code.

### Complexity Limits

- Methods longer than **~30 lines** → doing too much, split them.
- Prefer flat, clear code over clever, deeply nested code.
- Do not over-engineer. Solve the current problem, not imaginary ones.
- Interfaces are only worth creating when **two actual implementations exist**.

### Update() vs UniTask — When to use which

> **Principle:** Minimize `Update()` — only use when logic **truly needs to run every frame continuously and unconditionally**. Most code that looks like Update is actually "waiting for a condition" or "running a sequence" — use UniTask for those cases.

| Scenario | Use | Reason |
|---|---|---|
| Wait for condition (animation done, data ready, ...) | `await UniTask.WaitUntil()` | Clear, auto-cleanup on destroy |
| Timer / countdown / delay | `await UniTask.Delay()` | No need to accumulate deltaTime manually |
| Sequence (spawn → wait → fade → done) | `async UniTask` method | Sequential code, easy to read and debug |
| One-shot reaction (wait → do 1 thing) | `await UniTask.WaitUntil()` | No state variables needed, no loop |
| Continuous movement / lerp every frame | `Update()` | By nature — runs every frame unconditionally |
| Continuous input (drag, swipe, multi-touch) | `Update()` | Need to read input state every frame |
| Camera follow | `LateUpdate()` | Unity convention — runs after all Updates |
| Physics (force, velocity, continuous raycast) | `FixedUpdate()` | Unity physics step |

**Examples — wrong vs right:**

```csharp
// ❌ WRONG — using Update to wait for a condition
private bool _isReady;
private void Update()
{
    if (_isReady)
    {
        DoSomething();
        _isReady = false;
    }
}

// ✅ CORRECT — using UniTask
private async UniTask WaitAndDoAsync(CancellationToken ct)
{
    await UniTask.WaitUntil(() => _isReady, cancellationToken: ct);
    DoSomething();
}
```

```csharp
// ❌ WRONG — using Update for sequence with manual state machine
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

// ✅ CORRECT — using UniTask, code đọc như kịch bản
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
// ✅ CORRECT — Update for continuous per-frame movement (do not replace with UniTask)
private void Update()
{
    transform.position = Vector3.Lerp(transform.position, _target, Time.deltaTime * _speed);
}
```

**Code review rule:** When you see `Update()`, `LateUpdate()`, or `FixedUpdate()` in a PR — reviewer checks:
1. Does this logic **truly need to run every frame**?
2. Can it be converted to UniTask (WaitUntil, Delay, sequence)?
3. If keeping Update → is the method under 30 lines?

### Utils

```csharp
// Assets/Scripts/Utils.cs — ONE single file, ONE static class
namespace YourGame
{
    public static class Utils
    {
        /// <summary>Clamp giá trị về range [0, 1].</summary>
        public static float Clamp01(float value) => Mathf.Clamp(value, 0f, 1f);

        // All helper functions go here
        // Do not create separate utility classes
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

**Workflow:**
- Create branch `feature/xxx` from `dev` for each feature or task.
- Code, test on feature branch → create Pull Request to `dev`.
- Code review is mandatory before merging.
- When `dev` is stable with enough features for a build → merge into `main`.
- Hotfix: create branch `hotfix/xxx` from `main`, after fix merge into both `main` and `dev`.

---

## 21. Project Setup Checklist

Use this checklist when starting a new project. Follow the order from top to bottom.

```markdown
## Phase 1 — Project & Package Setup
- [ ] Create new Unity project (2022.3 LTS, URP)
- [ ] Import UniTask from GitHub Release (.unitypackage)
- [ ] Import Newtonsoft.Json (`com.unity.nuget.newtonsoft-json`)
- [ ] Create folder structure per Section 3
- [ ] Setup .gitignore for Unity
- [ ] Init Git repo, create `main` and `dev` branches

## Phase 2 — Bootstrap & Core Services
- [ ] Create `Bootstrap` scene, set as index 0 in Build Settings
- [ ] Implement `GameContext.cs` (singleton, DontDestroyOnLoad)
- [ ] Implement `ServiceLocator.cs` với Add/Get/TryGet/InitializeAllAsync/DisposeAll
- [ ] Implement `IInitializable.cs`
- [ ] Create `GameContextSettings` SO
- [ ] Validate: press Play → no errors in Console

## Phase 3 — Event System & Data
- [ ] Implement `GameEventService`
- [ ] Create first data classes: `ProgressData`, `EconomyData`
- [ ] Implement `UserDataService` (JsonConvert + PlayerPrefs, IDisposable)
- [ ] Register services in `GameContext.RegisterServices()`
- [ ] Test save/load cycle: write data → stop Play → start Play → data persists

## Phase 4 — Config SOs
- [ ] Create config SO for each service that needs config (Audio, Grid, Booster, ...)
- [ ] Tạo `PrefabReferencesSO`
- [ ] Drag all SOs into `GameContextSettings` Inspector
- [ ] Validate: services receive correct config on init

## Phase 5 — UI Foundation
- [ ] Implement `BaseScreen`, `BasePopup`
- [ ] Implement `UIService`
- [ ] Create first screen (HomeScreen or GameScreen)
- [ ] Validate: show/hide screen works correctly

## Phase 6 — Analytics & Booster
- [ ] Implement `AnalyticsService` + `ITrackingEvent`
- [ ] Implement `BoosterService` + `BoosterCommand` base class
- [ ] Create first event class (LevelCompleteEvent)
- [ ] Validate: track event logs to Console

## Phase 7 — Scene & Level
- [ ] Implement `SceneService`
- [ ] Create `Home` scene, `Game` scene
- [ ] Add all scenes to Build Settings
- [ ] Test flow: Bootstrap → Home → Game → ResultPopup → Home
- [ ] Implement `LevelManager`
- [ ] Create `AllLevelsSO`, add to `GameContextSettings`
- [ ] Create first `LevelDataSO`, drag into `AllLevelsSO` list
- [ ] Test: load level → gameplay receives correct data

## Phase 8 — Gameplay
- [ ] Implement gameplay controllers and elements
- [ ] Implement `ObstacleRegistry` if game has multiple element types
- [ ] Connect gameplay events (OnLevelComplete, OnLevelFail) to UI + save

## Phase 9 — Polish & Production
- [ ] Add Object Pooling for high-frequency spawn/destroy
- [ ] Add `RemoteConfigService` when A/B testing is needed
- [ ] Add DebugService (dev build only)
- [ ] Audit naming convention + code standards
- [ ] Final test entire game flow
```

---

## 22. Things NOT to Do

| Don't | Do this instead |
|---|---|
| Using `.instance` on individual services | `ServiceLocator.Get<T>()` |
| Serialized fields on MonoBehaviour for config | Use ScriptableObject |
| Interface when only one implementation exists | Write class directly, interface when needed |
| Inline strings for event/tracking names | Use `const string` |
| Multiple utility files/classes | One file `Utils.cs`, one class `Utils` |
| Comments explaining what code does | Comments explaining why |
| Methods longer than 30 lines | Split method |
| Commented-out code in commits | Delete entirely |
| Modifying enum values in use by data | Only add new ones, never modify/delete |
| `async void` in regular methods | `async UniTask` — `async void` only in Unity callbacks |
| Using Coroutine | UniTask — Coroutine is banned in new projects |
| Calling `PlayerPrefs` directly from gameplay code | Only `UserDataService` may call `PlayerPrefs` |
| Combining all data into one class | Separate data classes per feature |
| Subscribing to events with lambda | Named method — lambdas cannot be unsubscribed |
| `Destroy(obj)` for frequently spawned objects | Object pooling — `pool.Return(obj)` |
| UI scripts containing business logic | UI only displays data and fires events |
| `SetActive(true/false)` for show/hide UI | `CanvasGroup` alpha + interactable |
| `catch (Exception) { }` swallowing errors | Log error with context or don't catch |
| Deleting/renaming fields in data classes | Only add new ones, use `[JsonProperty]` if rename needed |
| Creating script folders outside `Assets/Scripts/` | All code in `Assets/Scripts/` |
| `Task.Run()` or `System.Threading.Tasks` | Use UniTask — runs on main thread |
| `Update()` for waiting on conditions or sequences | `UniTask.WaitUntil()`, `UniTask.Delay()`, async method |
| Calling `SceneManager` directly from gameplay code | Only `SceneService` may call `SceneManager` |
| `LoadSceneMode.Single` | `LoadSceneMode.Additive` — Single will destroy GameContext |
| Writing runtime state to LevelDataSO | Level SO only contains input — runtime state in controller |
| Loading level data every time level changes | Use `AllLevelsSO` — load once, O(1) access |

---

## Quick Reference — Service Registration Order

```csharp
// Recommended order: services with no dependencies → register first
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

*This template is compiled from real-world experience. Update the version table at the top when new architectural decisions are made.*
