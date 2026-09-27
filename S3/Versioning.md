# Amazon S3 Versioning — Notes

## 1. What is S3 Versioning?

**S3 Versioning** keeps multiple versions of the same object in a bucket.

Normally, when you upload a file with the same name:

```text
index.html
   ↓
Upload new index.html
   ↓
Old file overwritten
```

With versioning enabled:

```text
index.html
   │
   ├── Version 1
   └── Version 2
```

The old version is retained instead of being permanently overwritten.

---

# 2. Enable Bucket Versioning

Go to:

```text
S3
 → Bucket
 → Properties
 → Bucket Versioning
 → Edit
 → Enable
```

Once enabled, newly uploaded/modified objects receive **Version IDs**.

---

# 3. Example — Updating a Website

Initially:

```text
index.html

I love coffee
```

Upload a new version:

```text
index.html

I REALLY love coffee
```

With versioning enabled, S3 stores both:

```text
index.html
│
├── Version ID: null
│   └── I love coffee
│
└── Version ID: abc123...
    └── I REALLY love coffee
```

The latest version becomes the current version served by S3.

---

# 4. What Does `null` Version ID Mean?

If an object was uploaded **before versioning was enabled**, it can have:

```text
Version ID = null
```

For example:

```text
coffee.jpg
Version ID → null
```

After enabling versioning and uploading a new version:

```text
coffee.jpg
│
├── Version ID: null       ← uploaded before versioning
└── Version ID: abc123...  ← uploaded after versioning
```

### Important

> Objects uploaded before versioning was enabled don't suddenly receive a normal version ID. They can remain associated with the `null` version.

---

# 5. Show Object Versions

In the S3 console, enable:

```text
Show versions
```

You can then see the different versions of objects.

Example:

```text
index.html
│
├── Version ID: abc123
└── Version ID: xyz789
```

Without **Show versions**, you normally see only the current version.

---

# 6. Rolling Back to an Older Version

Suppose the current website says:

```text
I REALLY love coffee
```

but you want:

```text
I love coffee
```

With versioning, you can restore the older version.

You can identify the previous version by enabling:

```text
Show versions
```

Then you can remove the unwanted current version or otherwise restore the desired version.

---

# 7. Permanent Delete of a Version

When **Show versions** is enabled and you delete a specific version, you're deleting the actual version.

This is a:

> **Permanent delete**

Example:

```text
index.html
│
├── Version 1
└── Version 2  ← permanently delete
```

After permanently deleting Version 2:

```text
index.html
│
└── Version 1
```

### Important

A specific version permanently deleted from S3 cannot be recovered through S3 versioning.

---

# 8. Normal Delete vs Permanent Delete

This is one of the **most important concepts**.

### Without selecting a specific version

If you delete:

```text
coffee.jpg
```

S3 doesn't immediately delete the underlying object when versioning is enabled.

Instead, S3 creates a:

## Delete Marker

```text
coffee.jpg
│
├── Delete Marker ← current
└── Previous version ← still exists
```

---

# 9. What is a Delete Marker?

A **Delete Marker** is a special version that tells S3:

> "Treat this object as deleted."

The previous object is still stored.

Example:

```text
coffee.jpg

Version 1
   ↓
Actual image

Delete Marker
   ↓
Current version
```

Because the Delete Marker is now the current version, a normal request for:

```text
coffee.jpg
```

returns an object-not-found response.

But the original version still exists underneath.

---

# 10. Delete Marker Example

Before deletion:

```text
coffee.jpg
   ↓
Image exists
```

You click:

```text
Delete
```

S3 creates:

```text
coffee.jpg
│
├── Delete Marker ← current
└── Version 1     ← actual image still exists
```

From the normal bucket view:

```text
coffee.jpg
```

appears to be gone.

But with:

```text
Show versions
```

you can see:

```text
Delete Marker
Version 1
```

---

# 11. How to Restore a Deleted Object

Because the original version wasn't actually deleted, you can restore it.

### Step 1

Enable:

```text
Show versions
```

### Step 2

Find the **Delete Marker**.

### Step 3

Delete the Delete Marker **permanently**.

Result:

```text
Before:

coffee.jpg
│
├── Delete Marker ← current
└── Version 1

After deleting Delete Marker:

coffee.jpg
│
└── Version 1 ← current
```

The previous version becomes the current version again.

---

# 12. Versioning Flow

```text
             S3 Versioning
                  │
                  ↓
           Upload object
                  │
                  ↓
            Version 1
                  │
          Upload same file
                  │
                  ↓
            Version 2
                  │
          Delete object
                  │
                  ↓
          Delete Marker
                  │
       Delete Delete Marker
                  │
                  ↓
         Version 2 becomes
            current again
```

---

# 13. Versioning vs Delete

| Operation               | What happens?                            |
| ----------------------- | ---------------------------------------- |
| Upload same object      | Creates a new version                    |
| Update object           | Creates a new version                    |
| Normal delete           | Creates a Delete Marker                  |
| Delete specific version | Permanently deletes that version         |
| Delete Delete Marker    | Restores the previous version            |
| `Show versions`         | Displays all versions and Delete Markers |

---

# 14. Why Use S3 Versioning?

### Accidental overwrite protection

```text
Old file
   ↓
New file uploaded
   ↓
Old version still available
```

### Accidental deletion protection

```text
Delete object
     ↓
Delete Marker
     ↓
Previous version still exists
```

### Website rollback

```text
Version 1 → Working website
Version 2 → New website
Version 3 → Buggy website

        ↓

Restore previous version
```

### Data recovery

Versioning helps recover from accidental modifications or deletions.

---

# 15. Important Limitation

Versioning does **not** mean that deleted versions are automatically gone forever.

You can accumulate many versions:

```text
file.csv
├── Version 1
├── Version 2
├── Version 3
├── Version 4
├── Version 5
└── ...
```

All those versions can consume storage.

Therefore, in real-world systems, **S3 Lifecycle Rules** are often used to manage old versions and control storage costs.

---

# ⭐ Exam / Interview Points

1. **S3 Versioning keeps multiple versions of an object.**
2. Enable it from **Bucket → Properties → Bucket Versioning**.
3. Uploading the same object again creates a **new version**.
4. Objects uploaded before versioning was enabled can have **Version ID = `null`**.
5. **Show versions** lets you see all versions.
6. Normal deletion creates a **Delete Marker**.
7. A Delete Marker does **not** delete the underlying previous version.
8. Deleting a specific version is a **permanent delete**.
9. Deleting the Delete Marker can **restore the previous version**.
10. Versioning is useful for **rollback and recovery**.
11. Multiple versions consume storage, so **Lifecycle Rules** can help manage old versions.

### 🧠 One-line mental model

> **Upload = new version, normal delete = Delete Marker, permanent version delete = actually remove that version.**



<img width="1412" height="684" alt="image" src="https://github.com/user-attachments/assets/30514c9c-4a5d-4f7b-8b54-3ef2bd4a1547" />


<img width="1124" height="538" alt="image" src="https://github.com/user-attachments/assets/3d41e86c-5a7e-4cad-91c7-b97d683c5ac3" />

<img width="1218" height="328" alt="image" src="https://github.com/user-attachments/assets/4a983d14-f6fa-4be3-8b41-6b3f6efd1e51" />

<img width="1028" height="368" alt="image" src="https://github.com/user-attachments/assets/a90d2eb7-5add-4823-b3d2-fbfb2b91fa3a" />

<img width="734" height="260" alt="image" src="https://github.com/user-attachments/assets/0a016089-c98e-4f67-ae10-9451f33104a4" />


<img width="1106" height="258" alt="image" src="https://github.com/user-attachments/assets/bd96b0fc-716d-4fdb-bc7d-ac2f95d7fd1e" />

<img width="1132" height="424" alt="image" src="https://github.com/user-attachments/assets/d191d876-fd24-4dce-98c3-83dac0c51260" />

<img width="1144" height="314" alt="image" src="https://github.com/user-attachments/assets/7232992e-9b86-441e-8ab2-e249677cc9d1" />



## Disable Verisooning

1. Create NEw Bucket
<img width="1408" height="436" alt="image" src="https://github.com/user-attachments/assets/cf56980b-b07b-40cc-9157-f7e696c52a1c" />


<img width="1216" height="562" alt="image" src="https://github.com/user-attachments/assets/8a5c2e2f-76aa-4212-929c-c3b8f7240bcf" />

<img width="1436" height="362" alt="image" src="https://github.com/user-attachments/assets/c16b6e87-d322-435f-8556-763c42f7427e" />

<img width="1138" height="274" alt="image" src="https://github.com/user-attachments/assets/59c838bb-5aac-48aa-9c22-3b96023eedea" />


<img width="1144" height="214" alt="image" src="https://github.com/user-attachments/assets/5e6e6521-fbe6-4bfe-a1ba-ce4dab056a71" />

