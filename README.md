# Asynchronous Tasks with FastAPI and Celery

Example of how to handle background processes with FastAPI, Celery, and Docker. 
See:
    https://testdriven.io/blog/fastapi-and-celery/#conclusion
    https://github.com/testdrivenio/fastapi-celery

## Want to learn how to build this?

Check out the [post](https://testdriven.io/blog/fastapi-and-celery/).

## Want to use this project?

Spin up the containers:

```sh
$ docker-compose up -d --build
$ docker-compose stop
$ docker compose rm -f
```

Open your browser to [http://localhost:8004](http://localhost:8004) to view the app or to [http://localhost:5556](http://localhost:5556) to view the Flower dashboard.

Trigger a new task:

```sh
$ curl http://localhost:8004/tasks -H "Content-Type: application/json" --data '{"type": 0}'
```

Check the status:

```sh
$ curl http://localhost:8004/tasks/<TASK_ID>
```

### Docker compose generates the following images:
fastapi-celery_web
fastapi-celery_worker
fastapi-celery_dashboard

*But these images are no good - docker compose is using mounted volumes and not actually copying files into the containers.
Remove the persistent volume claim files and references to it in the deployment yaml files for worker and web.
Then add some layers to the dockerfile using the commands in the docker compose as the CMD in the dockerfile

```sh
docker build -t devrlsharedacr.azurecr.io/fastapi-celery_web:latest --target web ./project
docker build -t devrlsharedacr.azurecr.io/fastapi-celery_worker:latest --target worker ./project
docker build -t devrlsharedacr.azurecr.io/fastapi-celery_dashboard:latest --target dashboard ./project
```

### Install kompose: 
https://kompose.io/installation/

### run kompose:
```sh
$ kompose convert
```
This generates some volumes, due to how the dockerfile is written. I removed the volume related yaml blocks, and had to update the dockerfile to include some layers to build the images with the right commands.


### Upload images to a container registry, then update the images in web-deployment.yaml, worker-deployment.yaml, and dashboard-deployment.yaml
```sh
docker push devrlsharedacr.azurecr.io/fastapi-celery_web:latest
docker push devrlsharedacr.azurecr.io/fastapi-celery_worker:latest
docker push devrlsharedacr.azurecr.io/fastapi-celery_dashboard:latest
```

### Replace just the 'image' yaml field with the new image name with the fully qualified container registry version of the image:

```sh
image: web       -> image: registryHost.com/fastapi-celery_web
image: worker    -> image: registryHost.com/fastapi-celery_worker
image: dashboard -> image: registryHost.com/fastapi-celery_dashboard
```

#### Be sure your cluster has image pull creds for the registry you're pushing the images to
See: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

```sh
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

### Once the yaml files are updated, run:
```sh
kubectl apply -f k8s

kubectl delete -f k8s
```

### Create a helm chart:
helm create base-chart

### create override files and make sure they work. 
#### These will be similar to the k8s yaml, but we should be able to use the same chart for all components (web, dashboard, redis, worker).

### check rendering of overrides:
```sh
cd helm
helm template base-chart -f ./overrides/local.dashboard.yaml
helm template base-chart -f ./overrides/local.redis.yaml
helm template base-chart -f ./overrides/local.web.yaml
helm template base-chart -f ./overrides/local.worker.yaml
```

### install the helm charts
```sh
cd helm
helm install dashboard base-chart -f ./overrides/local.dashboard.yaml
helm install redis base-chart -f ./overrides/local.redis.yaml
helm install web base-chart -f ./overrides/local.web.yaml
helm install worker base-chart -f ./overrides/local.worker.yaml
```

```helm list``` to see charts installed in your cluster, 
```helm uninstall <NAME>``` to remove chart
