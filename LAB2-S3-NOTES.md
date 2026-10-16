# Lab 2 - Object storage with S3: notes

Completed on 2026-10-16.

## Results

- Bucket `user-p-veyret-ece` on SeaweedFS (https://s3.seaweedfs.adm.adaltas.cloud).
- Basic operations: upload, list (aws s3 and s5cmd), download (identical to the original), rename, delete.
- Metadata: the ETag of a single PUT equals the MD5 of the file; user-defined metadata is returned by `head-object`.
- Presigned GET URL downloaded with curl, without credentials. Presigned PUT URL generated with boto3.
- Multipart: a 200 MB file was uploaded in 25 parts (ETag ending in `-25`).
- Versioning: 3 versions (including a `null` version for the object uploaded before versioning was enabled), restore of a previous version, delete marker.
- Lifecycle: the 3 rules were accepted by SeaweedFS.
- Bucket policy: the Deny on `s3:DeleteObject` under `bronze/` is enforced for authenticated users (`AccessDenied`).
- Object ACL: owner `admin` with `FULL_CONTROL`.
- Consistency: read-after-write is immediate.
- Kubernetes Job: `users.csv` and `orders.csv` uploaded to `bronze/` from a Job using a ConfigMap and a Secret.

## Issues met and fixes

- No `default` AWS profile in the service: created it from the `AWS_*` environment variables.
- Storage returned `InternalError` (HTTP 500) on every PutObject (9 and 12 October): platform issue, fixed by the instructor.
- Expired credentials after the fix (`InvalidAccessKeyId`): reloaded fresh credentials from the Onyxia portal.
- AWS CLI checksums: set `request_checksum_calculation` and `response_checksum_validation` to `when_required` for SeaweedFS compatibility, in the profile and in the Job's ConfigMap.
- The Job was refused by the namespace quota (`must specify limits.cpu`): added `limits.cpu: 200m` to the container.
- `cp ... /dev/null` returns an error code because the file date cannot be updated: the consistency check was done with a temporary file instead.
