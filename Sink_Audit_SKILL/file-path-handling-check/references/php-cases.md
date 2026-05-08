# PHP Path Traversal Cases

## Purpose

This file contains PHP-specific path traversal patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- `file_get_contents`
- `fopen`
- `readfile`
- `unlink`
- `move_uploaded_file`
- include / require
- archive extraction
- PHP backends exposing download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. PHP Path Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- `file_get_contents`
- `fopen`
- `readfile`
- download handlers
- preview handlers
- log/config readers
- local file/resource loaders

## 1.2 File write, move, and delete paths
Look for:
- `file_put_contents`
- `unlink`
- `rename`
- `copy`
- `move_uploaded_file`
- export/save path logic
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- string path concatenation
- `realpath`
- custom path-cleaning helpers
- wrapper methods around storage/filesystem access
- include/resource path selection

## 1.4 Archive extraction and include/resource loading
Look for:
- zip extraction
- archive helpers
- `include`
- `require`
- template selection by path-like input
- local resource loading by user-controlled path

---

# 2. High-Coverage PHP Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, path construction behavior, file sink behavior, and missing containment controls.

## 2.1 Web, framework, and request entry candidates
Search for:
- `Route::get`
- `Route::post`
- `Route::put`
- `Route::patch`
- `Route::delete`
- `Route::any`
- `Route::match`
- `Route::resource`
- `Route::apiResource`
- `Controller`
- `__invoke`
- `Request $request`
- `$request->input`
- `$request->get`
- `$request->query`
- `$request->post`
- `$request->all`
- `$request->file`
- `$request->cookie`
- `$request->header`
- `$request->getContent`
- `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`
- `$_FILES`
- `php://input`
- `#[Route]`
- `@Route`
- `AbstractController`
- `Action`
- `Middleware`
- `Kernel`

## 2.2 Queue, command, webhook, import, export, and admin entries
Search for:
- `ShouldQueue`
- `handle`
- `Job`
- `Listener`
- `EventSubscriber`
- `Command`
- `Console`
- `schedule`
- `webhook`
- `callback`
- `import`
- `export`
- `upload`
- `download`
- `preview`
- `delete`
- `cleanup`
- `backup`
- `restore`
- `replay`
- `Storage::`
- `File::`
- `Cache::get`
- `Session::get`
- `$redis->get`

## 2.3 Path construction and transformation candidates
Search for:
- `$baseDir .`
- `$path .`
- `DIRECTORY_SEPARATOR`
- `realpath`
- `dirname`
- `basename`
- `pathinfo`
- `str_replace('../'`
- `str_replace('..'`
- `preg_replace`
- `ltrim`
- `rtrim`
- `trim`
- `parse_url`
- `urldecode`
- `rawurldecode`
- `base64_decode`
- `Storage::path`
- `storage_path`
- `base_path`
- `public_path`
- `resource_path`
- `app_path`
- `config_path`
- `UploadedFile::getClientOriginalName`
- `getClientOriginalName`
- `getClientOriginalExtension`
- `hashName`

## 2.4 File read, preview, download, include, and resource-load sink candidates
Search for:
- `file_get_contents`
- `fopen`
- `readfile`
- `fpassthru`
- `SplFileObject`
- `file`
- `stat`
- `filesize`
- `mime_content_type`
- `getimagesize`
- `exif_read_data`
- `response()->download`
- `response()->file`
- `BinaryFileResponse`
- `StreamedResponse`
- `Storage::get`
- `Storage::download`
- `Storage::response`
- `File::get`
- `include`
- `include_once`
- `require`
- `require_once`
- `view`
- `View::make`

## 2.5 File write, upload-save, copy, move, delete, and metadata sink candidates
Search for:
- `file_put_contents`
- `fwrite`
- `touch`
- `unlink`
- `rename`
- `copy`
- `mkdir`
- `rmdir`
- `chmod`
- `chown`
- `move_uploaded_file`
- `$file->move`
- `$file->storeAs`
- `$file->store`
- `Storage::put`
- `Storage::copy`
- `Storage::move`
- `Storage::delete`
- `Storage::makeDirectory`
- `Storage::deleteDirectory`
- `File::put`
- `File::copy`
- `File::move`
- `File::delete`
- `File::cleanDirectory`
- `File::deleteDirectory`

## 2.6 Archive extraction and compressed file candidates
Search for:
- `ZipArchive`
- `ZipArchive::extractTo`
- `getNameIndex`
- `statIndex`
- `PharData`
- `Phar`
- `RarArchive`
- `gzopen`
- `gzread`
- `tar`
- `extract`
- `unzip`
- `untar`
- `Archive`
- `Symfony\\Component\\Finder`

## 2.7 Required-control candidates
Search near sinks for:
- `realpath`
- `strpos($path, $baseDir) === 0`
- `str_starts_with`
- `basename`
- `pathinfo`
- `hashName`
- `Str::uuid`
- `Str::random`
- `Storage::disk`
- `allowedExtensions`
- `allowedMimes`
- `mimes:`
- `mimetypes:`
- `validate`
- `Validator`
- `Rule::in`
- `allowlist`
- `deny`
- `reject`
- `is_link`
- `readlink`
- `stream_resolve_include_path`
- `open_basedir`
- `LOCK_EX`

## 2.8 PHP graph search recipes
Useful combinations:

```text
Route::get + response()->download
$request->input + file_get_contents
$_GET + readfile
$_FILES + move_uploaded_file
getClientOriginalName + storeAs
Storage::path + File::get
Route::delete + Storage::delete
Command handle + unlink
ZipArchive::extractTo
include + request parameter
realpath + strpos prefix check
```

---

# 3. PHP Path Traversal Anti-Patterns

### A1. Direct read path from request
```php
readfile($baseDir . "/" . $_GET['file']);
```

Why risky:
User-controlled input may escape the intended base directory.

### A2. Unsafe delete path
```php
unlink($baseDir . "/" . $name);
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A3. Raw include path control
```php
include $templatePath;
```

Why risky:
User-controlled include/resource paths may expose unintended local files or cause unsafe local inclusion.

### A4. Weak `realpath` usage
```php
$path = realpath($baseDir . "/" . $userPath);
if (strpos($path, $baseDir) === 0) {
    echo file_get_contents($path);
}
```

Why risky:
Containment logic must be applied carefully and consistently across all operations.

### A5. Archive extraction without entry validation
```php
$zip->extractTo($destDir);
```

Why risky:
Archive entries may escape the intended extraction root.

---

# 4. Case Templates

## Case H-PATH-1: Arbitrary read via concatenated path

### Vulnerable pattern
```php
readfile($baseDir . "/" . $_GET['file']);
```

### Audit focus
Verify whether the input is attacker-controlled and whether resolved-path containment is enforced before use.

## Case H-PATH-2: Delete or overwrite path abuse

### Vulnerable pattern
```php
unlink($baseDir . "/" . $name);
```

### Audit focus
Verify canonical containment and whether destructive operations use the same safe path logic as reads.

## Case H-PATH-3: Local include/resource path control

### Vulnerable pattern
```php
include $templatePath;
```

### Audit focus
Verify whether raw path input can escape intended local resource scope.

## Case H-PATH-4: Unsafe archive extraction

### Vulnerable pattern
```php
$zip->extractTo($destDir);
```

### Audit focus
Verify extraction-root containment, entry validation, and overwrite behavior.

---

# 5. PHP-Specific Audit Heuristics

## 4.1 File API heuristics
Pay attention to:
- `file_get_contents`
- `fopen`
- `readfile`
- `file_put_contents`
- `unlink`
- `rename`
- `copy`
- wrapper helpers

## 4.2 Include/resource heuristics
Pay attention to:
- `include`
- `require`
- template path selection
- local resource loading
- raw PHP fallback templates

## 4.3 Upload and storage heuristics
Pay attention to:
- `move_uploaded_file`
- destination filename/path selection
- export path logic
- cleanup and overwrite logic

## 4.4 Archive heuristics
Pay attention to:
- zip extraction helpers
- archive entry trust
- destination assembly
- overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs admin/import path
- resource include vs file read helper
