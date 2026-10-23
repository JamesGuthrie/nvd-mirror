# nvd-mirror

A personal mirror of the [NVD](https://nvd.nist.gov/)'s JSON 2.0 data feeds,
for running [vulnix](https://github.com/nix-community/vulnix) without
depending on nvd.nist.gov being fast or available. It is not an official
mirror.

This product uses data from the NVD API but is not endorsed or certified by
the NVD.

## What is mirrored

The feeds vulnix reads: the yearly feeds of the last six years and the
`modified` feed, each as `nvdcve-2.0-<feed>.json.gz` with its `.meta` file,
in the same layout as <https://nvd.nist.gov/feeds/json/cve/2.0/>.

## Usage

```sh
vulnix --mirror https://nvd-mirror.fsn1.your-objectstorage.com/ ...
```

## How the sync works

The "Sync NVD feeds" workflow runs `sync-nvd` daily. NVD updates the yearly
feeds daily, and the `modified` feed about every two hours.

For each feed, `sync-nvd` compares the `.meta` file at NVD with the one in
the bucket and downloads the feed only if they differ. NVD currently delivers
about 30-60 KB/s per connection and drops connections in the middle of large
files, so downloads run in parallel and resume until complete. Each feed is
checked against the size and SHA256 in its `.meta` file before upload, and
the `.meta` file is uploaded after the feed, so a `.meta` file in the bucket
always describes a complete feed.

A failed scheduled run is the notification that the mirror is out of date.

## Setup

The bucket is `nvd-mirror`, a public Hetzner Object Storage bucket in
`fsn1`; the workflow sets these values. It needs two repository secrets:

| Secret                                | Value                                         |
| ------------------------------------- | --------------------------------------------- |
| `NVD_MIRROR_BUCKET_ACCESS_KEY_ID`     | S3 access key with write access to the bucket |
| `NVD_MIRROR_BUCKET_SECRET_ACCESS_KEY` | its secret key                                |

`sync-nvd` also runs locally, with the settings from the workflow as
environment variables and the keys as `AWS_ACCESS_KEY_ID` and
`AWS_SECRET_ACCESS_KEY`.
