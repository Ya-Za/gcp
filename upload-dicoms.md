Great! Your project number is `669153411426`. Now run the following command to grant the required permission:

---

### ✅ **Grant Healthcare API access to your GCS bucket**

```bash
gsutil iam ch serviceAccount:service-669153411426@gcp-sa-healthcare.iam.gserviceaccount.com:objectViewer gs://dicom-demo-bucket-456619
```

---

### ✅ **Then retry the import**

```bash
gcloud healthcare dicom-stores import gcs dicom_store \
  --dataset=dicom_dataset \
  --location=us-central1 \
  --gcs-uri=gs://dicom-demo-bucket-456619/image-*.dcm \
  --project=dicom-demo-456619
```

Let me know if it works or if you get any more errors.
