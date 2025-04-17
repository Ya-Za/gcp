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
Create a `package.json`:
json
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
