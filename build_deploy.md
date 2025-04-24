# 🚀 Google Cloud: Build & Deploy with gcloud

## 🏗️ Build Docker Image and Push to GCR

```bash
# Build and tag the image, push to Google Container Registry
gcloud builds submit --tag gcr.io/<PROJECT_ID>/<IMAGE_NAME>
```

Example:
```bash
gcloud builds submit --tag gcr.io/dicom-demo-456619/dicom-proxy
```

---

## ☁️ Deploy Image to Cloud Run

```bash
gcloud run deploy <SERVICE_NAME> \
  --image gcr.io/<PROJECT_ID>/<IMAGE_NAME> \
  --platform managed \
  --allow-unauthenticated \
  --region <REGION>
```

Example:
```bash
gcloud run deploy dicom-proxy \
  --image gcr.io/dicom-demo-456619/dicom-proxy \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1
```

---

## 📦 Common `gcloud builds` Commands

```bash
# View build history
gcloud builds list

# Cancel a running build
gcloud builds cancel <BUILD_ID>

# View build logs
gcloud builds log <BUILD_ID>
```

---

## 🔧 Common `gcloud run` Commands

```bash
# List deployed services
gcloud run services list

# View service details
gcloud run services describe <SERVICE_NAME> --region <REGION>

# List revisions
gcloud run revisions list --service <SERVICE_NAME> --region <REGION>

# Delete a service
gcloud run services delete <SERVICE_NAME> --region <REGION>
```

---

## 🔍 List Container Images in GCR

```bash
gcloud container images list
gcloud container images list-tags gcr.io/<PROJECT_ID>/<IMAGE_NAME>
```
