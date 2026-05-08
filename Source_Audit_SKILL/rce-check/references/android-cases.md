# Android Command Execution Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for command execution and code execution source discovery.

Use it when Android code can introduce execution-relevant values, especially:
- exported activities, services, receivers, and content providers
- deep links and app links
- WebView JavaScript bridges
- Binder, AIDL, Messenger, PendingIntent, and IPC handlers
- WorkManager jobs and background workers
- SDK callbacks, push handlers, diagnostic tools, automation features, shell wrappers, dynamic script/plugin loaders, and backend execution API clients

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage Android Source Candidate Inventory

## 1.1 Manifest, component, and mobile entry candidates

- `android:exported="true"`
- `<activity>`
- `<service>`
- `<receiver>`
- `<provider>`
- `<intent-filter>`
- deep links
- app links
- `onCreate`
- `onNewIntent`
- `onStartCommand`
- `onReceive`
- `ContentProvider.query`
- `ContentProvider.call`
- `WorkManager`
- `Worker`
- `CoroutineWorker`
- `FirebaseMessagingService.onMessageReceived`
- `PendingIntent`
- `ActivityResultLauncher`
- `onActivityResult`
- `@JavascriptInterface`
- `addJavascriptInterface`
- Binder/AIDL methods
- SDK callback interfaces

## 1.2 Intent, Bundle, WebView, and IPC source candidates

- `getIntent()`
- `Intent.getData()`
- `Uri.getQueryParameter(...)`
- `getStringExtra(...)`
- `getStringArrayExtra(...)`
- `getBundleExtra(...)`
- `getSerializableExtra(...)`
- `getExtras()`
- `Bundle.getString(...)`
- `Bundle.getStringArray(...)`
- `Parcel.readString`
- `Parcel.createStringArray`
- WebView bridge method arguments
- JavaScript `postMessage`
- AIDL method args
- Messenger message `Bundle`
- WorkManager `Data.getString`
- push notification data

## 1.3 Command, tool, action, and interpreter selector candidates

- `cmd`
- `command`
- `commandName`
- `tool`
- `toolName`
- `script`
- `scriptPath`
- `action`
- `operation`
- `mode`
- `subcommand`
- `task`
- `jobType`
- `plugin`
- `module`
- `handler`
- `runner`
- `interpreter`
- `shell`
- `diagnostic`
- `debug`
- `automation`

## 1.4 Argument, option, file, environment, and cwd candidates

- `args`
- `argv`
- `arguments`
- `option`
- `flags`
- `target`
- `host`
- `ip`
- `domain`
- `url`
- `file`
- `filename`
- `path`
- `input`
- `output`
- `config`
- `payload`
- `stdin`
- `env`
- `environment`
- `cwd`
- `workDir`
- `workingDir`
- `timeout`

## 1.5 Android execution mapping candidates

- `Runtime.getRuntime().exec`
- `ProcessBuilder`
- `ProcessBuilder.command`
- `ProcessBuilder.environment`
- `ProcessBuilder.directory`
- `su`
- `sh -c`
- `toybox`
- `toolbox`
- `logcat`
- `am`
- `pm`
- `app_process`
- `DexClassLoader`
- `PathClassLoader`
- `loadClass`
- `Class.forName`
- `Method.invoke`
- `ScriptEngine`
- native/JNI execution wrappers
- backend API calls that submit command/script/task payloads

## 1.6 Stored and second-order candidates

- SharedPreferences command settings
- Room/SQLite task definitions
- WorkManager input data
- offline queue payloads
- downloaded config
- plugin manifests
- WebView local storage
- push payload storage
- SDK cached tasks
- automation profiles
- diagnostic command history

## 1.7 Android graph search recipes

```text
android:exported/deep link + getStringExtra cmd/tool/action + Runtime.exec/ProcessBuilder
@JavascriptInterface + command/args/script + ProcessBuilder/backend execution API
Binder/AIDL/Messenger + args/options/file/path + shell/native wrapper
WorkManager/FirebaseMessagingService + inputData/payload commandTemplate + ProcessBuilder/API call
SharedPreferences/Room/offline queue + stored command/task + worker execution
DexClassLoader/Class.forName/Method.invoke + class/module/method from Intent/bridge/config
```

---

# 2. Android Source Patterns and Blind Spots

## A-S1. Exported component execution values

Intent extras, deep-link parameters, and push payload fields are source points when they influence diagnostics, shell wrappers, backend execution APIs, automation actions, or script/plugin loading.

## A-S2. WebView bridge command source

Bridge arguments can carry tool names, command fragments, script content, file paths, or backend task payloads.

## A-S3. WorkManager and offline task source

WorkManager input data, offline queues, Room records, and cached SDK tasks are second-order execution sources when written by external or user-controlled paths.

## A-S4. Dynamic class/plugin source

Class names, module names, plugin paths, and method names become code-execution source candidates when they reach class loaders or reflection.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not externally reachable and no indirect trigger is visible,
- the value only affects display/logging,
- the command/tool/action is fixed and arguments are strictly allowlisted,
- dynamic loading is fixed to trusted local assets with no attacker-controlled selector.

Use `Suspected source` or `Not enough evidence` if:
- component reachability is unclear,
- WebView bridge origin restrictions are hidden,
- worker/offline queue writer paths are missing,
- native/JNI wrapper behavior is hidden.

---

# 4. Quick Android Source Checklist

- Are exported components receiving command, action, tool, script, args, file path, or debug parameters?
- Are WebView bridges or SDK callbacks submitting execution-relevant values?
- Are WorkManager/offline queue payloads later used by command wrappers or backend execution APIs?
- Are class names, plugin paths, method names, or script payloads influenced?
- Are environment, working directory, stdin, config, or file path values passed to process wrappers?
