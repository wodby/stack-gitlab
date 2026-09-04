# GitLab Community Edition stack

Wodby 2 stack for a self-hosted GitLab Community Edition installation.

The stack combines GitLab 19, PostgreSQL 17, Valkey 8.1, and OpenSMTPD 7. GitLab repository data is persistent, and
PostgreSQL and Valkey use their own persistent volumes. The PostgreSQL service initializes the GitLab database with
the required `amcheck`, `btree_gist`, and `pg_trgm` extensions and applies GitLab's minimum connection and memory
settings.

## Create an app

Creating an app requires an S3-compatible object-storage integration that supplies the GitLab connection YAML, an
`s3cmd` configuration file for the backup utility, and distinct, pre-created bucket names for artifacts, LFS objects,
uploads, packages, backups, and temporary restore data.

Sign in with the username `root` and the password stored in the GitLab service's generated `root_password` token.
GitLab sends outgoing mail through OpenSMTPD. Connect the OpenSMTPD service to a third-party SMTP integration when the
cluster cannot deliver mail directly or authenticated relay is required.

GitLab Runner is intentionally not bundled. Register a separate Runner to execute CI/CD jobs; GitLab recommends
separate runners for production deployments.

## Capacity and availability

GitLab recommends 16 GB of memory as the baseline for a single-node installation. Ensure the Kubernetes cluster can
schedule the GitLab chart's webservice, Sidekiq, Gitaly, Shell, and Toolbox requests together with the PostgreSQL
request configured by this stack. See the
[GitLab installation requirements](https://docs.gitlab.com/install/requirements/) before increasing usage.

This is a single-instance, non-HA stack. Container Registry, GitLab Pages, the Kubernetes Agent Server, incoming email,
and the bundled monitoring stack are disabled. Production and high-availability installations should follow GitLab's
[Cloud Native Hybrid guidance](https://docs.gitlab.com/administration/reference_architectures/cloud_native/).

## Backups and upgrades

GitLab's Toolbox creates a daily archive containing the database, Gitaly repositories and wikis, and configured GitLab
object-storage data. It uploads the archive to the configured backups bucket at 01:00 in the Kubernetes controller's
time zone. Runs cannot overlap, have a six-hour deadline, and use a dynamically provisioned 100 GiB temporary volume.
Ensure the cluster can provision enough working space and configure backup-bucket retention and independent
replication. These archives are managed directly in that bucket and do not appear as Wodby backup records.

The PostgreSQL service's backup remains an additional database recovery layer. It is not a replacement for a matching
GitLab archive. GitLab excludes Rails secrets from its archives, so keep a separately protected copy. Restoration must
use the same GitLab version and follow the [GitLab Helm restore procedure](https://docs.gitlab.com/charts/backup-restore/restore/),
including stopping webservice and Sidekiq while the restore runs.

Upgrades must follow GitLab's [required upgrade stops](https://docs.gitlab.com/update/upgrade_paths/). Complete
background migrations and create a tested backup before advancing to the next required stop.

## Catalog requirements

Before importing this stack, the target Wodby catalog must provide GitLab 19, PostgreSQL 17, Valkey 8.1, and
OpenSMTPD 7. The Wodby control plane must also support exact selectors and managed replica values for Helm charts
that render multiple workloads.
