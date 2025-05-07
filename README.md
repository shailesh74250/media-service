# media-service
File Storage &amp; Media Management

✅ Purpose: Handles file uploads, storage, and retrieval.
✅ Tech Stack: AWS S3, Cloudinary, Google Cloud Storage, MinIO.
✅ Endpoints:

/upload

/get/{fileId}

/delete/{fileId}

# Scalable media uploading service 
- BE offload all the uploading processing on client itself by pre-signed URL
- FE request for pre-signed URL for particalar media uploading with media name
- BE will generate presigned url specific to media 
- FE upload media to S3 using presign url
- Once S3 media uploading done 
- S3 Trigger event and produce event to SNS topic, or SNS topic will light uploading event of S3
- Multiple services queues will subscribes SNS topic and receive media metadata 
- Media backend service worker threads subscribe to SNS using SQS or rabbitmq and store metadaba on database for further use. This will be asynchronousely task, main thread will not get block.
- Transcode service subscribe to SNS for takig media for transcoding in different band widths
- Backend service will not handle any uploading work.


## Get video CDN for streaming in client side
- in a presigned URL upload flow, the upload is handled directly by the client to S3, so your backend never sees the file. But you still need to:
- Generate a unique S3 object key (path)
- Presign a PUT request for that key
- Return both:
  - the presigned URL for uploading
  - and the public or CDN-streamable URL for accessing/streaming it later
    
        const objectKey = `videos/${tenantId}/${uuidv4()}.mp4`;
        const uploadUrl = await getSignedUrl(
          s3,
          new PutObjectCommand({
            Bucket: 'my-app-media',
            Key: objectKey,
            ContentType: 'video/mp4',
          }),
          { expiresIn: 3600 }
        );

        const videoUrl = `https://${CDN_DOMAIN}/${objectKey}`; // CloudFront or S3 public path

        return { uploadUrl, videoUrl };
    
    - uploadUrl: used by the client to upload directly to S3
    - videoUrl: permanent access URL, saved to DB and shown to users
    - Client Uses uploadUrl to PUT the file to S3
    - Your App/DB Stores videoUrl
    - DB Metadata

            {
              "tenantId": "tenant_abc",
              "videoUrl": "https://cdn.myapp.com/videos/tenant_abc/uuid.mp4",
              "title": "Demo Video",
              "duration": 102,
              "type": "mp4"
            }
- S3 Event Notification + SQS or Lambda
- Enable S3 Event Notifications for PutObject events.
- Configure it to send events to:
  - an SQS queue, or
  - an SNS topic, or
  - a Lambda function.
  - As soon as a client uploads to the presigned URL and S3 confirms the upload, S3 sends an event like:

            {
              "Records": [
                {
                  "eventName": "ObjectCreated:Put",
                  "s3": {
                    "bucket": { "name": "my-media-bucket" },
                    "object": { "key": "videos/tenant123/uuid.mp4", "size": 123456 }
                  }
                }
              ]
            }

- Your backend consumes that SQS event, looks up the object key, and updates the database to mark the video as "uploaded".
- You can also validate the tenant from the key structure: videos/{tenantId}/{uuid}.mp4
