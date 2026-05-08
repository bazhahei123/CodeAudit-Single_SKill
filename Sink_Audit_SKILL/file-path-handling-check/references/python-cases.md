# Python Path Traversal Cases

## Purpose

This file contains Python-specific path traversal patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- `open`
- `os.path`
- `pathlib`
- `zipfile` / `tarfile`
- file-serving helpers
- Python backends exposing download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. Python Path Control Points

When auditing Python applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- `open(...)`
- `send_file`
- file preview helpers
- log/config readers
- import readers
- local template/resource loading

## 1.2 File write, move, and delete paths
Look for:
- upload destination logic
- export/save path logic
- `os.remove`
- `os.rename`
- `shutil.move`
- `open(..., 'w')`
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- `os.path.join`
- `os.path.abspath`
- `os.path.realpath`
- `pathlib.Path`
- `resolve()`
- string path concatenation
- wrapper helpers

## 1.4 Archive extraction and resource loading
Look for:
- `zipfile`
- `tarfile`
- extraction destination assembly
- local file/resource loaders
- template selection by path-like input

---

# 2. High-Coverage Python Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, path construction behavior, file sink behavior, and missing containment controls.

## 2.1 Web, API, and request entry candidates
Search for:
- `@app.route`
- `@blueprint.route`
- `@bp.route`
- `Flask`
- `request.args`
- `request.form`
- `request.values`
- `request.files`
- `request.cookies`
- `request.headers`
- `request.get_json`
- `request.data`
- `@api_view`
- `APIView`
- `ViewSet`
- `ModelViewSet`
- `GenericAPIView`
- `FileUploadParser`
- `MultiPartParser`
- `path(`
- `re_path(`
- `View`
- `dispatch`
- `get`
- `post`
- `put`
- `patch`
- `delete`
- `@csrf_exempt`
- `@app.get`
- `@app.post`
- `FastAPI`
- `APIRouter`
- `UploadFile`
- `File`
- `Form`
- `Query`
- `Path`
- `Header`
- `Cookie`
- `Request`
- `GraphQLView`
- `Mutation`

## 2.2 Worker, message, import, export, and admin entries
Search for:
- `@shared_task`
- `@app.task`
- `Celery`
- `dramatiq.actor`
- `rq.job`
- `huey.task`
- `BaseCommand`
- `handle`
- `KafkaConsumer`
- `pika`
- `basic_consume`
- `on_message`
- `consume`
- `websocket`
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
- `cache.get`
- `session`

## 2.3 Path construction and transformation candidates
Search for:
- `os.path.join`
- `os.path.abspath`
- `os.path.realpath`
- `os.path.normpath`
- `os.path.dirname`
- `os.path.basename`
- `os.path.expanduser`
- `os.path.expandvars`
- `pathlib.Path`
- `Path(`
- `.joinpath`
- `.resolve`
- `.absolute`
- `.relative_to`
- `.parents`
- `PurePath`
- `secure_filename`
- `safe_join`
- `urlparse`
- `unquote`
- `urllib.parse.unquote`
- `base64.b64decode`
- `replace("../", "")`
- `replace("..", "")`
- `split`
- `f"{`
- `.format(`
- `+ "/" +`
- `filename`
- `file.filename`

## 2.4 File read, preview, download, and resource-load sink candidates
Search for:
- `open`
- `Path.open`
- `Path.read_text`
- `Path.read_bytes`
- `os.open`
- `io.open`
- `codecs.open`
- `send_file`
- `send_from_directory`
- `FileResponse`
- `StreamingResponse`
- `StaticFiles`
- `django.http.FileResponse`
- `django.views.static.serve`
- `default_storage.open`
- `storage.open`
- `Image.open`
- `yaml.safe_load(open`
- `json.load(open`
- `configparser.read`
- `jinja2.FileSystemLoader`
- `get_template`
- `render_template`
- `pkgutil.get_data`
- `importlib.resources`

## 2.5 File write, upload-save, copy, move, delete, and metadata sink candidates
Search for:
- `open(..., "w"`
- `open(..., "a"`
- `open(..., "x"`
- `Path.write_text`
- `Path.write_bytes`
- `os.remove`
- `os.unlink`
- `Path.unlink`
- `shutil.rmtree`
- `os.rmdir`
- `Path.rmdir`
- `os.rename`
- `os.replace`
- `shutil.move`
- `shutil.copy`
- `shutil.copyfile`
- `shutil.copy2`
- `shutil.copytree`
- `os.makedirs`
- `Path.mkdir`
- `file.save`
- `default_storage.save`
- `storage.save`
- `tempfile.NamedTemporaryFile`
- `os.chmod`
- `os.stat`
- `Path.exists`
- `Path.is_file`

## 2.6 Archive extraction and compressed file candidates
Search for:
- `zipfile.ZipFile`
- `ZipFile.extract`
- `ZipFile.extractall`
- `ZipInfo.filename`
- `namelist`
- `infolist`
- `tarfile.open`
- `TarFile.extract`
- `TarFile.extractall`
- `TarInfo.name`
- `shutil.unpack_archive`
- `patoolib.extract_archive`
- `gzip.open`
- `bz2.open`
- `lzma.open`
- `extract`
- `unzip`
- `untar`

## 2.7 Required-control candidates
Search near sinks for:
- `Path.resolve`
- `os.path.realpath`
- `os.path.commonpath`
- `relative_to`
- `safe_join`
- `werkzeug.utils.secure_filename`
- `django.utils._os.safe_join`
- `SuspiciousFileOperation`
- `base_dir`
- `root_dir`
- `allowed_extensions`
- `allowed_mime`
- `mimetypes`
- `PurePath(name).name`
- `os.path.basename`
- `follow_symlinks=False`
- `is_symlink`
- `samefile`
- `validate`
- `allowlist`
- `reject`
- `uuid.uuid4`

## 2.8 Python graph search recipes
Useful combinations:

```text
@app.route + send_file
request.args + os.path.join + open
UploadFile + Path.write_bytes
file.filename + save
APIView + FileResponse
@shared_task + os.remove
cache.get + open
ZipFile.extractall
TarFile.extractall
ZipInfo.filename + open/write
Path.resolve without commonpath/relative_to
os.path.abspath + startswith
```

---

# 3. Python Path Traversal Anti-Patterns

### A1. Unsafe joined read path
```python
path = os.path.join(base_dir, user_path)
with open(path, "rb") as f:
    return f.read()
```

Why risky:
User-controlled `user_path` may escape the intended base directory.

### A2. Weak absolute-path check
```python
path = os.path.abspath(os.path.join(base_dir, user_path))
if path.startswith(base_dir):
    return open(path).read()
```

Why risky:
Containment checks need careful real-path handling and consistent use.

### A3. Unsafe delete path
```python
os.remove(os.path.join(base_dir, user_path))
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Tar/zip extraction without entry validation
```python
tar.extractall(dest_dir)
```

Why risky:
Archive entry names may escape the extraction root.

### A5. Local template/resource path control
```python
return send_file(user_path)
```

Why risky:
Raw path input may expose unintended files or resources.

---

# 4. Case Templates

## Case P-PATH-1: Arbitrary read via joined path

### Vulnerable pattern
```python
path = os.path.join(base_dir, user_path)
return open(path).read()
```

### Audit focus
Verify whether `user_path` is attacker-controlled and whether resolved-path containment is enforced before use.

## Case P-PATH-2: Unsafe archive extraction

### Vulnerable pattern
```python
tar.extractall(dest_dir)
```

### Audit focus
Verify entry validation, extraction-root containment, and overwrite behavior.

## Case P-PATH-3: Delete or overwrite path abuse

### Vulnerable pattern
```python
os.remove(os.path.join(base_dir, user_path))
```

### Audit focus
Verify whether delete/write operations use the same strong containment logic as read operations.

## Case P-PATH-4: File-serving helper misuse

### Vulnerable pattern
```python
return send_file(user_path)
```

### Audit focus
Verify whether raw user input can control file-serving paths.

---

# 5. Python-Specific Audit Heuristics

## 4.1 `os.path` heuristics
Pay attention to:
- `join`
- `abspath`
- `realpath`
- simple prefix checks
- wrapper helpers around path validation

## 4.2 `pathlib` heuristics
Pay attention to:
- `Path(...)`
- `resolve()`
- parent containment checks
- differences between resolved and unresolved paths

## 4.3 File-serving heuristics
Pay attention to:
- `send_file`
- Django/Flask/FastAPI download helpers
- preview paths
- import/export helpers
- local resource readers

## 4.4 Archive heuristics
Pay attention to:
- `zipfile`
- `tarfile`
- `extractall`
- manual entry extraction
- destination assembly and overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs background/import path
