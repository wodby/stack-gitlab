# GitLab Community Edition stack

Wodby 2 stack for a self-hosted GitLab Community Edition installation.

The stack combines GitLab 19, PostgreSQL 17, and Valkey 8.1. GitLab repository data is persistent, and PostgreSQL and
Valkey use their own persistent volumes. Creating an app from this stack also requires an S3-compatible object-storage
integration that supplies the GitLab connection YAML and bucket names declared by the GitLab service.

## Rollout order

1. Merge and deploy the backend chart-compatibility changes.
2. Merge and import PostgreSQL 17 support.
3. Merge and import the GitLab Community Edition service.
4. Import this stack.

Repository publication and catalog import are intentionally separate rollout steps.
