# Option for sycning part or all of one bucket to another bucket.

## Tool
`s3cmd` cli tool

## License
The `s3cmd` command is published under the GPLv2 License.

## Why
The `s3cmd` command is built for the S3 infrastructure and thus should be able to be used for CEPH as well. This tool includes a lot of the features we're looking for, such as:
- MD5 checksum verification before and after transfer
- continue and restart uploading files if the size and MD5 don't match up
- multi-part chunked uploading with variable sizes between 5MB - 5GB
- deleting files from the source AFTER confirming upload to destination
- excluding or including based on GLOBs and REGEXP. (include overrides exclude)
- preserve or set ACL
- skip existing files. Prevents accedental re-uploading of same files
- rate limiting, if we wanted to use slower transfer speeds to prevent collisions over the wire
- ability to stop transfer if an error is detected

## Syntax
`s3cmd [OPTIONS] sync s3://BUCKET_SRC[/PREFIX] s3://BUCKET_DEST[/PREFIX]`

### Options that may be of interest
`--access_key=ACCESS_KEY`
`--secret_key=SECRET_KEY`
`--access_token=ACCESS_TOKEN`
`--debug`
`--dry-run`
`--encrypt`
`--continue-put` = restarts uploads that dont match size and MD5
`--upload-id=UPLOAD_ID` = explictly continue upload based on UPLOAD_ID
`--skip-existing`
`--recursive`
`--check-md5`
`--acl-public` = set ACL to allow all to read
`--acl-private` = set ACL to owner access only
`--acl-grant=PERMISSION:<EMAIL | USER_CANONICAL_ID>` = assign ACL per user
`--acl-revoke=PERMISSION:USER_CANONICAL_ID`
`--delete-after`
`--add-destination=ADDITIONAL_DESTINATIONS` = parallel uploads. Good idea for redundent backups
`--preserve` = keep filesystem attributes the same
`--no-preserve` = don't store filesystem attributes in destination
`--host=HOSTNAME:PORT`
`--host-bucket=HOST_BUCKET` = DNS-style bucket+hostname:port template for accessing a bucket. Used in conjunction with `--host=HOSTNAME:PORT` in the format of `HOST_BUCKET.HOSTNAME:PORT`
`--server-side-encryption` = use server side encryption
`--server-side-encryption-kms-id=KMS_KEY` = set server side encryption key
`--verbatim` = use the S3 name as given, no pre-processing or encoding
`--multipart-chunk-size-mb=SIZE` = size of each chunk of data in MB. SIZE >= 5MB <= 5GB
`--stop-on-error` = stop transfer if an error is detected

#### include > exclude
Can use the following options to include or exclude explictly via GLOB and/or REGEXP files. Included files take precedence over excluded files.
`--exclude=GLOB`
`--exclude-from=FILE`
`--rexclude=REGEXP`
`--rexclude-from=FILE`
`--include=GLOB`
`--include-from=FILE`
`--rinclude=REGEXP`
`--rinclude-from=FILE`

## Usage
`s3cmd --access_key=$ACCESS_KEY --secret_key=$SECRET_KEY --access_token=$ACCESS_TOKEN --debug --dry-run --continue-put --skip-existing --recursive --check-md5 --acl-public --delete-after --multipart-chunk-size-mb=5000 --stop-on-error sync s3://$BUCKET_SRC s3://$BUCKET_DEST`
