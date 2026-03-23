# Deploying the Notes API to Google Kubernetes Engine (GKE)

This project demonstrates how to containerize and deploy the Notes API back-end to Google Cloud using Artifact Registry and Google Kubernetes Engine (GKE).

## Purpose

The goal of this project is to run the Notes API as a containerized back-end service and expose it through a public endpoint so it can be integrated with the existing Notes App front-end.

## Source

The back-end source code is based on the Dicoding repository below:

- `https://github.com/dicodingacademy/a332-google-cloud-architect-labs`

## Architecture

- `Artifact Registry` stores the Docker image
- `Google Kubernetes Engine (Standard)` runs the application container
- `Kubernetes Deployment` manages the application pod
- `Kubernetes Service (LoadBalancer)` exposes the application publicly

## Deployment Configuration

- Project ID: `<PROJECT_ID>`
- Artifact Registry region: `<ARTIFACT_REGION>`
- GKE zone: `<GKE_ZONE>`
- Cluster mode: `Standard`
- Node count: `1`
- Machine type: `e2-small`
- Container port: `5000`
- Service port: `8080`
- External endpoint: `http://<EXTERNAL_IP>:8080`

## Deployment Steps

### 1. Build the Docker image

Build the application image locally from the project source code.

```bash
docker build -t notes-api:local .
```

### 2. Enable the required Google Cloud APIs

Enable the services needed for storing images and running Kubernetes workloads.

```bash
gcloud services enable artifactregistry.googleapis.com
gcloud services enable container.googleapis.com
```

### 3. Create an Artifact Registry repository

Create a Docker repository in Jakarta region to store the application image.

```bash
gcloud artifacts repositories create notes-api-repo \
  --repository-format=docker \
  --location=<ARTIFACT_REGION> \
  --description="Docker repository for Notes API submission"
```

### 4. Configure Docker authentication

Allow Docker to authenticate with Artifact Registry.

```bash
gcloud auth configure-docker <ARTIFACT_REGION>-docker.pkg.dev
```

### 5. Tag and push the image

Tag the local image with the Artifact Registry path, then push it to the remote repository.

```bash
docker tag notes-api:local <ARTIFACT_REGION>-docker.pkg.dev/<PROJECT_ID>/notes-api-repo/notes-api:v1
docker push <ARTIFACT_REGION>-docker.pkg.dev/<PROJECT_ID>/notes-api-repo/notes-api:v1
```

### 6. Create the GKE cluster

Provision a zonal Standard cluster in Jakarta with a single node.

```bash
gcloud container clusters create notes-api-cluster \
  --zone <GKE_ZONE> \
  --num-nodes 1 \
  --machine-type e2-small \
  --release-channel regular \
  --disk-size 30
```

### 7. Connect `kubectl` to the cluster

Install the GKE authentication plugin, then fetch the cluster credentials.

```bash
gcloud components install gke-gcloud-auth-plugin
export USE_GKE_GCLOUD_AUTH_PLUGIN=True
gcloud container clusters get-credentials notes-api-cluster --zone <GKE_ZONE>
```

For PowerShell:

```powershell
$env:USE_GKE_GCLOUD_AUTH_PLUGIN="True"
gcloud container clusters get-credentials notes-api-cluster --zone <GKE_ZONE>
```

### 8. Create the Kubernetes Deployment

Deploy the application image to the GKE cluster.

```bash
kubectl create deployment notes-api \
  --image=<ARTIFACT_REGION>-docker.pkg.dev/<PROJECT_ID>/notes-api-repo/notes-api:v1
```

### 9. Expose the application with a LoadBalancer service

Publish the application using a public service and map port `8080` to container port `5000`.

```bash
kubectl expose deployment notes-api \
  --type=LoadBalancer \
  --port=8080 \
  --target-port=5000
```

### 10. Verify the deployment

Check the pod and service status to confirm that the application is running and publicly accessible.

```bash
kubectl get pods -A
kubectl get svc
```

Expected result:

- The `notes-api` pod is in `Running` state
- The `notes-api` service has an `EXTERNAL-IP`
- `GET /` returns `404 Not Found`
- `GET /notes` returns a successful JSON response

## Operational Note

The cluster was finalized with `e2-small` because `e2-micro` caused `Insufficient memory` on essential system pods such as `kube-dns`, which prevented the workload from becoming healthy.
