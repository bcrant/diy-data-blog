# Learning Kubernetes

A project I designed as a serverless app needs to instead be deployed to a Kubernetes cluster. 

The plan is to build the serverless Python app with `fastapi` and deploy to the company's AWS Elastic Kubernetes Service.

Before we learn the specifics of AWS EKS, first I want to learn how to set up a local development environment. Off we go...

- Instead I have downloaded Minikube:
`brew install minikube`

- and the Kubernetes CLI `kubectl`
`brew install kubectl`

I recently discovered that Jetbrains has a great webinar series for their product PyCharm, and I've just stumbled across a great tutorial of theirs explaining exactly what we want to do:
https://github.com/mukulmantosh/FastAPI_EKS_Kubernetes

- Add a directory for postgres-data
`mkdir postgres-data` 

- We will use Docker as the Kubernetes engine
```
minikube start \
    --mount-string "$HOME/postgres-data:/data"\
    --driver=docker \
    --install-addons=true \
    --kubernetes-version=stable
```

- Test...
`minikube service list`
`kubectl get nodes`

Next need to create the Dockerfile, but need a Python project for this first!

Of note – look up the Docker 'slim-buster' images "${PYTHON_VERSION}-slim-buster". Might be useful.

detour spent way to long figuring out this
`pyenv install --list | egrep -v '[a-z]'`
and added a cron job to keep my pyenv up to date since it is not install through homebrew
`0 10 5,10,15,20,25 * * cd ~/.pyenv && git pull 2>&1`
back to work

going to use the docker image `python:3.9.12-slim-buster`

`slim` is a heavily pruned version of python that reduces image size from ~400MB to ~40MB. We will find out if that becomes problematic, but for now, sounds good to me.

`buster` is the latest stable Debian build.

Adding a virtualenv to vizifi.

Added Dockerfile and .dockerignore to vizifi (see repo).

Now we need to make a MVP fastapi python project

Added a simple main.py to vizifi.

Added Requirements.txt to vizif (see repo).

Seems like main things here are

fastapi and gunicorn

- Download Docker plugin for PyCharm.
- There is a Kubernetes plugin for PyCharm but it is professional version only.

- Building Docker image...
`docker build -t {DOCKERHUB_USERNAME}/{PROJECT_NAME}:latest .`
`docker push {DOCKERHUB_USERNAME}/{PROJECT_NAME}:latest`

- Make the K8s dir and configs (namespace, code, nginx, postgres, job)

- From `k8s` dir...
`kubectl apply -f namespace/`
- do this for all config dirs (namespace, code, nginx, postgres, job)

- check if running...
`kubectl get pods -n test-fastapi -w`

`kubectl describe pod postgres-deployment`

- Useful for debugging...
`kubectl get events --sort-by=.metadata.creationTimestamp`
and teardown
`kubectl delete all --all -n {NAMESPACE}`
also useful for debugging a pod. Postgres process keeps getting stuck...
`kubectl describe pod postgres-deployment-564f65fcd-wrnzb -n test-fastapi`

This is what you actually want forf debugging... duh:
`kubectl logs -p vizifi-deployment-777444fd6-pkm5c -n test-fastapi`

