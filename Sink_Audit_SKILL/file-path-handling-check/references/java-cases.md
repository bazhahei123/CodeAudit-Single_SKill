# Java Path Traversal Cases

## Purpose

This file contains Java-specific path traversal patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `java.io.File`
- `java.nio.file.Path` / `Paths`
- `Files.*`
- `ZipInputStream`
- resource or template loaders
- Java backends exposing file download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. Java Path Control Points

When auditing Java applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- download handlers
- preview handlers
- `Files.readAllBytes`
- `FileInputStream`
- `Resource` loaders
- log or config readers

## 1.2 File write, move, and delete paths
Look for:
- upload destination logic
- export destination logic
- `Files.copy`
- `Files.move`
- `Files.delete`
- `FileOutputStream`
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- `Paths.get(...)`
- `resolve(...)`
- `normalize()`
- `toRealPath()`
- string path concatenation
- wrapper helpers around file access

## 1.4 Archive extraction and resource loading
Look for:
- `ZipInputStream`
- archive entry name handling
- template/resource loading
- local include-like resource resolution

---

# 2. High-Coverage Java Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, path construction behavior, file sink behavior, and missing containment controls.

## 2.1 HTTP, controller, and request entry candidates
Search for:
- `@RestController`
- `@Controller`
- `@RequestMapping`
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@PatchMapping`
- `@DeleteMapping`
- `@RequestBody`
- `@RequestParam`
- `@PathVariable`
- `@RequestHeader`
- `@CookieValue`
- `@ModelAttribute`
- `MultipartFile`
- `Part`
- `HttpServletRequest`
- `ServletRequest`
- `ServletInputStream`
- `HttpServletResponse`
- `doGet`
- `doPost`
- `doPut`
- `doDelete`
- `doFilter`
- `Filter`
- `HandlerInterceptor`
- `OncePerRequestFilter`
- `WebMvcConfigurer`
- `ResourceHttpRequestHandler`
- `ResourceHandlerRegistry`
- `addResourceHandlers`

## 2.2 JAX-RS, GraphQL, RPC, message, and job entries
Search for:
- `@Path`
- `@GET`
- `@POST`
- `@PUT`
- `@DELETE`
- `@Consumes`
- `@Produces`
- `@QueryParam`
- `@FormParam`
- `@HeaderParam`
- `@CookieParam`
- `@GraphQlController`
- `@QueryMapping`
- `@MutationMapping`
- `@SchemaMapping`
- `@MessageMapping`
- `@ServerEndpoint`
- `@OnMessage`
- `@GrpcService`
- `BindableService`
- `StreamObserver`
- `@KafkaListener`
- `@RabbitListener`
- `@JmsListener`
- `MessageListener`
- `onMessage`
- `@Scheduled`
- `QuartzJobBean`
- `Tasklet`
- `ItemReader`
- `ItemProcessor`
- `import`
- `export`
- `upload`
- `download`
- `preview`
- `delete`
- `cleanup`
- `replay`

## 2.3 Path construction and transformation candidates
Search for:
- `Paths.get`
- `Path.of`
- `new File`
- `File.separator`
- `resolve`
- `resolveSibling`
- `relativize`
- `normalize`
- `toAbsolutePath`
- `toRealPath`
- `getCanonicalPath`
- `getCanonicalFile`
- `getAbsolutePath`
- `getOriginalFilename`
- `StringUtils.cleanPath`
- `FilenameUtils.normalize`
- `FilenameUtils.getName`
- `URLDecoder.decode`
- `UriUtils.decode`
- `Base64.getDecoder`
- `String.format`
- `StringBuilder`
- `concat`
- `replace("../", "")`
- `replace("..", "")`
- `substring`
- `split`
- `ServletContext.getRealPath`
- `ResourceUtils.getFile`

## 2.4 File read, preview, download, and resource-load sink candidates
Search for:
- `Files.readAllBytes`
- `Files.readString`
- `Files.lines`
- `Files.newInputStream`
- `Files.probeContentType`
- `FileInputStream`
- `RandomAccessFile`
- `BufferedReader`
- `InputStreamReader`
- `FileReader`
- `Scanner`
- `IOUtils.toByteArray`
- `FileUtils.readFileToString`
- `FileUtils.readFileToByteArray`
- `FileCopyUtils.copyToByteArray`
- `ResponseEntity<Resource>`
- `InputStreamResource`
- `FileSystemResource`
- `UrlResource`
- `PathResource`
- `ByteArrayResource`
- `resourceLoader.getResource`
- `ClassPathResource`
- `getResourceAsStream`
- `ImageIO.read`
- `Properties.load`
- `Configuration.getTemplate`
- `TemplateEngine`

## 2.5 File write, upload-save, copy, move, delete, and metadata sink candidates
Search for:
- `Files.write`
- `Files.writeString`
- `Files.newOutputStream`
- `Files.createFile`
- `Files.createDirectory`
- `Files.createDirectories`
- `Files.copy`
- `Files.move`
- `Files.delete`
- `Files.deleteIfExists`
- `Files.exists`
- `Files.isRegularFile`
- `Files.size`
- `Files.setPosixFilePermissions`
- `FileOutputStream`
- `FileWriter`
- `PrintWriter`
- `RandomAccessFile`
- `File.delete`
- `File.renameTo`
- `File.mkdirs`
- `File.createNewFile`
- `MultipartFile.transferTo`
- `Part.write`
- `FileUtils.write`
- `FileUtils.copyFile`
- `FileUtils.moveFile`
- `FileUtils.deleteDirectory`
- `FileUtils.forceDelete`
- `DiskFileItem.write`

## 2.6 Archive extraction and compressed file candidates
Search for:
- `ZipInputStream`
- `ZipFile`
- `ZipEntry`
- `entry.getName`
- `getNextEntry`
- `JarFile`
- `JarEntry`
- `TarArchiveInputStream`
- `TarArchiveEntry`
- `ArchiveInputStream`
- `ArchiveEntry`
- `SevenZFile`
- `GZIPInputStream`
- `InflaterInputStream`
- `Files.copy(zipInputStream`
- `extract`
- `unzip`
- `untar`

## 2.7 Required-control candidates
Search near sinks for:
- `toRealPath`
- `getCanonicalPath`
- `getCanonicalFile`
- `startsWith`
- `basePath`
- `baseDir`
- `rootDir`
- `normalize`
- `relativize`
- `LinkOption.NOFOLLOW_LINKS`
- `NOFOLLOW_LINKS`
- `SecureDirectoryStream`
- `isSameFile`
- `Files.isSymbolicLink`
- `FilenameUtils.getName`
- `StringUtils.cleanPath`
- `UUID.randomUUID`
- `allowedExtensions`
- `allowedMimeTypes`
- `ContentDisposition`
- `validate`
- `allowlist`
- `safePath`
- `safeFilename`
- `reject`
- `Zip Slip`

## 2.8 Java graph search recipes
Useful combinations:

```text
@GetMapping + Files.readAllBytes
@PostMapping + MultipartFile.transferTo
@RequestParam + Paths.get
@PathVariable + FileSystemResource
@DeleteMapping + Files.delete
@KafkaListener + Files.write
@Scheduled + cleanup + Files.delete
ZipInputStream + entry.getName + Files.copy
ResourceLoader.getResource + request parameter
StringUtils.cleanPath without toRealPath containment
Paths.get + normalize + startsWith
```

---

# 3. Java Path Traversal Anti-Patterns

### A1. Concatenated read path
```java
File f = new File(baseDir + "/" + fileName);
byte[] data = Files.readAllBytes(f.toPath());
```

Why risky:
User-controlled `fileName` may escape the intended base directory.

### A2. Normalize-only containment
```java
Path p = Paths.get(baseDir, userPath).normalize();
if (p.startsWith(baseDir)) {
    return Files.readString(p);
}
```

Why risky:
`normalize()` and simple prefix checks may not be sufficient depending on how containment is validated and used.

### A3. Unsafe delete path
```java
Files.delete(Paths.get(baseDir, userPath));
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Zip extraction without entry validation
```java
Path out = Paths.get(destDir, entry.getName());
Files.copy(zipStream, out);
```

Why risky:
Archive entry names may traverse outside the extraction root.

### A5. Resource/template selection by raw path
```java
return resourceLoader.getResource("file:" + templatePath);
```

Why risky:
Local resource loading may expose unintended files if `templatePath` is attacker-controlled.

---

# 4. Case Templates

## Case J-PATH-1: Arbitrary read via joined path

### Vulnerable pattern
```java
Path p = Paths.get(baseDir, userPath);
return Files.readString(p);
```

### Audit focus
Verify whether `userPath` is attacker-controlled and whether real containment is enforced before use.

## Case J-PATH-2: Unsafe archive extraction

### Vulnerable pattern
```java
Path out = Paths.get(destDir, entry.getName());
Files.copy(zipStream, out);
```

### Audit focus
Verify entry-path containment and overwrite behavior during extraction.

## Case J-PATH-3: Delete or overwrite path abuse

### Vulnerable pattern
```java
Files.delete(Paths.get(baseDir, userPath));
```

### Audit focus
Verify canonical containment and whether delete/write operations share the same path validation rules as read operations.

## Case J-PATH-4: Template or local resource path control

### Vulnerable pattern
```java
resourceLoader.getResource("file:" + templatePath);
```

### Audit focus
Verify whether raw path input can escape intended local resource scope.

---

# 5. Java-Specific Audit Heuristics

## 4.1 `Path` / `Paths` heuristics
Pay attention to:
- `Paths.get(...)`
- `resolve(...)`
- `normalize()`
- `toRealPath()`
- differences between string checks and resolved-path checks

## 4.2 `Files.*` heuristics
Pay attention to:
- `readAllBytes`
- `readString`
- `copy`
- `move`
- `delete`
- `exists`
- temp-file and cleanup logic

## 4.3 Spring/resource heuristics
Pay attention to:
- download and `Resource` handlers
- file-serving endpoints
- template/resource lookup by name
- local file vs classpath resource confusion

## 4.4 Archive heuristics
Pay attention to:
- `ZipInputStream`
- archive entry path usage
- extraction-root enforcement
- overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs admin/import path
