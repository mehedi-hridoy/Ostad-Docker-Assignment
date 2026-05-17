# Kubernetes Deployment for Ostad Chat Application

This directory contains all Kubernetes YAML manifests for deploying the multi-component Ostad application.

## Files Overview

| File | Purpose |
|------|---------|
| `01-namespace.yaml` | Creates isolated namespace `ostad-reza` |
| `02-configmap.yaml` | Stores non-sensitive environment variables |
| `03-secret.yaml` | Stores sensitive credentials (MongoDB username/password) |
| `04-pvc.yaml` | PersistentVolumeClaim for MongoDB data storage |
| `05-mongo-deployment.yaml` | MongoDB database deployment |
| `06-mongo-service.yaml` | Service to expose MongoDB internally |
| `07-backend-deployment.yaml` | Express.js backend deployment |
| `08-backend-service.yaml` | Service to expose backend internally |
| `09-frontend-deployment.yaml` | React/Vite frontend deployment |
| `10-frontend-service.yaml` | Service to expose frontend (NodePort) |
| `11-mongo-express-deployment.yaml` | Mongo Express admin panel deployment |
| `12-mongo-express-service.yaml` | Service to expose Mongo Express (NodePort) |
| `13-ingress.yaml` | Ingress for host-based routing (chat.local, mongo.local) |

## Deployment Order

Apply manifests in order (they depend on earlier resources):

```bash
# 1. Create namespace and storage
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-secret.yaml
kubectl apply -f 04-pvc.yaml

# 2. Deploy MongoDB first (backend depends on it)
kubectl apply -f 05-mongo-deployment.yaml
kubectl apply -f 06-mongo-service.yaml

# 3. Deploy backend (frontend depends on it)
kubectl apply -f 07-backend-deployment.yaml
kubectl apply -f 08-backend-service.yaml

# 4. Deploy frontend and mongo-express
kubectl apply -f 09-frontend-deployment.yaml
kubectl apply -f 10-frontend-service.yaml
kubectl apply -f 11-mongo-express-deployment.yaml
kubectl apply -f 12-mongo-express-service.yaml

# 5. Deploy ingress (last, requires ingress controller)
kubectl apply -f 13-ingress.yaml
```

Or apply all at once:
```bash
kubectl apply -f .
```

## Namespace

All resources are deployed in the `ostad-mehedi` namespace.

## Architecture

```
├─ MongoDB (Port 27017)
│  └─ PersistentVolume (1Gi storage)
├─ Backend API (Port 5050)
├─ Frontend (Port 5173) → NodePort 30173
├─ Mongo Express (Port 8081) → NodePort 30081
└─ Ingress (chat.local, mongo.local)
```

## Verification Commands

```bash
# Check namespace
kubectl get ns

# Check all resources in namespace
kubectl get all -n ostad-mehedi

# Check pods status
kubectl get pods -n ostad-mehedi

# Check services
kubectl get svc -n ostad-mehedi

# Check deployments
kubectl get deployments -n ostad-mehedi

# View pod logs
kubectl logs -n ostad-mehedi <pod-name>

# Describe a resource
kubectl describe pod -n ostad-mehedi <pod-name>
```

## Resource Limits

Each container includes:
- **Requests**: Minimum resources guaranteed
- **Limits**: Maximum resources allowed

- MongoDB: 256Mi-512Mi memory, 250m-500m CPU
- Backend: 128Mi-256Mi memory, 100m-500m CPU
- Frontend: 128Mi-256Mi memory, 100m-500m CPU
- Mongo Express: 128Mi-256Mi memory, 100m-500m CPU

## Accessing the Application

### Local Access (NodePort)
- Frontend: `http://localhost:30173`
- Mongo Express: `http://localhost:30081`

### Via Ingress (requires entries in /etc/hosts)
Add to `/etc/hosts`:
```
127.0.0.1 chat.local
127.0.0.1 mongo.local
```

Then access:
- Frontend: `http://chat.local`
- Mongo Express: `http://mongo.local`

## MongoDB Credentials

- Username: `root`
- Password: `root123`
- Database: `Ostad-DB`

## Environment Variables

### ConfigMap (ostad-config)
- MONGO_SERVER: mongo
- MONGO_PORT: 27017
- MONGO_DATABASE: Ostad-DB
- BACKEND_URL: http://ostad-server:5050
- VITE_SERVER_URL: http://localhost:5050
- ME_CONFIG_MONGODB_SERVER: mongo

### Secret (ostad-secret)
- MONGO_ROOT_USERNAME: root
- MONGO_ROOT_PASSWORD: root123
- MONGO_URI: mongodb://root:root123@mongo:27017/
- ME_CONFIG_MONGODB_ADMINUSERNAME: root
- ME_CONFIG_MONGODB_ADMINPASSWORD: root123

## Notes

- `imagePullPolicy: Never` is used for locally built images (ostad-server, ostad-ui)
- MongoDB uses PVC for persistent storage
- All internal services use ClusterIP
- Frontend and Mongo Express use NodePort for external access
- Ingress requires an ingress controller (nginx) to be installed
