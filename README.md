# 📸 Serverless Photo Gallery — AWS

A fully serverless, multi-user photo gallery application built on AWS. Users sign in via Cognito, upload photos to S3, and view only their own images — all without managing any servers.

**Developed by Krishna Yada**

---

## 🏗️ Architecture

```
Browser → Cognito (auth) → API Gateway → Lambda → S3 + DynamoDB
```

| Service | Resource | Purpose |
|---------|----------|---------|
| Amazon Cognito | `ap-south-1_l8Q8UF6XP` | User sign-up / sign-in (JWT) |
| Amazon S3 | `krishna01062026data` | Photo file storage (presigned URLs) |
| Amazon S3 | `krishna01062026hosting` | Static frontend hosting |
| AWS Lambda | `uploadPhoto` | Save metadata + return presigned PUT URL |
| AWS Lambda | `listPhotos` | Return user's photos + presigned GET URLs |
| API Gateway | `PhotoGalleryAPI` | REST endpoints (`/photos` GET & POST) |
| DynamoDB | `PhotoGallery` | Photo metadata (photoId, s3Key, userSub…) |
| IAM Role | `serverless-capstone-01062026` | Lambda execution permissions |

**Region:** `ap-south-1` (Mumbai)  
**API URL:** `https://ypq5mzgr00.execute-api.ap-south-1.amazonaws.com/prd`

---

## 🚀 Quick Start

### Prerequisites
- AWS account with CLI configured (`aws configure`, region `ap-south-1`)
- Python 3.12 (for Lambda code)

### Deploy in order

```bash
# 1. DynamoDB table
aws dynamodb create-table \
  --table-name PhotoGallery \
  --attribute-definitions AttributeName=photoId,AttributeType=S \
  --key-schema AttributeName=photoId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1

# 2. S3 buckets
aws s3api create-bucket --bucket krishna01062026data \
  --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1

aws s3api create-bucket --bucket krishna01062026hosting \
  --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1

# 3. Upload frontend
aws s3 cp s3/index.html s3://krishna01062026hosting/index.html --region ap-south-1
```

See [`PhotoGallery_Deployment_Guide.html`](./PhotoGallery_Deployment_Guide.html) for the complete step-by-step guide.

---

## 📁 Project Structure

```
├── lambda/
│   ├── uploadPhoto.py       # Handles photo upload + presigned PUT URL
│   └── listPhotos.py        # Lists user's photos + presigned GET URLs
├── s3/
│   └── index.html           # Frontend (configure API URL + Cognito values)
├── PhotoGallery_Deployment_Guide.html   # Full deployment guide
├── PhotoGallery_Deployment_Guide.pdf    # PDF version of the guide
└── README.md
```

---

## ⚙️ Frontend Configuration

Edit the config block in `s3/index.html` before uploading:

```js
const COGNITO_CONFIG = {
    domain:   'krishna.auth.ap-south-1.amazoncognito.com',
    clientId: '3k798826doeh2bnbld9qb4p5b6',
    redirectUri: window.location.href.split('#')[0],
    region: 'ap-south-1'
};

const CONFIG = {
    API_URL:        'https://ypq5mzgr00.execute-api.ap-south-1.amazonaws.com/prd',
    S3_BUCKET_NAME: 'krishna01062026data',
    AWS_REGION:     'ap-south-1'
};
```

---

## 🔐 Security

- All S3 photo access is via **short-lived presigned URLs** (PUT: 5 min, GET: 15 min)
- API Gateway enforces **Cognito JWT authorization** on every request
- Per-user isolation — `listPhotos` filters by `userSub` so users only see their own photos
- IAM role follows **least-privilege** principle (Lambda only, scoped to required services)

---

## 💰 Estimated Cost

~**$1.80 / month** for light usage (1,000 photos, 10,000 views). Cognito is free under 50K MAU.

---

## 📄 License

MIT — see [LICENSE](./LICENSE)

---

*Created: June 1, 2026 · Region: ap-south-1*
