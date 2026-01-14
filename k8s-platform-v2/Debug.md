# Debugging & Maintenance Guide

## Deployment Commands

### Standard Redeploy (Recommended)
Use this command to apply all application changes and restart JupyterHub.

```bash
kubectl apply -k 03-apps && kubectl rollout status deployment/jupyterhub
```

### Checking Pod Status & Logs
```bash
kubectl get pods -l app=jupyterhub
kubectl logs -l app=jupyterhub --tail=200 -f
```

### Checking Logs
To see the startup configuration and any Spark/Delta errors:

```bash
kubectl logs -l app=jupyterhub --tail=200 -f
```

## Common Issues & Fixes

### 1. `IllegalArgumentException` (UniForm/Iceberg)
If you see an error about `delta.universalFormat.enabledFormats`:
- Ensure `spark.databricks.delta.universalFormat.enabledFormats` is set to `none` in `spark-defaults.conf`.
- DO NOT set it to an empty string `""`, as Delta's requirement check fails on empty strings.

### 2. `NumberFormatException` (Spark 4.0 Timing)
If you see errors like `NumberFormatException: "60s"`:
- Spark 4.0 is strict about time units. Override Hadoop S3A properties with bare integers (seconds) in `spark-defaults.conf`.
- Example: `spark.hadoop.fs.s3a.threads.keepalivetime 60` (not `60s`).

### 3. Duplicate Pods / `InvalidImageName`
If you see pods stuck in `InvalidImageName`:
- We have standardized `03-apps/jupyterhub.yaml` by hardcoding the images. This bypasses Kustomize substitution issues.
- If you need to change the JupyterHub or Spark image, edit `03-apps/jupyterhub.yaml` directly.
- Cleanup: Run `kubectl delete rs -l app=jupyterhub` to remove old, failing replicasets.
