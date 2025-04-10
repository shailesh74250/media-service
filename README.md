# media-service
File Storage &amp; Media Management

✅ Purpose: Handles file uploads, storage, and retrieval.
✅ Tech Stack: AWS S3, Cloudinary, Google Cloud Storage, MinIO.
✅ Endpoints:

/upload

/get/{fileId}

/delete/{fileId}

## Video schema could be 
- video table properties
- id: UUID
- title: string
- description: string
- video_url: stirng
- thumbnail_url: string
- likes: BIGINT
- dislikes: BIGINT
- comments: BIGINT
- user_id: FK
- duration
- visibility: ENUM (public, private, unlisted)
- tags
- views
- cratedAt
- updateAt

## Video comments
- id: UUID
- video_id: FK
- user_id: FK
- comment: string
- parent_id for replies (self-reference FK)
- createdAt: timestamp
- updatedAt: timestamp

## Likes & Dislikes
- id: UUID
- video_id: FK
- user_id: FK
- is_like: boolean
- createdAt: timestamp
- updatedAt: timestamp

## Subscribe
- id
- channel_id FK
- user_id FK
- is_subscribed: boolean
- createdAt timestamp
- updateAt timestamp

## Channel Table
- id
- user_id FK
- video_id FK
- playlist: FK
- createdAt timestamp
- updateAt timestamp

## Video views table
- id 
- user_id FK
- video_id FK
- id_address:   varchar
- viewed_at timestamp
- createdAt
- updatedAT


## Playlists
- id: UUID
- user_id FK
- name varchar
- is_public boolean
- createdAt: timestamp

## PlaylistVideos
- id UUID
- playlist_id UUId
- video_id UUID
- order_index (order in the playlist)

