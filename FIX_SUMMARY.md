# Upload Issue Fix - May 29, 2026

## Problem
Users could log in successfully, but image uploads failed with "Network error occurred". Metadata was being stored in DynamoDB, but no files appeared in the S3 bucket.

## Root Cause
The Lambda functions were generating S3 presigned URLs using the **global S3 endpoint** (`s3.amazonaws.com`) instead of the **regional endpoint** (`s3.ap-south-1.amazonaws.com`).

When the browser attempted to upload via the global endpoint, S3 returned a **307 Temporary Redirect** to the regional endpoint. However, the browser's `fetch()` API does not automatically follow redirects for PUT requests with a body, causing the upload to fail silently.

## Evidence
- CloudWatch logs showed successful Lambda executions and presigned URL generation
- DynamoDB contained metadata entries with `status: "pending"`
- S3 bucket was empty (0 objects)
- Test upload via curl showed HTTP 307 redirect response

## Solution
Updated both Lambda functions (`uploadPhoto.py` and `listPhotos.py`) to explicitly configure the S3 client with:

1. **Region specification**: `region_name='ap-south-1'`
2. **Regional endpoint URL**: `endpoint_url='https://s3.ap-south-1.amazonaws.com'`
3. **Signature version**: `signature_version='s3v4'`
4. **Addressing style**: `s3={'addressing_style': 'virtual'}`

### Code Changes

**Before:**
```python
s3_client = boto3.client('s3')
```

**After:**
```python
s3_client = boto3.client(
    's3',
    region_name='ap-south-1',
    config=boto3.session.Config(
        signature_version='s3v4',
        s3={'addressing_style': 'virtual'}
    ),
    endpoint_url='https://s3.ap-south-1.amazonaws.com'
)
```

## Verification
- Test upload via presigned URL: **HTTP 200 Success**
- File successfully stored in S3 bucket
- Regional endpoint confirmed in generated URLs

## Files Modified
- `lambda/uploadPhoto.py`
- `lambda/listPhotos.py`

## Deployment
Both Lambda functions were updated and deployed to AWS Lambda in ap-south-1 region.

## Next Steps
1. Test the application in the browser
2. Clean up pending DynamoDB entries if needed
3. Monitor CloudWatch logs for any issues

## Technical Notes
- This issue is common when using boto3 without explicit region configuration
- The global endpoint works for some operations but causes redirect issues for uploads
- Regional endpoints provide better performance and avoid redirect issues
- CORS is properly configured on the S3 bucket, so no CORS changes were needed
