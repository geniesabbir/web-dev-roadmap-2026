# Day 2: Image Optimization - Faster Visual Content

## Introduction

Images typically account for 50% or more of a webpage's total size. Optimizing images is one of the highest-impact performance improvements you can make. Today, you'll learn modern image formats, responsive images, lazy loading, and automated optimization pipelines.

## Learning Objectives

By the end of this lesson, you will be able to:
- Choose the right image format for each use case
- Implement responsive images with srcset and sizes
- Set up lazy loading effectively
- Automate image optimization in your build process
- Use CDNs and image optimization services

---

## Image Format Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    IMAGE FORMAT GUIDE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FORMAT    USE CASE              PROS              CONS          │
│  ──────    ────────              ────              ────          │
│                                                                  │
│  AVIF      Photos, graphics      Best compression   Limited      │
│            (hero images)         30-50% smaller     browser      │
│                                  than WebP          support      │
│                                                                  │
│  WebP      General purpose       Good compression   Old browser  │
│            (all images)          25-35% smaller     fallback     │
│                                  than JPEG          needed       │
│                                                                  │
│  JPEG      Photos                Universal support  No alpha     │
│            (fallback)            Good compression   Lossy only   │
│                                                                  │
│  PNG       Transparency,         Lossless          Large files  │
│            screenshots           Alpha channel      for photos   │
│                                                                  │
│  SVG       Icons, logos,         Scalable          Not for      │
│            illustrations         Tiny file size     photos       │
│                                  Animatable                      │
│                                                                  │
│  GIF       Simple animations     Universal          256 colors   │
│            (use video instead)   Support            Large files  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Browser Support Strategy

```html
<!-- Using picture element for format fallback -->
<picture>
  <!-- AVIF for browsers that support it -->
  <source
    type="image/avif"
    srcset="image.avif"
  />
  <!-- WebP fallback -->
  <source
    type="image/webp"
    srcset="image.webp"
  />
  <!-- JPEG fallback for older browsers -->
  <img
    src="image.jpg"
    alt="Description"
    width="800"
    height="600"
  />
</picture>

<!-- Combined with responsive images -->
<picture>
  <source
    type="image/avif"
    srcset="
      image-400.avif 400w,
      image-800.avif 800w,
      image-1200.avif 1200w
    "
    sizes="(max-width: 600px) 100vw, 50vw"
  />
  <source
    type="image/webp"
    srcset="
      image-400.webp 400w,
      image-800.webp 800w,
      image-1200.webp 1200w
    "
    sizes="(max-width: 600px) 100vw, 50vw"
  />
  <img
    src="image-800.jpg"
    srcset="
      image-400.jpg 400w,
      image-800.jpg 800w,
      image-1200.jpg 1200w
    "
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="Description"
    width="800"
    height="600"
  />
</picture>
```

---

## Responsive Images

### srcset and sizes

```html
<!-- srcset with width descriptors -->
<img
  src="image-800.jpg"
  srcset="
    image-400.jpg 400w,
    image-800.jpg 800w,
    image-1200.jpg 1200w,
    image-1600.jpg 1600w
  "
  sizes="
    (max-width: 600px) 100vw,
    (max-width: 1200px) 50vw,
    33vw
  "
  alt="Responsive image"
  width="800"
  height="600"
/>

<!--
  sizes breakdown:
  - Up to 600px viewport: image fills 100% of viewport width
  - 600px to 1200px: image fills 50% of viewport width
  - Above 1200px: image fills 33% of viewport width

  Browser calculation:
  - 400px viewport → needs 400w image (100vw)
  - 800px viewport → needs 400w image (50vw = 400px)
  - 1600px viewport → needs 533w image (33vw = 528px) → picks 800w
-->

<!-- srcset with pixel density descriptors (for fixed-size images) -->
<img
  src="logo.png"
  srcset="
    logo.png 1x,
    logo@2x.png 2x,
    logo@3x.png 3x
  "
  alt="Logo"
  width="200"
  height="50"
/>
```

### Art Direction with Picture

```html
<!-- Different crops for different viewports -->
<picture>
  <!-- Mobile: square crop, portrait oriented -->
  <source
    media="(max-width: 600px)"
    srcset="hero-mobile.jpg"
  />

  <!-- Tablet: 4:3 crop -->
  <source
    media="(max-width: 1024px)"
    srcset="hero-tablet.jpg"
  />

  <!-- Desktop: wide cinematic crop -->
  <img
    src="hero-desktop.jpg"
    alt="Hero image"
    width="1600"
    height="600"
  />
</picture>

<!-- Dark mode support -->
<picture>
  <source
    media="(prefers-color-scheme: dark)"
    srcset="logo-dark.svg"
  />
  <img
    src="logo-light.svg"
    alt="Logo"
    width="200"
    height="50"
  />
</picture>
```

---

## Lazy Loading

### Native Lazy Loading

```html
<!-- Native lazy loading (widely supported) -->
<img
  src="image.jpg"
  alt="Lazy loaded image"
  loading="lazy"
  width="800"
  height="600"
/>

<!-- Eager loading for above-the-fold images -->
<img
  src="hero.jpg"
  alt="Hero image"
  loading="eager"
  fetchpriority="high"
  width="1200"
  height="600"
/>

<!-- Lazy loading iframes -->
<iframe
  src="https://www.youtube.com/embed/..."
  loading="lazy"
  width="560"
  height="315"
></iframe>
```

### Intersection Observer for Advanced Control

```javascript
// Custom lazy loading with blur-up effect
class LazyLoader {
  constructor(options = {}) {
    this.options = {
      rootMargin: '50px 0px',
      threshold: 0.01,
      ...options
    };

    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      this.options
    );
  }

  observe(images) {
    images.forEach(img => {
      // Set placeholder
      if (img.dataset.placeholder) {
        img.src = img.dataset.placeholder;
      }

      this.observer.observe(img);
    });
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }

  loadImage(img) {
    const src = img.dataset.src;
    const srcset = img.dataset.srcset;

    // Create temporary image to preload
    const tempImage = new Image();

    tempImage.onload = () => {
      img.src = src;
      if (srcset) {
        img.srcset = srcset;
      }
      img.classList.add('loaded');
    };

    tempImage.src = src;
  }
}

// Usage
const lazyLoader = new LazyLoader({ rootMargin: '100px' });

document.addEventListener('DOMContentLoaded', () => {
  const lazyImages = document.querySelectorAll('img[data-src]');
  lazyLoader.observe(lazyImages);
});

// HTML
// <img
//   data-src="image.jpg"
//   data-srcset="image-400.jpg 400w, image-800.jpg 800w"
//   data-placeholder="placeholder.jpg"
//   alt="Lazy image"
// />
```

### React Lazy Loading Component

```jsx
// components/LazyImage.jsx
import { useState, useEffect, useRef } from 'react';

export function LazyImage({
  src,
  srcSet,
  sizes,
  alt,
  width,
  height,
  placeholder,
  className = ''
}) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div
      ref={imgRef}
      className={`lazy-image-container ${className}`}
      style={{
        aspectRatio: `${width} / ${height}`,
        backgroundColor: '#f0f0f0'
      }}
    >
      {/* Low quality placeholder */}
      {placeholder && !isLoaded && (
        <img
          src={placeholder}
          alt=""
          className="lazy-image-placeholder"
          style={{
            filter: 'blur(10px)',
            transform: 'scale(1.1)'
          }}
        />
      )}

      {/* Actual image */}
      {isInView && (
        <img
          src={src}
          srcSet={srcSet}
          sizes={sizes}
          alt={alt}
          width={width}
          height={height}
          onLoad={() => setIsLoaded(true)}
          className={`lazy-image ${isLoaded ? 'loaded' : ''}`}
          style={{
            opacity: isLoaded ? 1 : 0,
            transition: 'opacity 0.3s ease'
          }}
        />
      )}
    </div>
  );
}

// CSS
// .lazy-image-container {
//   position: relative;
//   overflow: hidden;
// }
// .lazy-image-placeholder {
//   position: absolute;
//   inset: 0;
//   width: 100%;
//   height: 100%;
//   object-fit: cover;
// }
// .lazy-image {
//   width: 100%;
//   height: 100%;
//   object-fit: cover;
// }
```

---

## Image Optimization Tools

### Sharp for Node.js

```javascript
// image-optimizer.js
const sharp = require('sharp');
const path = require('path');
const fs = require('fs').promises;

class ImageOptimizer {
  constructor(options = {}) {
    this.options = {
      quality: 80,
      sizes: [400, 800, 1200, 1600],
      formats: ['avif', 'webp', 'jpeg'],
      ...options
    };
  }

  async optimize(inputPath, outputDir) {
    const filename = path.parse(inputPath).name;
    const results = [];

    for (const size of this.options.sizes) {
      for (const format of this.options.formats) {
        const outputPath = path.join(
          outputDir,
          `${filename}-${size}.${format}`
        );

        await this.processImage(inputPath, outputPath, size, format);
        results.push(outputPath);
      }
    }

    return results;
  }

  async processImage(input, output, width, format) {
    let pipeline = sharp(input)
      .resize(width, null, {
        withoutEnlargement: true,
        fit: 'inside'
      });

    switch (format) {
      case 'avif':
        pipeline = pipeline.avif({
          quality: this.options.quality,
          effort: 4
        });
        break;
      case 'webp':
        pipeline = pipeline.webp({
          quality: this.options.quality,
          effort: 4
        });
        break;
      case 'jpeg':
        pipeline = pipeline.jpeg({
          quality: this.options.quality,
          progressive: true,
          mozjpeg: true
        });
        break;
      case 'png':
        pipeline = pipeline.png({
          quality: this.options.quality,
          compressionLevel: 9
        });
        break;
    }

    await pipeline.toFile(output);

    // Log file size
    const stats = await fs.stat(output);
    console.log(`Created: ${output} (${(stats.size / 1024).toFixed(1)} KB)`);
  }

  // Generate blur placeholder (LQIP - Low Quality Image Placeholder)
  async generatePlaceholder(inputPath) {
    const buffer = await sharp(inputPath)
      .resize(20, 20, { fit: 'inside' })
      .blur(5)
      .jpeg({ quality: 30 })
      .toBuffer();

    return `data:image/jpeg;base64,${buffer.toString('base64')}`;
  }

  // Generate BlurHash
  async generateBlurHash(inputPath) {
    const { encode } = await import('blurhash');
    const { data, info } = await sharp(inputPath)
      .raw()
      .ensureAlpha()
      .resize(32, 32, { fit: 'inside' })
      .toBuffer({ resolveWithObject: true });

    return encode(
      new Uint8ClampedArray(data),
      info.width,
      info.height,
      4,
      3
    );
  }
}

// Usage
const optimizer = new ImageOptimizer({
  quality: 85,
  sizes: [400, 800, 1200]
});

await optimizer.optimize('./images/hero.jpg', './public/images');
```

### Build-Time Optimization

```javascript
// next.config.js - Next.js image optimization
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1 year
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
};

// Webpack image optimization
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|webp|avif)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 4 * 1024 // 4kb - inline as base64
          }
        },
        generator: {
          filename: 'images/[name].[contenthash][ext]'
        }
      }
    ]
  },
  plugins: [
    new ImageMinimizerPlugin({
      minimizer: {
        implementation: ImageMinimizerPlugin.sharpMinify,
        options: {
          encodeOptions: {
            jpeg: { quality: 80 },
            webp: { quality: 80 },
            avif: { quality: 75 }
          }
        }
      },
      generator: [
        {
          preset: 'webp',
          implementation: ImageMinimizerPlugin.sharpGenerate,
          options: {
            encodeOptions: {
              webp: { quality: 80 }
            }
          }
        }
      ]
    })
  ]
};
```

### Vite Plugin

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
  plugins: [
    imagetools({
      defaultDirectives: (url) => {
        if (url.searchParams.has('hero')) {
          return new URLSearchParams({
            format: 'avif;webp;jpg',
            width: '400;800;1200;1600',
            quality: '80'
          });
        }
        return new URLSearchParams();
      }
    })
  ]
});

// Usage in code
import heroImage from './hero.jpg?hero';
// Returns: { avif: [...], webp: [...], jpg: [...] }
```

---

## Image CDN Services

### Cloudinary

```javascript
// Cloudinary URL transformations
const cloudinaryUrl = (publicId, options = {}) => {
  const {
    width,
    height,
    crop = 'fill',
    quality = 'auto',
    format = 'auto',
    gravity = 'auto'
  } = options;

  const transforms = [
    width && `w_${width}`,
    height && `h_${height}`,
    `c_${crop}`,
    `q_${quality}`,
    `f_${format}`,
    gravity !== 'auto' && `g_${gravity}`
  ].filter(Boolean).join(',');

  return `https://res.cloudinary.com/YOUR_CLOUD/${transforms}/${publicId}`;
};

// Usage
const heroUrl = cloudinaryUrl('hero.jpg', {
  width: 1200,
  height: 600,
  crop: 'fill',
  gravity: 'face' // Focus on faces
});

// React component with Cloudinary
function CloudinaryImage({ publicId, alt, width, height, ...props }) {
  const baseUrl = `https://res.cloudinary.com/YOUR_CLOUD`;

  const srcSet = [400, 800, 1200, 1600]
    .map(w => `${baseUrl}/w_${w},q_auto,f_auto/${publicId} ${w}w`)
    .join(', ');

  return (
    <img
      src={`${baseUrl}/w_${width},q_auto,f_auto/${publicId}`}
      srcSet={srcSet}
      sizes={props.sizes || '100vw'}
      alt={alt}
      width={width}
      height={height}
      loading="lazy"
      {...props}
    />
  );
}
```

### imgix

```javascript
// imgix URL builder
import ImgixClient from '@imgix/js-core';

const client = new ImgixClient({
  domain: 'your-domain.imgix.net',
  secureURLToken: 'your-token'
});

// Generate responsive URLs
function getImgixSrcSet(path, options = {}) {
  const { widths = [400, 800, 1200, 1600], ...params } = options;

  return client.buildSrcSet(path, {
    auto: 'format,compress',
    ...params
  }, {
    widths
  });
}

// Usage
const srcSet = getImgixSrcSet('hero.jpg', {
  fit: 'crop',
  crop: 'faces',
  ar: '16:9'
});

// React component
function ImgixImage({ path, alt, width, height, sizes }) {
  const src = client.buildURL(path, {
    w: width,
    auto: 'format,compress'
  });

  const srcSet = client.buildSrcSet(path, {
    auto: 'format,compress'
  });

  return (
    <img
      src={src}
      srcSet={srcSet}
      sizes={sizes}
      alt={alt}
      width={width}
      height={height}
      loading="lazy"
    />
  );
}
```

### Vercel Image Optimization

```jsx
// Next.js Image component (built-in optimization)
import Image from 'next/image';

// Basic usage
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority
/>

// With blur placeholder
import heroImage from '../public/hero.jpg';

<Image
  src={heroImage}
  alt="Hero"
  placeholder="blur"
  priority
/>

// Remote images
<Image
  src="https://example.com/image.jpg"
  alt="Remote image"
  width={800}
  height={600}
/>

// Fill container
<div style={{ position: 'relative', width: '100%', height: '400px' }}>
  <Image
    src="/hero.jpg"
    alt="Hero"
    fill
    style={{ objectFit: 'cover' }}
  />
</div>

// Custom loader for external CDN
const cloudinaryLoader = ({ src, width, quality }) => {
  return `https://res.cloudinary.com/YOUR_CLOUD/image/upload/w_${width},q_${quality || 75}/${src}`;
};

<Image
  loader={cloudinaryLoader}
  src="hero.jpg"
  alt="Hero"
  width={800}
  height={600}
/>
```

---

## SVG Optimization

```javascript
// svgo.config.js
module.exports = {
  plugins: [
    'preset-default',
    'prefixIds',
    {
      name: 'removeViewBox',
      active: false
    },
    {
      name: 'removeDimensions',
      active: true
    }
  ]
};

// Inline SVG component
function Icon({ name, size = 24, className }) {
  return (
    <svg
      width={size}
      height={size}
      className={className}
      aria-hidden="true"
    >
      <use href={`/icons.svg#${name}`} />
    </svg>
  );
}

// icons.svg (sprite)
// <svg xmlns="http://www.w3.org/2000/svg">
//   <symbol id="arrow" viewBox="0 0 24 24">
//     <path d="..."/>
//   </symbol>
//   <symbol id="check" viewBox="0 0 24 24">
//     <path d="..."/>
//   </symbol>
// </svg>

// React SVG import (with SVGR)
import { ReactComponent as Logo } from './logo.svg';

<Logo className="logo" aria-label="Company Logo" />
```

---

## Performance Patterns

### Progressive Image Loading

```jsx
// components/ProgressiveImage.jsx
import { useState } from 'react';

export function ProgressiveImage({
  lowQualitySrc,
  highQualitySrc,
  alt,
  width,
  height
}) {
  const [highQualityLoaded, setHighQualityLoaded] = useState(false);

  return (
    <div
      className="progressive-image"
      style={{ aspectRatio: `${width} / ${height}` }}
    >
      {/* Low quality placeholder */}
      <img
        src={lowQualitySrc}
        alt=""
        className="progressive-image-low"
        style={{
          filter: highQualityLoaded ? 'none' : 'blur(20px)',
          opacity: highQualityLoaded ? 0 : 1
        }}
      />

      {/* High quality image */}
      <img
        src={highQualitySrc}
        alt={alt}
        width={width}
        height={height}
        onLoad={() => setHighQualityLoaded(true)}
        className="progressive-image-high"
        style={{ opacity: highQualityLoaded ? 1 : 0 }}
      />
    </div>
  );
}

// CSS
// .progressive-image {
//   position: relative;
//   overflow: hidden;
// }
// .progressive-image img {
//   position: absolute;
//   inset: 0;
//   width: 100%;
//   height: 100%;
//   object-fit: cover;
//   transition: opacity 0.3s, filter 0.3s;
// }
```

### BlurHash Placeholders

```jsx
// Using BlurHash for placeholders
import { Blurhash } from 'react-blurhash';
import { useState } from 'react';

export function BlurHashImage({
  src,
  blurHash,
  alt,
  width,
  height
}) {
  const [loaded, setLoaded] = useState(false);

  return (
    <div
      style={{
        position: 'relative',
        aspectRatio: `${width} / ${height}`
      }}
    >
      {!loaded && (
        <Blurhash
          hash={blurHash}
          width="100%"
          height="100%"
          resolutionX={32}
          resolutionY={32}
          punch={1}
          style={{ position: 'absolute', inset: 0 }}
        />
      )}

      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        onLoad={() => setLoaded(true)}
        style={{
          opacity: loaded ? 1 : 0,
          transition: 'opacity 0.3s'
        }}
      />
    </div>
  );
}

// Generate BlurHash server-side
// api/blurhash.js
import { encode } from 'blurhash';
import sharp from 'sharp';

export async function generateBlurHash(imagePath) {
  const { data, info } = await sharp(imagePath)
    .raw()
    .ensureAlpha()
    .resize(32, 32, { fit: 'inside' })
    .toBuffer({ resolveWithObject: true });

  return encode(
    new Uint8ClampedArray(data),
    info.width,
    info.height,
    4,
    3
  );
}
```

---

## Image Audit Checklist

```
┌─────────────────────────────────────────────────────────────────┐
│                    IMAGE OPTIMIZATION CHECKLIST                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FORMAT:                                                         │
│  □ Using AVIF/WebP with fallbacks                               │
│  □ SVG for icons and logos                                      │
│  □ Video for animated content (not GIF)                         │
│                                                                  │
│  SIZE:                                                           │
│  □ Multiple sizes generated for responsive                       │
│  □ Images sized appropriately for display size                  │
│  □ Not serving larger images than needed                        │
│                                                                  │
│  LOADING:                                                        │
│  □ Lazy loading for below-fold images                           │
│  □ Priority loading for LCP images                              │
│  □ Preload for critical images                                  │
│                                                                  │
│  LAYOUT:                                                         │
│  □ Width and height attributes set                              │
│  □ Aspect ratio reserved                                        │
│  □ Placeholders prevent CLS                                     │
│                                                                  │
│  OPTIMIZATION:                                                   │
│  □ Compressed to target quality                                 │
│  □ Metadata stripped                                            │
│  □ Using CDN with caching                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practice Exercises

### Exercise 1: Responsive Hero

Create a responsive hero image that:
- Uses AVIF, WebP, and JPEG
- Has 4 size variants
- Includes a blur placeholder
- Uses priority loading

### Exercise 2: Image Gallery

Build a lazy-loaded gallery that:
- Uses Intersection Observer
- Shows placeholders while loading
- Optimizes for different screen sizes
- Implements progressive loading

### Exercise 3: Build Pipeline

Set up an image optimization pipeline:
- Automatic format conversion
- Size generation
- Placeholder generation
- Integration with your build system

---

## Key Takeaways

1. **Modern formats first** - Use AVIF and WebP with fallbacks
2. **Right size for device** - Never serve larger than needed
3. **Lazy load below-fold** - But prioritize LCP images
4. **Always set dimensions** - Prevent layout shifts
5. **Automate optimization** - Don't rely on manual compression
6. **Use CDN** - Edge caching and on-the-fly optimization

---

## What's Next?

Tomorrow, we'll explore **Code Splitting** - learning how to break up your JavaScript bundles to load only what's needed and improve Time to Interactive.
