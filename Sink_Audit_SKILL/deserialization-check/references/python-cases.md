# Python Unsafe Deserialization Cases

## Purpose

This file contains Python-specific unsafe deserialization patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- pickle-based workflows
- PyYAML loaders
- cache/session restore helpers
- queue consumers
- Python backends exposing import, background-job, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, trust boundary, and dangerous post-deserialization behavior in the target code.

---

# 1. Python Deserialization Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Entry and restore points
Look for:
- `pickle.loads`
- `pickle.load`
- `yaml.load`
- unsafe YAML loaders
- custom decoder helpers
- cache/session restore code
- queue consumers
- import handlers

## 1.2 Object behavior controls
Look for:
- `__reduce__`
- `__setstate__`
- custom reconstruction hooks
- object factories
- downstream code trusting restored object state

## 1.3 Framework and library controls
Look for:
- signed session or cookie blobs
- Celery / queue payload restoration
- cache deserialization
- YAML loader selection
- wrapper helpers that hide underlying restore logic

## 1.4 Trust-boundary controls
Look for:
- unsigned or weakly trusted cookies
- import payload trust assumptions
- internal queue or cache trust assumptions
- stored blob replay or later restore paths

---

# 2. High-Coverage Python Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, sink behavior, trigger behavior, and missing controls.

## 2.1 Web, API, and request entry candidates
Search for:
- `@app.route`
- `@blueprint.route`
- `@bp.route`
- `Flask`
- `request.data`
- `request.get_data`
- `request.form`
- `request.files`
- `request.cookies`
- `request.headers`
- `request.args`
- `@api_view`
- `APIView`
- `ViewSet`
- `ModelViewSet`
- `GenericAPIView`
- `parser_classes`
- `FileUploadParser`
- `MultiPartParser`
- `path(`
- `re_path(`
- `url(`
- `View`
- `dispatch`
- `post`
- `get`
- `put`
- `patch`
- `delete`
- `@csrf_exempt`
- `@app.post`
- `@app.get`
- `Body`
- `File`
- `UploadFile`
- `Cookie`
- `Header`
- `Depends`
- `Request`
- `Form`
- `aiohttp.web.post`
- `tornado.web.RequestHandler`
- `GraphQLView`
- `Mutation`

## 2.2 Message, worker, job, and import entry candidates
Search for:
- `@shared_task`
- `@app.task`
- `Celery`
- `task_serializer`
- `accept_content`
- `kombu`
- `enable_insecure_serializers`
- `dramatiq.actor`
- `rq.job`
- `huey.task`
- `on_message`
- `consume`
- `KafkaConsumer`
- `pika`
- `basic_consume`
- `redis.pubsub`
- `websocket`
- `import`
- `upload`
- `restore`
- `replay`
- `sync`
- `management command`
- `BaseCommand`
- `handle`
- `cache.get`
- `session`
- `signed_cookie`

## 2.3 Native object-restoration sink candidates
Search for:
- `pickle.load`
- `pickle.loads`
- `_pickle.loads`
- `cPickle.loads`
- `dill.load`
- `dill.loads`
- `cloudpickle.load`
- `cloudpickle.loads`
- `joblib.load`
- `pandas.read_pickle`
- `torch.load`
- `numpy.load`
- `allow_pickle=True`
- `shelve.open`
- `marshal.load`
- `marshal.loads`
- `copyreg`
- `pickle.Unpickler`
- `Unpickler.find_class`
- `loads(data)`
- `deserialize(data)`
- `restore(data)`
- `decode(data)`

## 2.4 YAML, JSON object hook, and framework serializer candidates
Search for:
- `yaml.load`
- `yaml.Loader`
- `yaml.FullLoader`
- `yaml.UnsafeLoader`
- `yaml.unsafe_load`
- `ruamel.yaml.YAML`
- `typ='unsafe'`
- `jsonpickle.decode`
- `jsonpickle.loads`
- `json.loads`
- `object_hook`
- `object_pairs_hook`
- `msgpack.unpackb`
- `ext_hook`
- `raw=False`
- `bson.decode`
- `xmlrpc.client.loads`
- `Serializer.loads`
- `django.core.signing.loads`
- `PickleSerializer`
- `SESSION_SERIALIZER`
- `cache serializer`
- `redis serializer`
- `kombu.serialization.register`

## 2.5 Trigger and gadget behavior candidates
Search for:
- `__reduce__`
- `__reduce_ex__`
- `__setstate__`
- `__getstate__`
- `__getnewargs__`
- `__getnewargs_ex__`
- `__new__`
- `__del__`
- `find_class`
- `persistent_load`
- `object_hook`
- `ext_hook`
- `importlib.import_module`
- `getattr`
- `setattr`
- `eval`
- `exec`
- `os.system`
- `subprocess`
- `open`
- `requests.get`
- `socket`
- `Path`
- `tempfile`

## 2.6 Required-control candidates
Search near sinks for:
- `safe_load`
- `CSafeLoader`
- `SafeLoader`
- `FullLoader` review
- `allow_pickle=False`
- `weights_only=True`
- `RestrictedUnpickler`
- `find_class` allowlist
- `allowed`
- `allowlist`
- `signature`
- `Signer`
- `TimestampSigner`
- `BadSignature`
- `loads(..., salt=`
- `max_age`
- `schema`
- `pydantic`
- `dataclass`
- `TypedDict`
- `validate`
- `content_type`
- `accept_content`
- `task_serializer='json'`

## 2.7 Python graph search recipes
Useful combinations:

```text
@app.route + pickle.loads
request.data + pickle.loads
APIView + yaml.load
UploadFile + joblib.load
cache.get + pickle.loads
@shared_task + pickle.loads
accept_content + pickle
enable_insecure_serializers
torch.load + request/file/import
numpy.load + allow_pickle=True
jsonpickle.decode + request
```

---

# 3. Python Unsafe Deserialization Anti-Patterns

### A1. `pickle.loads` on request-derived data
```python
obj = pickle.loads(request.data)
```

Why risky:
Untrusted input reaches native Python object restoration directly.

### A2. Unsafe YAML loader
```python
data = yaml.load(payload, Loader=yaml.Loader)
```

Why risky:
Unsafe loaders may construct arbitrary Python objects.

### A3. Stored blob later unpickled
```python
blob = cache.get(key)
obj = pickle.loads(blob)
```

Why risky:
The attacker may influence stored data indirectly, creating a second-order path.

### A4. Implicit trust in queue message restore
```python
obj = pickle.loads(message.body)
```

Why risky:
Internal message transport does not automatically imply trusted origin.

### A5. Signed-looking but weakly trusted restore path
```python
payload = request.cookies.get("session_data")
obj = deserialize_session(payload)
```

Why risky:
The code may assume integrity without verifying it strongly.

---

# 4. Case Templates

## Case P-DESER-1: Direct pickle restore

### Vulnerable pattern
```python
obj = pickle.loads(request.data)
```

### Audit focus
Verify whether the input is attacker-controlled and whether dangerous object reconstruction is possible.

## Case P-DESER-2: Unsafe YAML load

### Vulnerable pattern
```python
data = yaml.load(payload, Loader=yaml.Loader)
```

### Audit focus
Verify whether the loader permits unsafe object construction and whether `payload` is attacker-controlled.

## Case P-DESER-3: Second-order cache or stored blob restore

### Vulnerable pattern
```python
blob = cache.get(key)
obj = pickle.loads(blob)
```

### Audit focus
Verify who can influence `blob` and whether integrity or trusted-origin controls exist.

## Case P-DESER-4: Queue message restore

### Vulnerable pattern
```python
obj = pickle.loads(message.body)
```

### Audit focus
Verify whether upstream producers can be attacker-influenced and whether safer formats or strict validation exist.

---

# 5. Python-Specific Audit Heuristics

## 4.1 Pickle heuristics
Pay attention to:
- `pickle.loads`
- `pickle.load`
- helper wrappers around pickle
- cache/session restore helpers
- hidden pickle usage in utilities

## 4.2 YAML heuristics
Pay attention to:
- `yaml.load`
- loader class selection
- safe vs unsafe loader modes
- helper wrappers that hide actual loader choice

## 4.3 Object behavior heuristics
Pay attention to:
- `__reduce__`
- `__setstate__`
- custom reconstruction side effects
- classes that can trigger external behavior after restore

## 4.4 Queue / cache heuristics
Pay attention to:
- message consumers
- Celery or background tasks
- cache reload helpers
- trust assumptions about "internal" data

## 4.5 Layer inconsistency heuristics
Check whether deserialization safety is consistent across:
- request path vs worker path
- import path vs session/cache restore
- public API vs internal consumer
- safe loader path vs legacy helper path
