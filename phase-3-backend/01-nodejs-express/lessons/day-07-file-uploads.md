# Day 7: File Uploads in Express

## Introduction

Handling file uploads is a common requirement for web applications. This lesson covers uploading files with Multer, validating uploads, storing files locally and in the cloud, and implementing best practices for file handling.

## Learning Objectives

By the end of this lesson, you will:
- Handle file uploads with Multer
- Validate file types and sizes
- Store files locally and in cloud storage
- Process and transform images
- Implement secure file handling
- Build a complete file upload system

---

## Understanding File Uploads

### How File Uploads Work

```
Client (form/fetch)         Server (Express + Multer)
       │                            │
       │ POST multipart/form-data   │
       │ ─────────────────────────> │
       │                            │
       │ • File binary data         │ Parse multipart
       │ • Form fields              │ Store file
       │ • Boundary markers         │ Return response
       │                            │
       │ <───────────────────────── │
       │      { url: '/uploads/...' }│
```

### Multipart Form Data

```html
<!-- HTML Form -->
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="avatar" />
  <input type="text" name="username" />
  <button type="submit">Upload</button>
</form>
```

```javascript
// JavaScript Fetch
const formData = new FormData();
formData.append('avatar', fileInput.files[0]);
formData.append('username', 'john');

fetch('/upload', {
  method: 'POST',
  body: formData,
  // Don't set Content-Type header - browser sets it with boundary
});
```

---

## Multer Basics

### Installation

```bash
npm install multer
```

### Basic Setup

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');

const app = express();

// Basic disk storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({ storage });

// Single file upload
app.post('/upload', upload.single('avatar'), (req, res) => {
  console.log(req.file);  // Uploaded file info
  console.log(req.body);  // Other form fields

  res.json({
    message: 'File uploaded successfully',
    file: {
      filename: req.file.filename,
      size: req.file.size,
      mimetype: req.file.mimetype,
    }
  });
});

app.listen(3000);
```

### File Object Properties

```javascript
req.file = {
  fieldname: 'avatar',           // Form field name
  originalname: 'photo.jpg',     // Original filename
  encoding: '7bit',              // Encoding type
  mimetype: 'image/jpeg',        // MIME type
  destination: 'uploads/',       // Upload directory
  filename: 'avatar-1234567.jpg', // Saved filename
  path: 'uploads/avatar-1234567.jpg', // Full path
  size: 12345                    // File size in bytes
}
```

---

## Upload Types

### Single File

```javascript
// One file from field 'avatar'
app.post('/avatar', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file });
});
```

### Multiple Files (Same Field)

```javascript
// Up to 5 files from field 'photos'
app.post('/photos', upload.array('photos', 5), (req, res) => {
  res.json({ files: req.files }); // Array of files
});
```

### Multiple Fields

```javascript
const uploadFields = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 },
  { name: 'documents', maxCount: 3 }
]);

app.post('/profile', uploadFields, (req, res) => {
  res.json({
    avatar: req.files['avatar'],
    gallery: req.files['gallery'],
    documents: req.files['documents']
  });
});
```

### Any File

```javascript
// Accept any file field
app.post('/any', upload.any(), (req, res) => {
  res.json({ files: req.files });
});
```

### No Files (Only Form Data)

```javascript
// Parse multipart but no files
app.post('/data', upload.none(), (req, res) => {
  res.json({ body: req.body });
});
```

---

## File Validation

### File Type Validation

```javascript
const fileFilter = (req, file, cb) => {
  // Accept images only
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];

  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, GIF, and WebP are allowed.'), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
});
```

### File Size Limits

```javascript
const upload = multer({
  storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
    files: 5,                   // Max 5 files
    fields: 10,                 // Max 10 non-file fields
    fieldSize: 100 * 1024,      // Max 100KB per field
  }
});
```

### Custom Validation Middleware

```javascript
const validateImage = (req, file, cb) => {
  // Check file extension
  const ext = path.extname(file.originalname).toLowerCase();
  const allowedExts = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];

  if (!allowedExts.includes(ext)) {
    return cb(new Error(`Invalid extension: ${ext}`));
  }

  // Check MIME type
  const allowedMimes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];

  if (!allowedMimes.includes(file.mimetype)) {
    return cb(new Error(`Invalid MIME type: ${file.mimetype}`));
  }

  cb(null, true);
};

// Validate file content after upload
const validateFileContent = async (req, res, next) => {
  if (!req.file) return next();

  const fileType = require('file-type');
  const type = await fileType.fromFile(req.file.path);

  if (!type || !['image/jpeg', 'image/png'].includes(type.mime)) {
    // Delete the invalid file
    fs.unlinkSync(req.file.path);
    return res.status(400).json({ error: 'Invalid file content' });
  }

  next();
};
```

### Complete Validation Setup

```javascript
const createUploader = (options = {}) => {
  const {
    destination = 'uploads/',
    allowedTypes = ['image/jpeg', 'image/png', 'image/gif'],
    maxSize = 5 * 1024 * 1024,
    maxFiles = 5,
  } = options;

  const storage = multer.diskStorage({
    destination: (req, file, cb) => {
      cb(null, destination);
    },
    filename: (req, file, cb) => {
      const uniqueName = `${Date.now()}-${Math.random().toString(36).substring(7)}`;
      cb(null, `${uniqueName}${path.extname(file.originalname)}`);
    }
  });

  const fileFilter = (req, file, cb) => {
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error(`Invalid file type: ${file.mimetype}`));
    }
  };

  return multer({
    storage,
    fileFilter,
    limits: {
      fileSize: maxSize,
      files: maxFiles,
    }
  });
};

// Usage
const imageUpload = createUploader({
  destination: 'uploads/images/',
  allowedTypes: ['image/jpeg', 'image/png', 'image/webp'],
  maxSize: 2 * 1024 * 1024, // 2MB
});

const documentUpload = createUploader({
  destination: 'uploads/documents/',
  allowedTypes: ['application/pdf', 'application/msword'],
  maxSize: 10 * 1024 * 1024, // 10MB
});
```

---

## Error Handling

### Multer Error Handling

```javascript
const handleUpload = (req, res, next) => {
  upload.single('file')(req, res, (err) => {
    if (err instanceof multer.MulterError) {
      // Multer-specific errors
      switch (err.code) {
        case 'LIMIT_FILE_SIZE':
          return res.status(400).json({ error: 'File too large' });
        case 'LIMIT_FILE_COUNT':
          return res.status(400).json({ error: 'Too many files' });
        case 'LIMIT_UNEXPECTED_FILE':
          return res.status(400).json({ error: 'Unexpected field' });
        default:
          return res.status(400).json({ error: err.message });
      }
    } else if (err) {
      // Custom errors from fileFilter
      return res.status(400).json({ error: err.message });
    }
    next();
  });
};

app.post('/upload', handleUpload, (req, res) => {
  res.json({ file: req.file });
});
```

### Error Handling Middleware Factory

```javascript
const uploadMiddleware = (upload, fieldConfig) => {
  return (req, res, next) => {
    const handler = typeof fieldConfig === 'string'
      ? upload.single(fieldConfig)
      : Array.isArray(fieldConfig)
        ? upload.fields(fieldConfig)
        : upload.array(fieldConfig.name, fieldConfig.maxCount);

    handler(req, res, (err) => {
      if (err) {
        if (err instanceof multer.MulterError) {
          return res.status(400).json({
            error: 'Upload error',
            code: err.code,
            message: getMulterErrorMessage(err)
          });
        }
        return res.status(400).json({ error: err.message });
      }
      next();
    });
  };
};

const getMulterErrorMessage = (err) => {
  const messages = {
    LIMIT_FILE_SIZE: 'File is too large',
    LIMIT_FILE_COUNT: 'Too many files',
    LIMIT_FIELD_KEY: 'Field name too long',
    LIMIT_FIELD_VALUE: 'Field value too long',
    LIMIT_FIELD_COUNT: 'Too many fields',
    LIMIT_UNEXPECTED_FILE: 'Unexpected file field',
    LIMIT_PART_COUNT: 'Too many parts',
  };
  return messages[err.code] || err.message;
};
```

---

## Memory Storage

### Using Buffer Instead of Disk

```javascript
const memoryStorage = multer.memoryStorage();

const upload = multer({
  storage: memoryStorage,
  limits: { fileSize: 5 * 1024 * 1024 }
});

app.post('/upload', upload.single('image'), async (req, res) => {
  // File is in memory as buffer
  const buffer = req.file.buffer;

  // Process the buffer (e.g., upload to cloud, process image)
  const result = await uploadToCloud(buffer, req.file.originalname);

  res.json({ url: result.url });
});
```

---

## Cloud Storage Integration

### AWS S3 Upload

```bash
npm install @aws-sdk/client-s3 @aws-sdk/lib-storage
```

```javascript
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { Upload } = require('@aws-sdk/lib-storage');
const multer = require('multer');
const crypto = require('crypto');

// Configure S3
const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  }
});

// Memory storage for cloud upload
const upload = multer({ storage: multer.memoryStorage() });

// Upload to S3
const uploadToS3 = async (file) => {
  const key = `uploads/${Date.now()}-${crypto.randomBytes(8).toString('hex')}${path.extname(file.originalname)}`;

  const upload = new Upload({
    client: s3Client,
    params: {
      Bucket: process.env.S3_BUCKET,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
    }
  });

  const result = await upload.done();

  return {
    key,
    url: `https://${process.env.S3_BUCKET}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`,
    etag: result.ETag,
  };
};

app.post('/upload', upload.single('file'), async (req, res) => {
  try {
    const result = await uploadToS3(req.file);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: 'Upload failed' });
  }
});
```

### Cloudinary Integration

```bash
npm install cloudinary multer-storage-cloudinary
```

```javascript
const cloudinary = require('cloudinary').v2;
const { CloudinaryStorage } = require('multer-storage-cloudinary');
const multer = require('multer');

// Configure Cloudinary
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET
});

// Cloudinary storage for Multer
const storage = new CloudinaryStorage({
  cloudinary,
  params: {
    folder: 'uploads',
    allowed_formats: ['jpg', 'jpeg', 'png', 'gif', 'webp'],
    transformation: [{ width: 1000, height: 1000, crop: 'limit' }],
  }
});

const upload = multer({ storage });

app.post('/upload', upload.single('image'), (req, res) => {
  res.json({
    url: req.file.path,
    publicId: req.file.filename,
    format: req.file.format,
    width: req.file.width,
    height: req.file.height
  });
});

// Delete from Cloudinary
app.delete('/upload/:publicId', async (req, res) => {
  try {
    await cloudinary.uploader.destroy(req.params.publicId);
    res.json({ message: 'Deleted successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Delete failed' });
  }
});
```

---

## Image Processing

### Sharp for Image Manipulation

```bash
npm install sharp
```

```javascript
const sharp = require('sharp');
const path = require('path');
const fs = require('fs').promises;

const upload = multer({ storage: multer.memoryStorage() });

// Process and save images
app.post('/upload', upload.single('image'), async (req, res) => {
  const filename = `${Date.now()}-${Math.random().toString(36).substring(7)}`;

  // Create multiple sizes
  const sizes = [
    { name: 'thumb', width: 150, height: 150 },
    { name: 'medium', width: 500, height: 500 },
    { name: 'large', width: 1200, height: 1200 }
  ];

  const results = await Promise.all(
    sizes.map(async (size) => {
      const outputPath = path.join('uploads', `${filename}-${size.name}.webp`);

      await sharp(req.file.buffer)
        .resize(size.width, size.height, {
          fit: 'inside',
          withoutEnlargement: true
        })
        .webp({ quality: 80 })
        .toFile(outputPath);

      return { size: size.name, path: outputPath };
    })
  );

  res.json({ files: results });
});

// Image transformation middleware
const processImage = (options = {}) => async (req, res, next) => {
  if (!req.file) return next();

  const {
    width = 1000,
    height = 1000,
    quality = 80,
    format = 'webp'
  } = options;

  try {
    const processed = await sharp(req.file.buffer)
      .resize(width, height, { fit: 'inside', withoutEnlargement: true })
      [format]({ quality })
      .toBuffer();

    req.file.buffer = processed;
    req.file.mimetype = `image/${format}`;
    req.file.size = processed.length;

    next();
  } catch (error) {
    next(error);
  }
};

app.post('/avatar',
  upload.single('avatar'),
  processImage({ width: 200, height: 200, quality: 90 }),
  async (req, res) => {
    // Save processed image
    const filename = `avatar-${req.user.id}.webp`;
    await fs.writeFile(`uploads/${filename}`, req.file.buffer);
    res.json({ url: `/uploads/${filename}` });
  }
);
```

---

## Serving Uploaded Files

### Static File Serving

```javascript
const express = require('express');
const path = require('path');

const app = express();

// Serve uploaded files
app.use('/uploads', express.static(path.join(__dirname, 'uploads')));

// With cache headers
app.use('/uploads', express.static(path.join(__dirname, 'uploads'), {
  maxAge: '1d',
  etag: true,
  lastModified: true,
}));
```

### Protected File Serving

```javascript
const fs = require('fs');
const path = require('path');

// Check authorization before serving
app.get('/files/:filename', authenticate, async (req, res) => {
  const { filename } = req.params;
  const filePath = path.join(__dirname, 'uploads', filename);

  // Check if user has access
  const file = await File.findOne({ filename });

  if (!file || file.owner.toString() !== req.user.id) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // Check if file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }

  res.sendFile(filePath);
});
```

### Signed URLs

```javascript
const crypto = require('crypto');

// Generate signed URL
const generateSignedUrl = (filename, expiresIn = 3600) => {
  const expires = Math.floor(Date.now() / 1000) + expiresIn;
  const signature = crypto
    .createHmac('sha256', process.env.SIGNING_SECRET)
    .update(`${filename}${expires}`)
    .digest('hex');

  return `/files/${filename}?expires=${expires}&signature=${signature}`;
};

// Validate signed URL
const validateSignedUrl = (req, res, next) => {
  const { filename } = req.params;
  const { expires, signature } = req.query;

  if (!expires || !signature) {
    return res.status(403).json({ error: 'Invalid URL' });
  }

  // Check expiration
  if (parseInt(expires) < Math.floor(Date.now() / 1000)) {
    return res.status(403).json({ error: 'URL expired' });
  }

  // Validate signature
  const expectedSignature = crypto
    .createHmac('sha256', process.env.SIGNING_SECRET)
    .update(`${filename}${expires}`)
    .digest('hex');

  if (signature !== expectedSignature) {
    return res.status(403).json({ error: 'Invalid signature' });
  }

  next();
};

app.get('/files/:filename', validateSignedUrl, (req, res) => {
  res.sendFile(path.join(__dirname, 'uploads', req.params.filename));
});
```

---

## Complete File Upload System

```javascript
// config/upload.js
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');

const createStorage = (destination) => {
  return multer.diskStorage({
    destination: (req, file, cb) => {
      cb(null, destination);
    },
    filename: (req, file, cb) => {
      const uniqueName = crypto.randomBytes(16).toString('hex');
      cb(null, `${uniqueName}${path.extname(file.originalname)}`);
    }
  });
};

const imageFilter = (req, file, cb) => {
  const allowed = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  cb(null, allowed.includes(file.mimetype));
};

const documentFilter = (req, file, cb) => {
  const allowed = ['application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document'];
  cb(null, allowed.includes(file.mimetype));
};

module.exports = {
  imageUpload: multer({
    storage: createStorage('uploads/images'),
    fileFilter: imageFilter,
    limits: { fileSize: 5 * 1024 * 1024 }
  }),
  documentUpload: multer({
    storage: createStorage('uploads/documents'),
    fileFilter: documentFilter,
    limits: { fileSize: 10 * 1024 * 1024 }
  }),
  avatarUpload: multer({
    storage: multer.memoryStorage(),
    fileFilter: imageFilter,
    limits: { fileSize: 2 * 1024 * 1024 }
  })
};

// routes/uploads.js
const express = require('express');
const router = express.Router();
const sharp = require('sharp');
const fs = require('fs').promises;
const path = require('path');
const { imageUpload, documentUpload, avatarUpload } = require('../config/upload');
const { authenticate } = require('../middleware/auth');

// Upload image
router.post('/images', authenticate, imageUpload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No image provided' });
  }

  res.status(201).json({
    filename: req.file.filename,
    url: `/uploads/images/${req.file.filename}`,
    size: req.file.size,
    mimetype: req.file.mimetype
  });
});

// Upload avatar with processing
router.post('/avatar', authenticate, avatarUpload.single('avatar'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No image provided' });
  }

  const filename = `avatar-${req.user.id}.webp`;
  const outputPath = path.join('uploads/avatars', filename);

  await sharp(req.file.buffer)
    .resize(200, 200, { fit: 'cover' })
    .webp({ quality: 85 })
    .toFile(outputPath);

  res.json({
    url: `/uploads/avatars/${filename}`
  });
});

// Upload document
router.post('/documents', authenticate, documentUpload.single('document'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No document provided' });
  }

  res.status(201).json({
    filename: req.file.filename,
    url: `/uploads/documents/${req.file.filename}`,
    originalName: req.file.originalname,
    size: req.file.size
  });
});

// Delete file
router.delete('/:type/:filename', authenticate, async (req, res) => {
  const { type, filename } = req.params;
  const validTypes = ['images', 'documents', 'avatars'];

  if (!validTypes.includes(type)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }

  const filePath = path.join('uploads', type, filename);

  try {
    await fs.access(filePath);
    await fs.unlink(filePath);
    res.json({ message: 'File deleted' });
  } catch {
    res.status(404).json({ error: 'File not found' });
  }
});

module.exports = router;
```

---

## Key Takeaways

1. **Use Multer** - Standard library for handling multipart forms
2. **Validate thoroughly** - Check type, size, and content
3. **Use memory storage for cloud** - Upload buffer directly
4. **Process images with Sharp** - Resize, compress, convert
5. **Secure file serving** - Validate access, use signed URLs
6. **Clean up on errors** - Delete partial uploads
7. **Organize by type** - Separate images, documents, avatars

---

## What's Next?

Next we'll build a complete **REST API Project** over days 8-10, bringing together everything you've learned about Node.js and Express!
