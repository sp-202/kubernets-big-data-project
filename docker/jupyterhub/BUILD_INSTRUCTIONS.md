# JupyterHub Spark Connect Docker Build

This directory contains the Dockerfile for building a lightweight JupyterHub image that acts as a **Spark Connect thin client**.

## Architecture

- **Base Image**: `subhodeep2022/spark-bigdata:jupyterhub-4.0.7-pyspark-scala-sql-v2`
- **Spark Mode**: Spark Connect (thin client)
- **Spark Server**: Connects to `sc://spark-connect-server-driver-svc:15002`

## Build Instructions

### Option 1: Build Locally

```bash
cd /home/subhodeep/Documents/kubernets-big-data-project/docker/jupyterhub

docker build -f Dockerfile.spark-connect \
  -t subhodeep2022/spark-bigdata:jupyterhub-spark-connect-v1 .
```

### Option 2: Build on Remote Server (Recommended for faster internet)

1. **Copy the Dockerfile to your remote server**:
   ```bash
   scp Dockerfile.spark-connect user@remote-server:/path/to/build/
   ```

2. **SSH into the remote server and build**:
   ```bash
   ssh user@remote-server
   cd /path/to/build/
   
   docker build -f Dockerfile.spark-connect \
     -t subhodeep2022/spark-bigdata:jupyterhub-spark-connect-v1 .
   ```

3. **Push to Docker Hub**:
   ```bash
   docker login
   docker push subhodeep2022/spark-bigdata:jupyterhub-spark-connect-v1
   ```

## Verification

After building, verify the image:

```bash
# Check PySpark version
docker run --rm subhodeep2022/spark-bigdata:jupyterhub-spark-connect-v1 \
  python -c "import pyspark; print(f'PySpark: {pyspark.__version__}')"

# Check grpcio-status
docker run --rm subhodeep2022/spark-bigdata:jupyterhub-spark-connect-v1 \
  python -c "import grpc_status; print('grpcio-status: OK')"
```

Expected output:
```
PySpark: 4.0.1
grpcio-status: OK
```

## What's Different from the Base Image?

This Dockerfile adds:
- `pyspark==4.0.1` (Python library only, no Spark binaries)
- `grpcio-status` (for Spark Connect gRPC communication)

This Dockerfile **removes**:
- No Spark binaries (`SPARK_HOME` not needed)
- No local Spark execution (all jobs run remotely)

## Deployment

Once the image is built and pushed to Docker Hub, update the Kubernetes deployment:

```bash
kubectl apply -f /home/subhodeep/Documents/kubernets-big-data-project/k8s-platform-v2/03-apps/jupyterhub.yaml
```

See the main [implementation_plan.md](../../.gemini/antigravity/brain/3eb7422c-f919-499c-8237-929cf3da60f9/implementation_plan.md) for full deployment instructions.
