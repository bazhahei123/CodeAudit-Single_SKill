# Python File Path Handling Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for path traversal and unsafe file path handling source discovery.

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

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream file operation in the target code.

---

# 1. Python Source Discovery Points

Prioritize these source values and events:
- route parameters, query values, form fields, JSON body fields, headers, and cookies
- uploaded original filenames and metadata
- stored file paths from database, cache, session, queue, or task payloads
- archive entry names from `zipfile`, `tarfile`, import bundles, and manifests
- template names, resource selectors, local file keys, and report names
- export destinations, upload destinations, cleanup targets, and delete targets
- file-serving helper arguments and background worker file arguments

Source questions:
- Which source supplies the filename, path, template name, or resource selector?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value decoded, normalized, joined, resolved, or mapped before use?
- Which operation context should be audited next: read, write, delete, move, extract, include, or resource load?
- Which follow-up control should inspect resolved containment, allowlists, symlinks, or storage abstraction behavior?

---

# 2. High-Coverage Python Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when the code shows file path construction, resource selection, archive extraction, upload/export destination, or file operation relevance.

## 2.1 Web, API, and request entry candidates

- Django URL `path(...)`
- Django `re_path(...)`
- Django views
- DRF `APIView`
- DRF `ViewSet`
- DRF `GenericViewSet`
- DRF `ModelViewSet`
- DRF `@api_view`
- DRF `@action`
- Flask `@app.route`
- Flask `@blueprint.route`
- FastAPI `@app.get`
- FastAPI `@app.post`
- FastAPI `@app.put`
- FastAPI `@app.delete`
- FastAPI `APIRouter`
- Starlette `Route`
- Tornado `RequestHandler`
- GraphQL resolver
- GraphQL mutation
- Celery task
- RQ job
- webhook handler
- admin import command

## 2.2 Request and upload source candidates

- `request.GET`
- `request.POST`
- `request.FILES`
- `request.data`
- `request.query_params`
- `request.args`
- `request.form`
- `request.json`
- `request.get_json()`
- `request.headers`
- `request.cookies`
- FastAPI `Path`
- FastAPI `Query`
- FastAPI `Body`
- `UploadFile.filename`
- `FileStorage.filename`
- Django `UploadedFile.name`
- Pydantic fields named `file`, `filename`, `path`, `template`, `resource`, `export_path`

## 2.3 Path-like field and selector candidates

- `file`
- `filename`
- `file_name`
- `path`
- `file_path`
- `dir`
- `directory`
- `folder`
- `resource`
- `template`
- `template_name`
- `theme`
- `locale`
- `language`
- `report`
- `config`
- `log`
- `storage_key`
- `object_key`
- `bucket`
- `prefix`
- `uri`
- `url`
- `download`
- `preview`
- `export`
- `destination`
- `target`
- `cleanup_target`

## 2.4 Path construction and transform candidates

- `os.path.join`
- `os.path.abspath`
- `os.path.realpath`
- `os.path.normpath`
- `os.path.basename`
- `os.path.dirname`
- `pathlib.Path`
- `Path.joinpath`
- `Path.resolve`
- `Path.absolute`
- `werkzeug.utils.secure_filename`
- `urllib.parse.unquote`
- `posixpath.normpath`
- `PurePath`
- `safe_join`
- `base_dir`
- `upload_dir`
- `download_dir`
- `temp_dir`
- `extract_dir`
- `storage_path`

## 2.5 Archive, stored, queue, and second-order candidates

- `zipfile.ZipFile`
- `ZipInfo.filename`
- `namelist()`
- `infolist()`
- `tarfile.open`
- `TarInfo.name`
- `extract`
- `extractall`
- import manifest paths
- database path fields
- Django model `FileField`
- Django `ImageField`
- cache values
- session values
- Celery task args
- queue payloads
- job payloads
- export paths
- cleanup targets
- object storage keys

## 2.6 Downstream mapping candidates

- `open`
- `Path.open`
- `send_file`
- `send_from_directory`
- Django `FileResponse`
- Starlette/FastAPI `FileResponse`
- `shutil.copy`
- `shutil.move`
- `os.remove`
- `os.unlink`
- `os.rename`
- `os.replace`
- `Path.unlink`
- `Path.rename`
- `tempfile.NamedTemporaryFile`
- `render_template`
- `get_template`
- `jinja2.FileSystemLoader`
- storage helper reads/writes

## 2.7 Python graph search recipes

```text
route/view/@app.get + request args/path/query file/path/name + os.path/pathlib/open
UploadFile/FileStorage/UploadedFile + filename/name + save/open/shutil.copy
zipfile/tarfile + member filename/name + extract/extractall/write
Celery/RQ/task + path/file/export_path + open/delete/move
template/theme/locale/resource + get_template/FileSystemLoader/render
database FileField/path field + stored path + send_file/FileResponse/cleanup
```

---

# 3. Python Source Patterns

## P-S1. Request-derived filename or path
Example idea:
- route/query/body value becomes `filename`, `path`, `template`, `resource`, or `name`.

Audit relevance:
Direct request-derived path values often feed `open`, `send_file`, import/export, cleanup, or template/resource paths.

Follow-up:
- trace the value into `os.path`, `pathlib`, `open`, `send_file`, storage helpers, or file-serving wrappers.

## P-S2. Uploaded filename source
Example idea:
- uploaded original filename or extension influences save destination.

Audit relevance:
Upload metadata is client-controlled and can become write or second-order read source material.

Follow-up:
- verify server-generated filenames or strict filename allowlists.

## P-S3. Archive member name source
Example idea:
- zip/tar member name or import manifest path influences extraction destination.

Audit relevance:
Archive member names are path sources for extraction and overwrite behavior.

Follow-up:
- verify per-entry extraction-root containment and safe tar/zip handling.

## P-S4. Stored path or task payload source
Example idea:
- database path, cached path, session value, Celery/task argument, or queue payload is later used by file helpers.

Audit relevance:
Stored values may be second-order path sources when earlier attacker influence exists.

Follow-up:
- identify writer path and revalidation before file operations.

## P-S5. File-serving or resource selector source
Example idea:
- request value selects a file-serving path, template, local resource, report, or generated file.

Audit relevance:
Selectors can become local file paths if not mapped through strict allowlists.

Follow-up:
- verify safe file-serving roots and resource-key mapping.

---

# 4. Case Templates

## Case P-S-PATH-1: Download/read path source

Source focus:
Identify route/query/body values or stored paths that reach `open`, `send_file`, or download helpers.

Recommended follow-up:
Verify resolved containment and safe read operation context.

## Case P-S-PATH-2: Upload destination source

Source focus:
Identify uploaded original filename, extension, storage key, and destination path construction.

Recommended follow-up:
Verify server-generated names, allowlists, and overwrite behavior.

## Case P-S-PATH-3: Archive extraction source

Source focus:
Identify zip/tar member names, import manifest paths, and extraction destination assembly.

Recommended follow-up:
Verify extraction-root containment for every member.

## Case P-S-PATH-4: Cleanup/delete source

Source focus:
Identify delete, move, rename, or cleanup target source from request, storage, or queue/task payload.

Recommended follow-up:
Verify destructive operations use the same strong containment rules as reads.

---

# 5. Python-Specific Audit Heuristics

## 4.1 Request and upload source heuristics
Pay attention to:
- Django `request.GET`, `request.POST`, `request.FILES`
- Flask/FastAPI request args, form, JSON, and uploads
- route parameters
- uploaded original filenames
- extension or MIME-derived path values

## 4.2 `os.path` / `pathlib` source heuristics
Pay attention to:
- `os.path.join`
- `abspath`
- `realpath`
- `pathlib.Path`
- `resolve()`
- source values before and after path normalization
- simple prefix checks

## 4.3 File-serving and resource source heuristics
Pay attention to:
- `open`
- `send_file`
- `FileResponse`
- Django/Flask/FastAPI download helpers
- preview paths
- import/export helpers
- local template/resource readers

## 4.4 Archive and worker source heuristics
Pay attention to:
- `zipfile`
- `tarfile`
- `extractall`
- member names
- destination assembly
- Celery/task payload paths
- background cleanup paths

## 4.5 Layer inconsistency heuristics
Check whether path source handling is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs background/import path
- file-serving route vs export/import helper

---

# 6. False-Positive Controls

Do not mark a Python source as high-priority if:
- the value is selected from a strict allowlist of safe resource keys,
- the uploaded filename is replaced by a server-generated name before use,
- the value never reaches path construction or a file/resource operation,
- the stored path is trusted-only and cannot be attacker-influenced,
- the value is only used for display, logging, or labels.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- storage abstraction path resolution is not visible,
- archive member handling is abstracted,
- the source is visible but operation context is unclear,
- resolved containment may exist elsewhere.

---

# 7. What Good Evidence Looks Like

Good Python source evidence includes:
- route/view/worker/import entry point,
- source API such as request input, upload filename, archive member, cache read, database value, or queue/task argument,
- propagation such as decode, join, normalize, resolve, storage, or mapping,
- `open`, `os.path`, `pathlib`, `send_file`, archive helper, storage helper, or cleanup helper receiving the value,
- operation context when visible.

Good source evidence answers:
1. Which Python entry point receives the path-like value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which path construction or file/resource operation should be audited next?
4. Is the source used for read, write, delete, move, extract, include/resource load, or file serving?

---

# 8. Quick Python Source Checklist

- Are request values used as filenames, resource keys, templates, or path fragments?
- Are uploaded filenames or extensions used for save paths?
- Are archive member names used for extraction destinations?
- Are stored paths read from database, cache, session, queue, or task payloads?
- Are file-serving helper paths mapped safely or used as raw paths?
- Are delete, move, copy, export, cleanup, and read paths sourced differently?
