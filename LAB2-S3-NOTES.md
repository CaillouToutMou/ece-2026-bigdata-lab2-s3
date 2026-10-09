# Lab 2 - Object storage with S3: progress notes

Date: 2026-10-09

## Done

- Created the `default` AWS profile from the Onyxia environment variables (it did not exist in the service).
- Endpoint: https://s3.seaweedfs.adm.adaltas.cloud
- Bucket `user-p-veyret-ece` created through the Onyxia file explorer and visible with `aws s3 ls`.
- Installed s5cmd in `~/.local/bin`.
- Generated `users.csv` (7351 bytes) with `uv run dataset-users -o csv`.

## Blocked

Every upload to the bucket fails with an HTTP 500 from the storage backend:


- Same error with `aws s3 cp` and with `s5cmd cp`, even for a 6-byte test file.
- `aws s3 ls` and `aws s3api head-bucket` work, so the credentials and the bucket are fine.
- Request id: 18DCE73D566FBDFCE51E2C37

The rest of the lab will be done once uploads work again.
