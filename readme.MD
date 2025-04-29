# MySQL Deployment with Docker & K8s

This project demonstrates the deployment of a MySQL database in two stages:
- Docker: Running MySQL in a Docker container with persistent storage.
- Kubernetes: Deploying MySQL in a Kubernetes cluster with StatefulSets for high availability and persistent data storage.

## Tools Required
- Docker: Ensure Docker is installed on your local machine for containerization.
- Kubernetes (``kubectl``): To manage the Kubernetes cluster and deploy MySQL.
- Minikube (optional): If you need a local Kubernetes cluster on your machine.

## Prerequisites
 - Docker installed on your system.
- Kubernetes cluster setup (e.g., Minikube or a cloud-based Kubernetes).
- ```kubectl``` and ```docker``` CLI tools installed and configured.

## Directory Structure
```
/project-directory
├── docker-compose.yml          # Docker Compose file for running MySQL container
├── k8s/
│   ├── mysql-pv.yaml           # Persistent Volume configuration for MySQL
│   ├── mysql-pvc.yaml          # Persistent Volume Claim for MySQL
│   ├── mysql-statefulset.yaml  # StatefulSet configuration for MySQL
│   └── mysql-service.yaml      # Service configuration for MySQL
└── README.md                   # Project documentation
```

## Deployment

1. Docker MySQL Setup
1.1. Start MySQL in a Docker container
```docker run --name mysql-container \
  -e MYSQL_ROOT_PASSWORD=rootpass123 \
  -p 3307:3306 \
  -v mysql_data:/var/lib/mysql \
  -d mysql:latest
```

1.2 Connect to MySQL
` docker exec -it mysql-container mysql -u root -p `

2. Kubernetes MySQL Setup

2.1 Apply the Persistent Volume
``` kubectl apply -f k8s/mysql-pv.yaml ```

2.2 Apply the Persistent Volume Claim
``` kubectl apply -f k8s/mysql-pvc.yaml ```

2.3 Deploy the StatefulSet
``` kubectl apply -f k8s/mysql-statefulset.yaml ```

2.4 Create the Service
``` kubectl apply -f k8s/mysql-service.yaml ```

2.5 Verify the resources
```
kubectl get pods
kubectl get pvc
kubectl get svc
```

2.6 Access MySQL inside Kubernetes
``` kubectl exec -it mysql-statefulset-0 -- mysql -u root -p ```

## 3. Configuration

You can customize your MySQL deployment by setting environment variables in  
`docker-compose.yml` or your StatefulSet YAML:

- `MYSQL_ROOT_PASSWORD` – root user password (**required**)  
- `MYSQL_DATABASE` – name of a database to create on startup (**optional**)  
- `MYSQL_USER` – name of a non-root user to create (**optional**)  
- `MYSQL_PASSWORD` – password for the non-root user (**optional**)

### Example excerpt from `docker-compose.yml` environment:

```
services:
  mysql:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass123
      - MYSQL_DATABASE=test_db
      - MYSQL_USER=app_user
      - MYSQL_PASSWORD=app_pass
    ports:
      - "3306:3306"

```

## 4. Data Initialization

If you want to pre-load your database with tables or seed data, place `.sql` scripts in a folder and mount it at `/docker-entrypoint-initdb.d`:

```
volumes:
  - ./init-scripts:/docker-entrypoint-initdb.d
```

Any `.sql` or `.sh` files in `init-scripts/` will be executed in **alphabetical order** on container startup.


## 5. Backup & Restore

### Backup
```
docker exec mysql-container \
  mysqldump -u root -p"$MYSQL_ROOT_PASSWORD" \
  --all-databases > all_databases.sql
```

### Restore

```
docker exec -i mysql-container \
  mysql -u root -p"$MYSQL_ROOT_PASSWORD" < all_databases.sql
```

**For Kubernetes**:  
You can use `kubectl cp` to copy the dump file into a pod and run the same backup/restore commands inside the pod.

## 6. Scaling & High Availability

### Docker Compose

You can scale up by running multiple MySQL containers, but you’ll need to configure replication manually.

### Kubernetes

Increase replicas in your `mysql-statefulset.yaml`:

```
spec:
  replicas: 3
```

Then apply the updated configuration:

```
kubectl apply -f k8s/mysql-statefulset.yaml
```

## 7. Monitoring & Logging

### Logs

For Docker:

`
docker logs mysql-container
`

For Kubernetes:

`
kubectl logs <mysql-pod-name>
`

### Metrics

Integrate with `Prometheus` using the `MySQL Exporter` to collect real-time metrics from your MySQL database.


## 8. Cleanup

### Docker Compose

`
docker-compose down --volumes
`

### Kubernetes

```
kubectl delete -f k8s/mysql-service.yaml \
  -f k8s/mysql-statefulset.yaml \
  -f k8s/mysql-pvc.yaml \
  -f k8s/mysql-pv.yaml
```

## 9. Contributing

1. Fork the repository
2. Create a new feature branch:

    `
    git checkout -b feature/awesome `
    

3. Commit your changes:

    `
    git commit -m 'Add awesome feature'`
    

4. Push to the feature branch:

    `
    git push origin feature/awesome`
    

5. Open a **Pull Request** on GitHub
