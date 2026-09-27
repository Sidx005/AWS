# AWS S3 Static Website Hosting

**S3 Static Website Hosting** allows you to host a **static website directly from an S3 bucket** without running a web server.

Static websites typically contain:

* HTML
* CSS
* JavaScript
* Images
* Other static assets

### Architecture

```text
User
  ↓
Internet
  ↓
Amazon S3
  ↓
S3 Bucket
  ├── index.html
  ├── style.css
  ├── script.js
  └── images/
```

---

## 1. Create an S3 Bucket

Create an S3 bucket with a unique name.

Example:

```text
my-static-website-12345
```

For a traditional S3 website endpoint, the website's objects need to be accessible to website visitors, so the bucket's public-access configuration and bucket policy must be configured accordingly.

---

## 2. Upload Website Files

Upload your website files to the bucket.

Example:

```text
my-static-website-12345/
│
├── index.html
├── error.html
├── style.css
├── script.js
└── images/
    ├── logo.png
    └── banner.jpg
```

### Important

The main page is normally:

```text
index.html
```

---

## 3. Enable Static Website Hosting

Go to:

```text
S3
 → Your Bucket
 → Properties
 → Static website hosting
```

Choose:

```text
Static website hosting: Enable
```

Set:

```text
Index document: index.html
Error document: error.html
```

Then save the changes.

---

## 4. Allow Public Access

For the **classic S3 website endpoint**, visitors need permission to read the website objects.

A bucket policy can allow public `GetObject`.

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-website-12345/*"
    }
  ]
}
```

### What this means

```text
Principal: *
        ↓
Anyone

Action: s3:GetObject
        ↓
Can read/download objects

Resource: bucket/*
        ↓
All objects inside the bucket
```

So visitors can retrieve the HTML, CSS, JavaScript, images, etc.

---

## 5. Block Public Access

This is an important point from the previous S3-security topic.

If **Block Public Access** prevents the public bucket policy from taking effect, the traditional public S3 website endpoint won't work for anonymous visitors.

So you need to carefully configure public access if you're intentionally using the classic S3 website endpoint.

> **Do not disable public-access protections for a bucket unless public access is actually required.**

---

# 6. Access the Website

After enabling website hosting, S3 provides a **Website endpoint**.

It looks roughly like:

```text
http://bucket-name.s3-website-region.amazonaws.com
```

For example:

```text
http://my-static-website-12345.s3-website-us-east-1.amazonaws.com
```

When someone visits the endpoint:

```text
Browser
   ↓
S3 Website Endpoint
   ↓
index.html
   ↓
Website displayed
```

---

# 7. What Happens When a User Visits?

Suppose the user visits:

```text
http://my-static-website-12345.s3-website-us-east-1.amazonaws.com
```

S3 looks for:

```text
index.html
```

and returns it.

Then the browser requests:

```text
style.css
script.js
images/logo.png
```

S3 returns those objects as well.

---

# 8. Error Document

You can configure an error page:

```text
Error document: error.html
```

For example, if someone requests:

```text
/about
```

and S3 can't find the requested object, it can return the configured error document.

Example:

```text
index.html
     ↓
Website

Unknown page
     ↓
error.html
```

---

# 9. S3 Static Website Hosting vs Normal S3

These are slightly different concepts.

### Normal S3

```text
Application
    ↓
S3 API
    ↓
Bucket
    ↓
Object
```

Usually accessed through AWS APIs/SDKs.

### Static Website Hosting

```text
Browser
   ↓
HTTP request
   ↓
S3 Website Endpoint
   ↓
HTML/CSS/JS
```

S3 behaves as the static website host.

---

# 10. Important Limitation

S3 Static Website Hosting is for **static websites**.

It can serve:

```text
HTML
CSS
JavaScript
Images
Videos
Static files
```

But it does **not run backend code** such as:

```text
Python
FastAPI
Node.js backend
PHP
Java
```

For example:

```text
React frontend
      ↓
      S3 ✅
      
FastAPI backend
      ↓
      S3 ❌
```

You would host the backend separately, such as on EC2, ECS, Lambda/API Gateway, etc.

---

# 11. S3 + CloudFront

A common production architecture is:

```text
              ┌──────────────┐
              │     User     │
              └──────┬───────┘
                     ↓
                CloudFront
                     ↓
                S3 Bucket
                     ↓
              Static Website
```

CloudFront provides:

* HTTPS
* CDN/caching
* Better global performance
* Custom domain
* Additional security controls

For modern production deployments, **CloudFront + S3** is generally preferable to exposing the S3 website endpoint directly.

---

# 12. Example: React Website

Suppose you build:

```bash
npm run build
```

You get:

```text
dist/
├── index.html
├── assets/
│   ├── index-abc123.js
│   └── index-xyz456.css
└── logo.png
```

Upload the contents of `dist/` to S3:

```text
S3 Bucket
│
├── index.html
├── assets/
│   ├── index-abc123.js
│   └── index-xyz456.css
└── logo.png
```

Then configure:

```text
Static Website Hosting → Enabled
Index document → index.html
```

The S3 bucket can now serve the static frontend.

---

# ⭐ Important Notes

| Concept                   | Meaning                       |
| ------------------------- | ----------------------------- |
| S3 Static Website Hosting | Hosts static web content      |
| `index.html`              | Default homepage              |
| `error.html`              | Error page                    |
| Bucket Policy             | Can grant public object reads |
| `s3:GetObject`            | Allows reading objects        |
| Website Endpoint          | HTTP endpoint provided by S3  |
| CloudFront                | CDN placed in front of S3     |
| Backend                   | Cannot run directly on S3     |

### Mental Model

```text
Build website
     ↓
HTML / CSS / JS
     ↓
Upload to S3
     ↓
Enable Static Website Hosting
     ↓
Configure access
     ↓
Website Endpoint
     ↓
🌐 Website
```

**In one sentence:** S3 Static Website Hosting lets you upload static website files to an S3 bucket and serve them over HTTP, with `index.html` acting as the entry page.




<img width="651" height="323" alt="image" src="https://github.com/user-attachments/assets/5ad59e8e-2a89-446a-83f8-8c09c653c7fa" />


<img width="624" height="328" alt="image" src="https://github.com/user-attachments/assets/6e84f0ab-43bb-4ad1-8826-77f98e8cbb50" />

<img width="684" height="466" alt="image" src="https://github.com/user-attachments/assets/4b025974-60d1-4147-a10b-add0def9c66a" />


<img width="643" height="349" alt="image" src="https://github.com/user-attachments/assets/a90bcb26-6909-44df-b99f-b202a2c8e48a" />

<img width="308" height="164" alt="image" src="https://github.com/user-attachments/assets/5cf1dda2-8248-4a36-97a9-9348bd371e26" />
