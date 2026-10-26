# Shipwave on Kubernetes

The chart deploys the API and the pre-built worker. The API image is built from `api/`; its protected shipment routes require an `API_KEY` Kubernetes Secret, while `/health` is used for Kubernetes probes.

## Build and publish the API image

From the repository root, set the image tag to the commit SHA and publish to the namespace's registry path:

```powershell
$imageTag = (git rev-parse HEAD).Trim()
$image = "$env:ACR_LOGIN_SERVER/$env:NAMESPACE/api:$imageTag"
docker build -t $image .\api
docker push $image
```

## Configure the API key

Create a Kubernetes Secret named `shipwave-api` in the target namespace, with a key named `API_KEY`. Provision the value through your approved secret-management process; do not put it in Helm values or commit it to the repository. The chart defaults reference this Secret and key. Override `api.apiKeySecret.name` or `api.apiKeySecret.key` if using a different Secret.

## Deploy or upgrade

```powershell
helm upgrade --install shipwave .\helm `
  --namespace $env:NAMESPACE --create-namespace `
  --set-string api.image.repository="$env:ACR_LOGIN_SERVER/$env:NAMESPACE/api" `
  --set-string api.image.tag=$imageTag
```

Check rollout and service state:

```powershell
kubectl -n $env:NAMESPACE rollout status deployment/shipwave-api
kubectl -n $env:NAMESPACE get deployment,service shipwave-api
```

The API Service is `ClusterIP` by default. For a local smoke test, forward it with `kubectl -n $env:NAMESPACE port-forward service/shipwave-api 8000:80`, then request `http://127.0.0.1:8000/health`. Shipment routes require the `X-API-Key` header.