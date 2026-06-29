# File Uploads

_Part of [Express.js](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. How do you handle file uploads in Express?](#1-how-do-you-handle-file-uploads-in-express)
- [2. What is `multer`, and why is it commonly used?](#2-what-is-multer-and-why-is-it-commonly-used)

**🟡 Medium**
- [3. How do you restrict uploaded file types/size with multer?](#3-how-do-you-restrict-uploaded-file-typessize-with-multer)
- [4. How do you store uploaded files on disk vs. in memory?](#4-how-do-you-store-uploaded-files-on-disk-vs-in-memory)
- [5. How do you handle multiple file uploads in a single request?](#5-how-do-you-handle-multiple-file-uploads-in-a-single-request)

**🔴 Hard**
- [6. How would you stream a large file upload directly to cloud storage instead of buffering it on the server?](#6-how-would-you-stream-a-large-file-upload-directly-to-cloud-storage-instead-of-buffering-it-on-the-server)
- [7. How do you validate actual file content, not just the extension?](#7-how-do-you-validate-actual-file-content-not-just-the-extension)
- [8. How would you generate unique filenames to avoid collisions/overwrites?](#8-how-would-you-generate-unique-filenames-to-avoid-collisionsoverwrites)
- [9. How do you handle upload errors (e.g. file too large) gracefully?](#9-how-do-you-handle-upload-errors-eg-file-too-large-gracefully)
- [10. What security risks come with accepting file uploads?](#10-what-security-risks-come-with-accepting-file-uploads)

---

### 1. How do you handle file uploads in Express? 🟢

- Express has no built-in multipart form parsing — use `multer` (the standard middleware for `multipart/form-data`).

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file });
});
```

[↑ Back to top](#table-of-contents)

---

### 2. What is `multer`, and why is it commonly used? 🟢

- Middleware specifically for parsing `multipart/form-data` (the encoding used by file-upload forms) — populates `req.file`/`req.files` with metadata about the uploaded file(s), and `req.body` with any other form fields.

[↑ Back to top](#table-of-contents)

---

### 3. How do you restrict uploaded file types/size with multer? 🟡

```js
const upload = multer({
  dest: 'uploads/',
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB max
  fileFilter: (req, file, cb) => {
    const allowed = ['image/png', 'image/jpeg'];
    cb(null, allowed.includes(file.mimetype));
  },
});
```

[↑ Back to top](#table-of-contents)

---

### 4. How do you store uploaded files on disk vs. in memory? 🟡

- `multer.diskStorage()`: writes directly to disk — better for large files, doesn't hold the whole file in process memory.
- `multer.memoryStorage()`: keeps the file as a `Buffer` in `req.file.buffer` — convenient when immediately forwarding the file elsewhere (e.g. uploading to S3) without needing a temp file on disk, but risky for large files since it consumes memory per upload.

```js
const upload = multer({ storage: multer.memoryStorage() });
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you handle multiple file uploads in a single request? 🟡

```js
upload.array('photos', 5); // up to 5 files under the same field name -> req.files (array)
upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 10 }]); // different field names
```

[↑ Back to top](#table-of-contents)

---

### 6. How would you stream a large file upload directly to cloud storage instead of buffering it on the server? 🔴

- Use a streaming-capable approach instead of multer's default disk/memory buffering — pipe the incoming multipart stream directly to the destination (e.g. S3's multipart upload API) as it arrives, so the file never fully sits on the Express server's disk or memory at once.

```js
// Conceptual: stream directly to S3 instead of writing locally first
busboy.on('file', (fieldname, fileStream) => {
  s3Upload.uploadStream(fileStream); // pipe straight through
});
```

[↑ Back to top](#table-of-contents)

---

### 7. How do you validate actual file content, not just the extension? 🔴

- Check the file's **magic bytes** (file signature) rather than trusting the extension or client-supplied `mimetype` (both are trivially spoofable) — libraries like `file-type` read the first few bytes to determine the real format.

```js
const FileType = require('file-type');
const type = await FileType.fromBuffer(buffer);
if (!type || !['image/png', 'image/jpeg'].includes(type.mime)) {
  throw new Error('Invalid file content');
}
```

[↑ Back to top](#table-of-contents)

---

### 8. How would you generate unique filenames to avoid collisions/overwrites? 🔴

- Don't trust the original filename — generate a new one (UUID, hash, timestamp) while preserving the validated extension.

```js
const storage = multer.diskStorage({
  filename: (req, file, cb) => {
    cb(null, `${crypto.randomUUID()}${path.extname(file.originalname)}`);
  },
});
```

[↑ Back to top](#table-of-contents)

---

### 9. How do you handle upload errors (e.g. file too large) gracefully? 🔴

- Multer raises a `MulterError` (e.g. `LIMIT_FILE_SIZE`) — catch it in error-handling middleware and return a clear, specific message instead of a generic 500.

```js
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    return res.status(400).json({ error: `Upload failed: ${err.code}` });
  }
  next(err);
});
```

[↑ Back to top](#table-of-contents)

---

### 10. What security risks come with accepting file uploads? 🔴

- **Malicious file content**: executable scripts disguised as images/documents.
- **Path traversal**: a crafted filename (`../../etc/passwd`) attempting to write outside the intended directory — mitigated by always generating your own filenames (Q8), never using the client-supplied one directly.
- **Resource exhaustion**: unbounded file size/count leading to disk/memory exhaustion — mitigated by size/count limits (Q3).
- **Serving uploads from an executable path**: if uploads are stored where the web server might execute them (e.g. inside a PHP-enabled directory on a misconfigured host), an uploaded script could run server-side.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is it safer to serve user-uploaded files from a separate domain/subdomain (or signed, time-limited URLs from cloud storage) rather than directly from your main app's domain?

[↑ Back to top](#table-of-contents)

---
