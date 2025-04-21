# 🪣 Google Cloud Storage – Quick Cheat Sheet

## 📦 What is a Bucket?
A **bucket** is a top-level container in Google Cloud Storage for storing files (called **objects**).

**Similar Services:**
- AWS: S3 Bucket
- Azure: Blob Storage Container

---

## 🖥️ View Buckets

### Web UI:
Go to [https://console.cloud.google.com/storage/browser](https://console.cloud.google.com/storage/browser)

### Cloud Shell:
```bash
gcloud storage buckets list
```

---

## 🛠️ Bucket Operations (CRUD)

### ✅ Create a Bucket
```bash
gcloud storage buckets create my-bucket-name --location=us-central1
```

### 📄 List Buckets
```bash
gcloud storage buckets list
```

### ✏️ Update Bucket (e.g., change default storage class)
```bash
gcloud storage buckets update my-bucket-name --default-storage-class=nearline
```

### ❌ Delete a Bucket
```bash
gcloud storage buckets delete my-bucket-name
```

---

## 🗃️ File (Object) Operations in Buckets

### ✅ Upload File
```bash
gcloud storage cp localfile.txt gs://my-bucket-name/
```

### 📄 List Files
```bash
gcloud storage ls gs://my-bucket-name/
```

### ⬇️ Download File
```bash
gcloud storage cp gs://my-bucket-name/remote.txt ./localcopy.txt
```

### ❌ Delete File
```bash
gcloud storage rm gs://my-bucket-name/remote.txt
```

---

## 📁 Folder Structure?

Google Cloud Storage is **flat**, but supports **virtual folders** by using `/` in object names.

Example:
- `folder1/image.jpg` is just a file named with a `/`, but shows up like a folder in UI or CLI.

---
