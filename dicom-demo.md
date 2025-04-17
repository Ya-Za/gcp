Great! Here's a **step-by-step guide** to help you set up a **GCP + OHIF demo project** for viewing DICOM files, keeping it **secure and budget-friendly (≤ $100)**.

---

## 🌐 Step 1: Set Up Google Cloud Project

1. Go to [GCP Console](https://console.cloud.google.com/).
2. Create a **new project** (e.g., `dicom-demo`).
3. Enable **Billing**, but set a **budget alert**:
   - Go to **Billing > Budgets & alerts**
   - Set a $100 budget with email alerts at 50%, 75%, 90%, 100%

---

## 🧠 Step 2: Enable Required APIs

In your project, enable these APIs:
- **Cloud Healthcare API**
- **Cloud Storage**
- **IAM & Admin**
- **Compute Engine (optional, if hosting OHIF yourself)**

You can do this via UI or CLI:

```bash
gcloud services enable healthcare.googleapis.com storage.googleapis.com compute.googleapis.com
```

---

## 📦 Step 3: Upload DICOM Files to GCS (Google Cloud Storage)

1. Create a bucket:
   ```bash
   gsutil mb -p dicom-demo -l us-central1 gs://dicom-demo-bucket/
   ```

2. Upload your DICOM files:
   ```bash
   gsutil cp *.dcm gs://dicom-demo-bucket/
   ```

---

## 🏥 Step 4: Create a DICOM Store with DICOMweb

1. Create a Cloud Healthcare dataset:
   ```bash
   gcloud healthcare datasets create dicom_dataset --location=us-central1
   ```

2. Create a DICOM store with DICOMweb enabled:
   ```bash
   gcloud healthcare dicom-stores create dicom_store \
     --dataset=dicom_dataset \
     --location=us-central1 \
     --enable-dicom-web
   ```

3. Import DICOM files:
   ```bash
   gcloud healthcare dicom-stores import gcs dicom_store \
     --dataset=dicom_dataset \
     --location=us-central1 \
     --gcs-uri=gs://dicom-demo-bucket/*
   ```

---

## 👁️ Step 5: Deploy OHIF Viewer (Frontend)

You have 3 options:
1. **Quick demo**: Use the [OHIF Standalone Viewer](https://viewer.ohif.org/) (not customizable).
2. **Host on Firebase or GCS** (cheaper, static site).
3. **Host on VM** (more customizable, but may exceed budget).

### Option 2 (recommended for cost-saving):

1. Build OHIF locally:
   ```bash
   git clone https://github.com/OHIF/Viewers.git
   cd Viewers
   yarn install
   yarn run build
   ```

2. Deploy to Firebase:
   ```bash
   npm install -g firebase-tools
   firebase login
   firebase init
   firebase deploy
   ```

3. Configure OHIF to point to your DICOMWeb:
   In `.env` or config files, set:

   ```js
   window.config = {
     servers: [
       {
         name: 'GCP DICOMWeb',
         wadoUriRoot: '',
         qidoRoot: 'https://healthcare.googleapis.com/v1/projects/YOUR_PROJECT/locations/us-central1/datasets/dicom_dataset/dicomStores/dicom_store/dicomWeb',
         wadoRoot: 'same_as_above',
         supportsFuzzyMatching: true,
         imageRendering: 'wadors',
         thumbnailRendering: 'wadors',
         requestOptions: {
           requestFromBrowser: true,
         },
       },
     ],
   };
   ```

---

## 🔒 Step 6: Add Authentication

Use **Firebase Authentication**:

1. Enable Firebase Auth (Email or Google sign-in)
2. Wrap OHIF with Firebase auth:
   - Add login page
   - Protect routes
   - Use Firebase SDK to control access

3. Use **IAM** roles in GCP to ensure only specific users can access the DICOMWeb endpoints.

---

## 💰 Step 7: Monitor and Control Costs

- Set **budget alert**
- Use **Cloud Monitoring** to track DICOMWeb API usage
- Enable **Cloud Armor** or basic IP allowlist (to avoid abuse)

---

## ✅ Final Checklist

| Task                             | Done? |
|----------------------------------|-------|
| Project + Budget created         | ☐     |
| Healthcare API + GCS setup       | ☐     |
| DICOM files uploaded             | ☐     |
| DICOM Store + DICOMweb ready     | ☐     |
| OHIF Viewer deployed             | ☐     |
| OHIF configured to use DICOMWeb  | ☐     |
| Authentication added             | ☐     |
| Alerts and cost caps set         | ☐     |

---

Want a ready-made `firebase.json` and `dicomweb.json` for OHIF? Or want help deploying to GCP App Engine or Cloud Run instead?
---
Great setup! Since you're hosting **OHIF on Firebase** and getting a **401 error** when trying to access the **GCP Cloud Healthcare API**, the **cleanest and safest way forward** is to build a **proxy server that uses a service account** — **Option 1**.

Let’s go step by step to implement this solution using an Express-based proxy and deploy it to **Cloud Run**, which is easy, scalable, and Firebase-friendly:

---

### ✅ Step-by-Step Guide: Proxy with Service Account Token

---

#### **Step 1: Enable APIs**

Make sure these APIs are enabled in the GCP project where your DICOM store is:
```bash
gcloud services enable \
  healthcare.googleapis.com \
  run.googleapis.com \
  iamcredentials.googleapis.com
```
---

#### **Step 2: Create a Service Account for Proxy**

```bash
gcloud iam service-accounts create dicom-proxy-sa \
  --description="Service account for OHIF proxy" \
  --display-name="DICOM Proxy Service Account"
```

Give it permissions:

```bash
gcloud projects add-iam-policy-binding dicom-demo-456619 \
  --member="serviceAccount:dicom-proxy-sa@dicom-demo-456619.iam.gserviceaccount.com" \
  --role="roles/healthcare.dicomViewer"
```
---

#### **Step 3: Create and Download the Service Account Key**
```bash
gcloud iam service-accounts keys create key.json \
  --iam-account=dicom-proxy-sa@dicom-demo-456619.iam.gserviceaccount.com
```
---

#### **Step 4: Create the Proxy App**

Save this as `index.js`:
```js
const express = require('express');
const { GoogleAuth } = require('google-auth-library');
const fetch = require('node-fetch');

const app = express();
app.use(express.json());

const auth = new GoogleAuth({
  keyFile: 'key.json',
  scopes: ['https://www.googleapis.com/auth/cloud-platform'],
});

app.use('/dicomweb', async (req, res) => {
  const client = await auth.getClient();
  const token = await client.getAccessToken();

  const url = https://healthcare.googleapis.com/v1${req.originalUrl.replace('/dicomweb', '')};
  const headers = {
    ...req.headers,
    Authorization: Bearer ${token.token},
  };

  const response = await fetch(url, {
    method: req.method,
    headers,
    body: ['POST', 'PUT'].includes(req.method) ? JSON.stringify(req.body) : undefined,
  });

  const data = await response.text();
  res.status(response.status).send(data);
});

app.listen(8080, () => console.log('Proxy server is running on port 8080'));
```

Create a `package.json`:

```json
{
  "name": "dicom-proxy",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "google-auth-library": "^9.0.0",
    "node-fetch": "^2.6.9"
  }
}
```
---

#### **Step 5: Deploy to Cloud Run**

1. Create a `Dockerfile`:
```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```
2. Build and deploy:
```bash
gcloud builds submit --tag gcr.io/dicom-demo-456619/dicom-proxy
gcloud run deploy dicom-proxy \
  --image gcr.io/dicom-demo-456619/dicom-proxy \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1
```
Copy the **URL** of your Cloud Run service — you’ll use it in OHIF config.

---

#### **Step 6: Update OHIF Viewer Config**

Set the proxy URL as your DICOMWeb entrypoints:
```js
dicomWeb: [
  {
    name: 'DICOM Proxy',
    wadoRoot: 'https://<your-cloud-run-url>/dicomweb',
    qidoRoot: 'https://<your-cloud-run-url>/dicomweb',
    wadoUriRoot: 'https://<your-cloud-run-url>/dicomweb',
  },
],
```
---

Let me know if you'd like:
- A ready-to-clone GitHub template for this proxy
- Help deploying this via Firebase Functions instead of Cloud Run
- Setting up CORS headers

Happy to walk through any step in more detail!
