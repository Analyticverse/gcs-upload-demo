## Demo: Using REST API for Uploading a EHR to Google Cloud Storage (GCS)

### Goal

This tutorial is intended to guide participating **Integrated Health Care Sites** in securely uploading **EHR data** to the designated Google Cloud Storage (GCS) buckets at the **Connect Coordinating Center**. Two methods are provided to accommodate different security policies regarding CLI usage. 

---

### Prerequisites

- You must have a GCS bucket to upload your file.
- Your Google Cloud project must have the necessary APIs enabled (Google Cloud Storage API).
- You should have appropriate permissions to upload files to the GCS bucket.

---

### Method 1: Uploading CSV to GCS Using `gcloud auth print-access-token`

#### Step 1: Obtain Access Token using `gcloud`

Run the following command to get an access token using `gcloud`:

```bash
gcloud auth print-access-token
```

This will print an OAuth 2.0 access token.

#### Step 2: Upload CSV to GCS

Replace the placeholders in the following `curl` command with your details:

- `OBJECT_LOCATION`: Local path to your CSV file.
- `OBJECT_CONTENT_TYPE`: The MIME type of the file (e.g., `text/csv`).
- `BUCKET_NAME`: The name of your GCS bucket.
- `OBJECT_NAME`: The desired name of the object in the GCS bucket.

```bash
curl -X POST --data-binary @OBJECT_LOCATION \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: OBJECT_CONTENT_TYPE" \
  "https://storage.googleapis.com/upload/storage/v1/b/BUCKET_NAME/o?uploadType=media&name=OBJECT_NAME"
```

This will upload the specified CSV file to the GCS bucket.

---

### Method 2: Uploading CSV to GCS Without Using the `gcloud` CLI

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

#### Step 4: Upload CSV to GCS

Now that you have the access token, replace the placeholders in the following `curl` command:

- `YOUR_CSV_FILE_PATH`: Path to the CSV file on your local machine.
- `YOUR_ACCESS_TOKEN`: The access token obtained in Step 3.
- `YOUR_BUCKET_NAME`: The name of the GCS bucket.
- `YOUR_OBJECT_NAME`: The name to use for the uploaded file in GCS.

```bash
curl -X POST --data-binary @YOUR_CSV_FILE_PATH \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: text/csv" \
  "https://storage.googleapis.com/upload/storage/v1/b/YOUR_BUCKET_NAME/o?uploadType=media&name=YOUR_OBJECT_NAME"
```

This command will upload the CSV file to the specified bucket.

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

This SOP provides two methods for uploading a CSV file to Google Cloud Storage (GCS). The first method leverages the `gcloud` CLI to obtain an access token, while the second method uses OAuth 2.0 to manually retrieve an access token without relying on the CLI.

