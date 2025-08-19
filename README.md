# Storage S3 Gateway

A full-stack application that lets users upload, list, download, and delete files in **Amazon S3** using a **React.js** frontend and a **Spring Boot 3** backend (REST API).

---

## Features

- Upload images/files to a designated S3 bucket
- List uploaded objects (name, size, last modified)
- Download / view files
- Delete files
- **Key naming** convention: `/images/{userId}/{imageId}`
- **Metadata** persisted in a relational DB (Amazon RDS) to associate `imageId` with a customer/user
- IAM-role based access in prod (no long‑lived keys on servers)
- CORS configured for local dev (React ↔ Spring)
- Local S3 storage for tests/demo

---

## Tech Stack

- **Frontend:** React
- **Backend:** Java 17+, Spring Boot 3, Spring Web, Spring Validation
- **Cloud:** AWS S3 (AWS SDK v2), Amazon RDS
- **Runtime/Deploy:** Docker (local), AWS Elastic Beanstalk on **EC2** (prod), IAM role `aws-elasticbeanstalk-ec2-role`
- **Build/Run:** Maven, Node.js

---

## Architecture

### High level
```
React Client ──HTTP(CORS)──▶ Spring Boot API ──S3Client──▶ Amazon S3 (bucket)
                                   │
                                   └──────────────▶ Amazon RDS (store image/customer mapping)
```

- **Client (React):** selects a file and calls the REST API with `multipart/form-data`.
- **API (Spring Boot on EC2 / Elastic Beanstalk):**
    - Validates input, determines an `imageId`, and stores a DB record mapping `{userId, imageId, key}` in **RDS**.
    - Uses **AWS SDK v2 (S3Client)** to `putObject` to the S3 bucket using the convention `/images/{userId}/{imageId}`.
    - In production, the EC2 instance obtains credentials from the **instance profile** (IAM role `aws-elasticbeanstalk-ec2-role`), so no static secrets are on the box.
- **S3 bucket:** durable object storage for the uploaded files.
- **RDS:** source of truth for file metadata and application‑level relationships (e.g., which user owns which image).

### Local testing/dev
```
IntelliJ / Docker ──▶ Spring Boot API ──▶ (a) S3 SDK to AWS S3  OR  (b) Fake/local path ~/.amigoscode/s3/images/{userId}/{imageId}
                                                │
                                                └────▶ Test doubles for S3 (integration tests)
```

### Request flow (upload)
1. React posts `multipart/form-data` to the API.
2. API authenticates/validates, generates `imageId`.
3. API writes `{userId, imageId, s3Key}` into **RDS**.
4. API uploads the bytes to **S3** at key: `/images/{userId}/{imageId}`.
5. API returns a success payload (and optionally a presigned URL or the `s3Key`).

---

## Credits

- Starter frontend and backend scaffolding are based on [**amigoscode**](https://github.com/amigoscode) materials.
- Extended with AWS S3 + RDS integration and IAM role-based deployment notes

