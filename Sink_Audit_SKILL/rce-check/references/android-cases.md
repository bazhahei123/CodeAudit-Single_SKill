# Android Command Execution Cases

## Purpose

This file contains Android-specific command execution, shell invocation, interpreter abuse, dynamic code loading, native library loading, WebView bridge, and IPC-triggered execution cases.

Use it when the target application includes Android code, especially in:
- exported activities, services, broadcast receivers, and content providers
- deep links, app links, share targets, and push notification handlers
- WebView JavaScript bridges and URL interception
- Binder, AIDL, Messenger, and Intent-based IPC
- SDK callbacks, background work, import/export, diagnostics, and device utility features
- native library loading, plugin loading, Dex loading, and root/shell helper wrappers

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify exported reachability, attacker-controlled input, execution context, sink behavior, and missing controls.

---

# 1. Android Execution Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- intent filters and deep links
- WebView bridge methods
- Binder or AIDL calls
- ContentProvider methods
- push notification callbacks
- WorkManager and background services
- diagnostic and device utility flows

## 1.2 Execution points
Look for:
- process and shell APIs
- root command wrappers
- interpreter invocations
- WebView JavaScript execution
- Dex/class loading
- native library loading
- external tool and binary wrappers

## 1.3 Trust-boundary controls
Look for:
- `android:exported`
- component permissions
- caller identity checks
- package/signature validation
- fixed command allowlists
- no shell invocation
- trusted-only dynamic loading

---

# 2. High-Coverage Android Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm reachability, input control, execution context, and missing controls.

## 2.1 Manifest and component entry candidates
Search for:
- `android:exported="true"`
- `android:exported`
- `<activity`
- `<service`
- `<receiver`
- `<provider`
- `<intent-filter>`
- `<action`
- `<category`
- `<data`
- `android:scheme`
- `android:host`
- `android:path`
- `android:mimeType`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:authorities`

## 2.2 Activity, service, receiver, provider, job, and callback entries
Search for:
- `onCreate`
- `onNewIntent`
- `getIntent`
- `getAction`
- `getData`
- `getExtras`
- `onStartCommand`
- `onBind`
- `onHandleIntent`
- `Worker.doWork`
- `BroadcastReceiver`
- `onReceive`
- `ContentProvider`
- `query`
- `insert`
- `update`
- `delete`
- `openFile`
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `ACTION_VIEW`
- `ACTION_SEND`
- `Intent.EXTRA_TEXT`
- `Intent.EXTRA_STREAM`

## 2.3 WebView, bridge, and IPC entries
Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `evaluateJavascript`
- `loadUrl`
- `javascript:`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `onTransact`
- `Binder`
- `IBinder`
- `IInterface`
- `AIDL`
- `Messenger`
- `Handler`
- `Message`
- `ResultReceiver`

## 2.4 Process, shell, and command sink candidates
Search for:
- `Runtime.getRuntime().exec`
- `Runtime.exec`
- `ProcessBuilder`
- `ProcessBuilder.start`
- `Process`
- `/system/bin/sh`
- `sh -c`
- `/system/bin/su`
- `su`
- `toybox`
- `toolbox`
- `cmd`
- `am`
- `pm`
- `settings`
- `logcat`
- `getprop`
- `setprop`
- `run-as`
- `runCommand`
- `execCommand`
- `Shell`
- `ShellUtils`
- `RootTools`
- `libsuperuser`

## 2.5 Dynamic code, WebView script, and native loading candidates
Search for:
- `DexClassLoader`
- `PathClassLoader`
- `InMemoryDexClassLoader`
- `BaseDexClassLoader`
- `loadClass`
- `Class.forName`
- `Method.invoke`
- `newInstance`
- `System.load`
- `System.loadLibrary`
- `Runtime.load`
- `Runtime.loadLibrary`
- `dlopen`
- `JNI`
- `evaluateJavascript`
- `loadUrl("javascript:`
- `addJavascriptInterface`
- `ScriptEngineManager`
- `KotlinScript`

## 2.6 Command construction candidates
Search for:
- `String command`
- `String cmd`
- `String[] args`
- `List<String>`
- `StringBuilder`
- `String.format`
- `joinToString`
- `split(" ")`
- `concat`
- `+ " " +`
- `Uri.decode`
- `URLDecoder.decode`
- `Base64.decode`
- `getStringExtra`
- `getQueryParameter`
- `toolName`
- `scriptPath`
- `workingDirectory`
- `environment`

## 2.7 Required-control candidates
Search near sinks for:
- `android:exported="false"`
- `android:permission`
- `checkCallingPermission`
- `checkCallingOrSelfPermission`
- `getCallingPackage`
- `getCallingUid`
- `PackageManager`
- `signature`
- `setPackage`
- `setComponent`
- fixed command arrays
- command allowlist
- option allowlist
- no `sh -c`
- `timeout`
- `destroy`
- trusted Dex path
- signature verification
- `WebView.setJavaScriptEnabled(false)`
- origin allowlist
- `Uri.getHost`
- `Uri.getScheme`

## 2.8 Android graph search recipes
Useful combinations:

```text
android:exported="true" + Runtime.exec
ACTION_VIEW + getQueryParameter + ProcessBuilder
onReceive + sh -c
onStartCommand + su
@JavascriptInterface + Runtime.exec
evaluateJavascript + request/deep-link value
DexClassLoader + downloaded path
System.load + external path
ContentProvider.call + runCommand
getStringExtra + shell command
```

---

# 3. Android Command Execution Anti-Patterns

### A1. Exported component executes shell command from Intent
```java
String cmd = getIntent().getStringExtra("cmd");
Runtime.getRuntime().exec(cmd);
```

Why risky:
Another app or deep link may control command execution if the component is exported or indirectly reachable.

### A2. WebView bridge reaches command execution
```java
@JavascriptInterface
public void run(String arg) {
    new ProcessBuilder("sh", "-c", arg).start();
}
```

Why risky:
JavaScript bridge methods become an execution boundary when exposed to untrusted content.

### A3. Dynamic Dex loading from attacker-influenced path
```java
DexClassLoader loader = new DexClassLoader(path, out, null, getClassLoader());
```

Why risky:
Attacker-controlled code paths can lead to arbitrary code loading.

### A4. Native library loading from external path
```java
System.load(userPath);
```

Why risky:
Loading attacker-controlled native libraries is direct code execution.

---

# 4. Case Templates

## Case A-CMD-1: Exported Activity command execution

### Vulnerable pattern
```java
Runtime.getRuntime().exec(getIntent().getStringExtra("cmd"));
```

### Audit focus
Verify exported reachability, caller controls, command context, and allowlist enforcement.

## Case A-CMD-2: JavaScript bridge to shell

### Vulnerable pattern
```java
new ProcessBuilder("sh", "-c", script).start();
```

### Audit focus
Verify WebView origin control, JavaScript bridge exposure, and shell context.

## Case A-CMD-3: Dynamic code or native library loading

### Vulnerable pattern
```java
new DexClassLoader(remotePath, optimizedDir, null, parent);
```

### Audit focus
Verify path trust, signature validation, storage location, and caller influence.

---

# 5. Android-Specific Audit Heuristics

## 5.1 Exported reachability heuristics
Check whether the entry is reachable from another app, browser deep link, file share, push payload, or IPC caller.

## 5.2 Shell helper heuristics
Treat root wrappers, shell utility classes, and diagnostic features as first-class execution sinks.

## 5.3 WebView heuristics
Review JavaScript bridge methods, origin restrictions, and `evaluateJavascript` or `javascript:` URLs.

## 5.4 Dynamic loading heuristics
Review Dex, class, plugin, and native library loading when paths or bytes are attacker-influenced.
