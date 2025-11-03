
````markdown
# 🧠 Data Version Control (DVC) with Google Cloud Storage

This repository demonstrates the integration of **Data Version Control (DVC)** with **Google Cloud Storage (GCS)** to enable efficient, reproducible, and collaborative data management for machine learning projects.

---

## 📌 Overview

Data versioning is a crucial aspect of any machine learning workflow.  
With DVC, you can track datasets and model files just like you track code with Git.  
The actual data is stored remotely (in Google Cloud Storage), while Git only tracks lightweight `.dvc` metadata files — keeping your repository fast and manageable.

This lab walks through setting up a DVC pipeline with GCS as a remote storage backend.

---

## ⚙️ Prerequisites

- A **Google Cloud account**
- A **GCS bucket** (e.g., `dvc_lab_1`)
- A **Service Account JSON key** with `Storage Object Admin` permission
- Python 3.8+ with `pip`

---

## 🏗️ Step 1: Create a Google Cloud Storage Bucket

1. Go to the [**Google Cloud Console**](https://console.cloud.google.com/).  
2. In the left menu, navigate to **Storage → Buckets → Create**.  
3. Enter a **unique name** (for example, `dvc_lab_1`).  
4. Set **Region:** `us-east1`.  
5. Keep other settings default and click **Create**.

---

## 🔐 Step 2: Create and Download Service Account Key

1. In Google Cloud Console, go to **IAM & Admin → Service Accounts**.  
2. Click **Create Service Account** → name it (e.g., `dvc-labs`).  
3. Click **Continue** and assign the role **Storage Object Admin**.  
4. Click **Done** to finish creation.  
5. On the Service Account list, click **Actions → Manage Keys → Add Key → Create new key**.  
6. Choose **JSON**, and a key file will be downloaded (e.g., `dvc-labs-477121-4e8d0f56cba4.json`).  
   Store this file securely — it authenticates DVC to access your bucket.

---

## 🧩 Step 3: Install DVC with GCS Support

```bash
pip install dvc[gs]
````

> 💡 Note: Use `[s3]`, `[azure]`, or `[gdrive]` instead if using a different cloud provider.
> To install all remotes: `pip install dvc[all]`

---

## 🚀 Step 4: Initialize Git and DVC

```bash
git init
dvc init
```

This creates the `.dvc/` folder and initializes DVC tracking in your repo.

---

## 🌩️ Step 5: Connect DVC to Google Cloud Storage

Add your bucket as the DVC remote:

```bash
dvc remote add -d myremote gs://dvc_lab_1
```

Connect your service account credentials (use the full local path to your key):

```bash
dvc remote modify --local myremote credentialpath "C:\Users\MargiShah\Downloads\dvc-labs-477121-4e8d0f56cba4.json"
```

Verify:

```bash
dvc config -l
dvc config --local -l
```

Expected:

```
core.remote=myremote
remote.myremote.url=gs://dvc_lab_1
remote.myremote.credentialpath=C:\Users\MargiShah\Downloads\dvc-labs-477121-4e8d0f56cba4.json
```

---

## 📦 Step 6: Track and Push Data

Place your dataset files inside the `data/` folder (for example: `CC_GENERAL.csv`, `data.txt`).

Then run:

```bash
dvc add data/CC_GENERAL.csv
dvc add data/data.txt
```

Commit DVC metadata to Git:

```bash
git add data/CC_GENERAL.csv.dvc data/data.txt.dvc data/.gitignore .dvc/config
git commit -m "Track datasets with DVC"
```

Push data to your remote bucket:

```bash
dvc push
```

Push code and metadata to GitHub:

```bash
git push origin main
```

---

## 🔁 Step 7: Handling Data Updates

Whenever your dataset changes:

```bash
# Replace or modify the dataset
dvc add data/CC_GENERAL.csv
git add data/CC_GENERAL.csv.dvc
git commit -m "Update dataset version"
dvc push
git push
```

DVC automatically detects content changes, computes a new hash, and uploads only the updated parts to GCS.

---

## ⏪ Step 8: Revert to a Previous Version

To roll back to a specific version:

```bash
git checkout <commit-hash>
dvc checkout
```

This restores the dataset state corresponding to that Git commit.

---

## 🧠 Step 9: Verify Everything

Check sync status:

```bash
dvc status -c
```

Expected output:

```
Data and pipelines are up to date.
```

View data in the GCP console:

* Go to [**Storage → Buckets → dvc_lab_1**](https://console.cloud.google.com/storage/browser/dvc_lab_1)
* You’ll see subfolders like `files/md5/...` — these are DVC-managed dataset versions.

---

## 🧾 Folder Structure

```
DVC_Labs/
│
├── data/
│   ├── .gitignore
│   ├── CC_GENERAL.csv.dvc
│   └── data.txt.dvc
│
├── .dvc/
│   ├── config
│   ├── .gitignore
│   └── cache/
│
├── .gitignore
├── .dvcignore
├── README.md
└── requirements.txt   # optional
```

---

## ✅ Summary

| Component                | Description                              |
| ------------------------ | ---------------------------------------- |
| **Git**                  | Tracks source code and DVC metadata      |
| **DVC**                  | Tracks large data and model files        |
| **Google Cloud Storage** | Stores actual dataset blobs              |
| **Service Account JSON** | Authenticates access between DVC and GCS |

---

## ✨ Verification Commands

```bash
dvc status -c           # Check sync with remote
dvc push                # Upload data to GCS
dvc pull                # Download from GCS
git log --oneline       # View version history
```