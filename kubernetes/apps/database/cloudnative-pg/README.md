# CloudNativePG

## S3 Configuration

1. Create `~/.mc/config.json`.

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

2. Create the cloudnative-pg user and password.

    ```sh
    mc admin user add minio cloudnative-pg <super-secret-password>
    ```

3. Create the CloudNativePG bucket.

    ```sh
    mc mb minio/cloudnative-pg
    ```

4. Create `cloudnative-pg-user-policy.json`

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
    mc admin policy create minio cloudnative-pg-private ~/.mc/cloudnative-pg-user-policy.json
    ```

6. Associate private policy with the user.

    ```sh
    mc admin policy attach minio cloudnative-pg-private --user cloudnative-pg
    ```
