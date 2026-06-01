# Direct-to-S3 Video Upload Architecture

## The Problem
When uploading large media files (especially videos), cloud hosting providers and frameworks enforce strict request size limits:
1. **Vercel Limits:** Vercel Serverless Functions (which Next.js API proxies run on) have a hardcoded, unchangeable **4.5 MB** limit for incoming request payloads.
2. **PHP Limits:** Standard PHP environments have `upload_max_filesize` defaults of **2 MB** and `post_max_size` of **8 MB**.

If an admin attempts to upload a 50MB video through the Next.js Dashboard to the Symfony backend, the upload is instantly killed by Vercel before it even reaches the server.

## The Solution: Direct-to-S3 Presigned URLs
To completely bypass all server bottlenecks and size limits, we implemented a "Presigned URL" architecture. This means the actual file binary NEVER touches Vercel or Symfony; it flows directly from the user's browser to the cloud storage bucket (Backblaze B2).

### The 3-Step Workflow

#### 1. The Ticket Request
When the user drops a video file into the dashboard, the Next.js app sends a tiny, lightweight `GET` request to Symfony (`/api/media/presigned-url`).
Symfony uses its AWS S3 credentials to ask Backblaze B2 for a temporary, secure upload URL that is valid for 15 minutes. It generates a unique, safe filename and returns the URL and filename to Next.js.

#### 2. The Direct Upload
The Next.js app uses the native browser `fetch()` API with the `PUT` method to send the raw video file **directly** to the Backblaze B2 URL.
Because this goes straight from the browser to the storage bucket, it bypasses Vercel and PHP completely. You can upload gigabytes of data at maximum network speed.

#### 3. The Confirmation
Once the browser confirms the `PUT` request was successful, Next.js sends a lightweight `POST` request to Symfony (`/api/media/confirm-upload`) with the generated filename and folder ID.
Symfony creates the `Media` entity in the database, associates it with the correct Workspace and Folder, and saves it. The dashboard then reloads the media grid.

## Important Note: CORS Configuration
For Direct-to-S3 uploads to work, your **Backblaze B2** bucket **must** have CORS (Cross-Origin Resource Sharing) configured to allow `PUT` requests from your Next.js domain (e.g., `https://plaisoram-web.vercel.app` and `http://localhost:3000`). 

If CORS is not configured, the browser will block the direct upload in Step 2 with an `OPTIONS 403 Forbidden` error, and the user will see a network error. 

### How to Configure CORS on Backblaze B2:
You can configure CORS rules in Backblaze B2 by adding a custom CORS JSON rule in the B2 Web Console:
1. Log into your **Backblaze Account**.
2. Click on **Buckets** in the left menu.
3. Find your bucket (`B2_BUCKET_NAME`) and click on **Bucket Settings**.
4. Scroll down to the **CORS Rules** section and choose **Custom JSON**.
5. Paste the following CORS configuration:
   ```json
   [
     {
       "corsRuleName": "Allow-Plaisoram-Uploads",
       "allowedOrigins": [
         "https://plaisoram-web.vercel.app",
         "http://localhost:3000"
       ],
       "allowedMethods": [
         "s3_get",
         "s3_put",
         "s3_post",
         "s3_delete",
         "s3_head"
       ],
       "allowedHeaders": ["*"],
       "exposeHeaders": ["ETag"],
       "maxAgeSeconds": 3600
     }
   ]
   ```
6. Click **Save Changes**.
