## Demo: Uploading a CSV or JSONL File to Google Cloud Storage (GCS)

### Goal

This tutorial is intended to guide participating **Integrated Health Care Sites** in securely uploading **EHR data** to the designated Google Cloud Storage (GCS) buckets at the **Connect Coordinating Center**. This document outlines how to upload files using the REST API, with considerations for different file types (e.g., CSV, JSON, JSONL) and the use of absolute vs. relative file paths. Two methods are provided to accommodate different security policies regarding CLI usage.

---

### Prerequisites

- You must have a GCS bucket to upload your file.
- You should have appropriate permissions to upload files to the GCS bucket.

---

### File Types

Depending on the file type you are uploading, the `Content-Type` header must be set correctly:

- **CSV files**: Use `text/csv`.
- **JSONL (Newline-delimited JSON) files**: Use `application/json` or `application/x-ndjson`.

---

### Method 1: Uploading Files to GCS Using `gcloud auth print-access-token`

#### Step 1: Obtain Access Token using `gcloud`

Run the following command to get an access token using `gcloud`:

```bash
gcloud auth print-access-token
```

This will print an OAuth 2.0 access token.

#### Step 2: Upload File to GCS

Replace the placeholders in the following `curl` command with your details:

- **File Path**: Use the relative or full path to the file, e.g., `@/Users/username/Documents/test.csv` for a full path or `@test.csv` if it's in your current directory.
- **Content-Type**: Depending on your file type:
  - For CSV: `text/csv`
  - For JSONL: `application/x-ndjson` or `application/json`
- **Bucket Name**: The name of your GCS bucket.
- **Object Name**: The desired name of the file in the GCS bucket.

##### Example for CSV File:
```bash
curl -X POST --data-binary @/Users/username/Documents/test.csv \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: text/csv" \
  "https://storage.googleapis.com/upload/storage/v1/b/ehr_sanford/o?uploadType=media&name=test.csv"
```

##### Example for JSONL File:
```bash
curl -X POST --data-binary @/Users/username/Documents/data.jsonl \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/x-ndjson" \
  "https://storage.googleapis.com/upload/storage/v1/b/ehr_sanford/o?uploadType=media&name=data.jsonl"
```

If your file is in the current directory, you can use a **relative path** like this:

```bash
curl -X POST --data-binary @test.csv \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: text/csv" \
  "https://storage.googleapis.com/upload/storage/v1/b/ehr_sanford/o?uploadType=media&name=test.csv"
```

---

### Method 2: Uploading Files to GCS Without Using the `gcloud` CLI

#### Step 1: Set up OAuth Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project (or use an existing one).
3. Navigate to **APIs & Services > OAuth consent screen** and configure it.
4. Go to **Credentials** and create **OAuth 2.0 Client IDs**. 
   - Set your **Authorized redirect URIs** (use `http://localhost` for testing).
5. Note down the **Client ID** and **Client Secret**.

#### Step 2: Get Authorization Code

Open the following URL in your browser, replacing `YOUR_CLIENT_ID` and `YOUR_REDIRECT_URI` with the appropriate values:

```
https://accounts.google.com/o/oauth2/auth?client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI&response_type=code&scope=https://www.googleapis.com/auth/devstorage.read_write
```

This will prompt you to authorize access to your Google Cloud project. Once approved, Google will redirect you to the specified redirect URI with an authorization code in the URL.

Example:
```
http://localhost/?code=AUTHORIZATION_CODE
```

Copy the `AUTHORIZATION_CODE` from the URL.

#### Step 3: Exchange Authorization Code for Access Token

Run the following `curl` command to exchange the authorization code for an access token:

```bash
curl -X POST \
  https://oauth2.googleapis.com/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "code=AUTHORIZATION_CODE" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=YOUR_REDIRECT_URI" \
  -d "grant_type=authorization_code"
```

The response will include an `access_token`, which can be used for authentication.

#### Step 4: Upload File to GCS

Now that you have the access token, replace the placeholders in the following `curl` command with the appropriate file path, content type, and bucket/object names.

##### Example for CSV File:
```bash
curl -X POST --data-binary @/Users/username/Documents/test.csv \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: text/csv" \
  "https://storage.googleapis.com/upload/storage/v1/b/YOUR_BUCKET_NAME/o?uploadType=media&name=test.csv"
```

##### Example for JSONL File:
```bash
curl -X POST --data-binary @/Users/username/Documents/data.jsonl \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-ndjson" \
  "https://storage.googleapis.com/upload/storage/v1/b/YOUR_BUCKET_NAME/o?uploadType=media&name=data.jsonl"
```

---

### Optional: Refresh Token for Reuse

If you received a `refresh_token` in Step 3, you can use it to get a new access token when the current one expires. Use this command to refresh the token:

```bash
curl -X POST \
  https://oauth2.googleapis.com/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "refresh_token=YOUR_REFRESH_TOKEN" \
  -d "grant_type=refresh_token"
```

This will return a new access token without requiring you to go through the authorization process again.

---

### Conclusion

This SOP provides two methods for uploading a CSV or JSONL file to Google Cloud Storage (GCS). It includes details on how to specify file types and the appropriate `Content-Type` headers, as well as instructions for using both absolute and relative paths.
