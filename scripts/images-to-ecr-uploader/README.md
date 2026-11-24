# Download and Upload Docker Images to Amazon ECR

This script downloads the latest available **Spatial SDK Docker Image** delivery from Precisely Data Experience (PDX), extracts its Docker image archives, and pushes the images to Amazon Elastic Container Registry (ECR).

Access to the PDX product requires a subscription. Contact Precisely or visit [Precisely Data Experience](https://data.precisely.com/) to request access.

## Prerequisites

- Python and the packages listed in `requirements.txt`
- Docker installed and running
- AWS CLI installed and available on `PATH`
- PDX API key and shared secret with access to the product
- AWS credentials with permission to identify the account, authenticate to ECR, create repositories when needed, and push images

AWS credentials can be supplied as command-line options or through the AWS CLI's configured credential chain.

## Install Python requirements

From this directory, run:

```console
pip install -r requirements.txt
```

## Run the uploader

From this directory, run:

```console
python upload_ecr.py --pdx-api-key <pdx-api-key> --pdx-api-secret <pdx-api-secret> --aws-region <aws-region> --aws-access-key <aws-access-key> --aws-secret-key <aws-secret-key>
```

The AWS options are optional if AWS credentials are already configured. The region defaults to `us-east-1`. To use a different local staging location, add `--local-path <directory>`; by default, the script uses the current working directory.

For each `.tar` archive in the downloaded delivery, the script loads the image into Docker, creates an ECR repository if it does not already exist, tags the image with `1.3.3`, and pushes it. The script prints the ECR image URIs when uploads succeed.

> **Note:** The script uses a `docker_images` directory under the selected local path for downloads and extraction, then deletes that directory when it finishes. Do not place files you need to keep in that directory.

[🔗 Return to `Table of Contents` 🔗](../../README.md#components)
