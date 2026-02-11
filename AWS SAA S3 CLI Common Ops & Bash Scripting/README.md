# Overview

This project demonstrates hands-on automation of the Amazon S3 object lifecycle using Bash scripting and the AWS CLI.

The goal was to understand how object storage works operationally (not just conceptually) while building reusable automation similar to what Cloud Engineers, DevOps, and Security Operations teams use in production.

## Lifecycle Covered

Create buckets

Upload single files

Sync folders

List objects

Download objects

Inspect object metadata

Empty buckets (including versioned objects)

Delete buckets

## Section 0 – Environment Setup
# What I Did

Used WSL (Linux shell on Windows)

Installed and configured AWS CLI

Authenticated using IAM credentials

Created a working directory:

mkdir ~/aws-scripts
cd ~/aws-scripts


Made scripts executable:

chmod u+x *.sh

# What This Is

A Linux-based automation environment with authenticated AWS CLI access.

# Why It Matters

Production cloud automation almost never happens in the AWS Console.
Real environments rely on Linux + CLI + scripting.

Workforce Usage

Cloud engineering

DevOps automation

SOC scripting

Infrastructure management

Security Considerations

Use IAM roles/users with least privilege

Never hardcode credentials

Rotate access keys

Prefer IAM roles when possible

Use profiles or environment variables

## Section 1 – Create S3 Bucket
Script: create-s3-bucket.sh
#!/usr/bin/env bash

if [ -z "$1" ]; then
  echo "Usage: ./create-s3-bucket.sh <bucket-name>"
  exit 1
fi

aws s3api create-bucket --bucket "$1"

Example Run
./create-s3-bucket.sh example-bucket

# What This Does

Creates a new S3 bucket from the CLI.

# Why It Matters

Buckets are the foundation of S3 storage. Every workload relies on them.

Real-World Usage

Log storage

Backups

CI/CD artifacts

Security evidence

Data lakes

Security Considerations

Block public access by default

Enable encryption

Enable versioning

Log bucket activity

## Section 2 – Upload Single Object
Script: put-s3-object.sh
#!/usr/bin/env bash

if [ -z "$1" ] || [ -z "$2" ]; then
  echo "Usage: ./put-s3-object.sh <local-file> <bucket>"
  exit 1
fi

aws s3 cp "$1" "s3://$2/"

Example Run
./put-s3-object.sh ./file.jpg example-bucket

# What This Does

Uploads a single file to S3.

Workforce Usage

Evidence uploads

Artifact publishing

User file ingestion

## Section 3 – Sync Folder
Script: sync-s3-folder.sh
#!/usr/bin/env bash

if [ -z "$1" ] || [ -z "$2" ]; then
  echo "Usage: ./sync-s3-folder.sh <bucket> <local-folder>"
  exit 1
fi

aws s3 sync "$2" "s3://$1/uploads/"

Example Run
./sync-s3-folder.sh example-bucket ./data

# What This Does

Synchronizes an entire directory to S3.

Workforce Usage

Backups

Website deployment

Dataset ingestion

## Section 4 – List Objects
Script: list-s3-objects.sh
#!/usr/bin/env bash

if [ -z "$1" ]; then
  echo "Usage: ./list-s3-objects.sh <bucket>"
  exit 1
fi

aws s3 ls "s3://$1"

# What This Does

Lists all objects in a bucket.

## Section 5 – Download Object
Script: get_object.sh
#!/usr/bin/env bash

if [ -z "$1" ] || [ -z "$2" ] || [ -z "$3" ]; then
  echo "Usage: ./get_object.sh <bucket> <object> <output>"
  exit 1
fi

aws s3 cp "s3://$1/$2" "$3"

## Section 6 – Inspect Metadata
Script: head_object.sh
#!/usr/bin/env bash

if [ -z "$1" ] || [ -z "$2" ]; then
  echo "Usage: ./head_object.sh <bucket> <object>"
  exit 1
fi

aws s3api head-object --bucket "$1" --key "$2"

## Section 7 – Empty Bucket (Version Aware)
Script: empty_bucket.sh
#!/usr/bin/env bash

if [ -z "$1" ]; then
  echo "Usage: ./empty_bucket.sh <bucket>"
  exit 1
fi

aws s3 rm "s3://$1" --recursive

## Section 8 – Delete Bucket
Script: delete-s3-bucket.sh
#!/usr/bin/env bash

if [ -z "$1" ]; then
  echo "Usage: ./delete-s3-bucket.sh <bucket>"
  exit 1
fi

aws s3 rb "s3://$1"

AWS SAA Exam Mapping

This project directly reinforces:

S3 fundamentals

Bucket lifecycle

Object operations

CLI vs Console

IAM permissions

Encryption concepts

Versioning

Automation

Secure access patterns

# Key Takeaways

Implemented full S3 lifecycle via CLI

Built reusable Bash automation

Practiced validation and error handling

Applied security best practices

Gained real-world cloud operational experience

# Why This Matters

This mirrors how production teams manage storage:

Not manually.
Not through the console.

Through automation.
