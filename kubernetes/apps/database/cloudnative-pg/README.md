# CloudNativePG

## S3 Configuration

1. Create `~/.mcli/config.json`.

    ```json
    {
        "version": "10",
        "aliases": {
            "minio": {
                "url": "https://s3.<secret domain>",
                "accessKey": "<access key>",
                "secretKey": "<secret key>",
                "api": "S3v4",
                "path": "auto"
            }
        }
    }
    ```

2. Create the cloudnative-pg bucket user and password.

    ```sh
    export BUCKET_PASSWORD="$(openssl rand -hex 20)"
    mcli admin user add minio cloudnative-pg "${BUCKET_PASSWORD}"
    ```

3. Create the CloudNativePG bucket.

    ```sh
    mcli mb minio/cloudnative-pg
    ```

4. Create `~/.mcli/cloudnative-pg-user-policy.json`

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:ListBucket",
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Effect": "Allow",
                "Resource": ["arn:aws:s3:::cloudnative-pg/*", "arn:aws:s3:::cloudnative-pg"],
                "Sid": ""
            }
        ]
    }
    ```

5. Apply the bucket policies.

    ```sh
    mcli admin policy create minio cloudnative-pg-private ~/.mcli/cloudnative-pg-user-policy.json
    ```

6. Associate private policy with the user.

    ```sh
    mcli admin policy attach minio cloudnative-pg-private --user cloudnative-pg
    ```
