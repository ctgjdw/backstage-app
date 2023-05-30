# Backstage App

This is a backstage app basic setup with PSQL backend and GitLab auth provider. You will need to setup an application in GitLab to enable the authentication feature. Follow the instructions [here](https://backstage.io/docs/auth/gitlab/provider/).

## Running the app

This backstage app can be run in:

1. Kubernetes (Production)
    1. `kubectl apply -f kubernetes/backstage-rbac.yaml` to create the required RBAC permissions and objects
    1. Create the required [secrets](#kubernetes-secrets)
    1. Create the directory in Windows (`C:\KubeData\backstage-db`) for the db vol mount
    1. Deploy via `kubectl apply -f kubernetes/backstage.yaml`
2. Docker (Production)
    1. Deploy via `docker-compose up -d --build`
3. Local Node Environment (Development)
    1. cd to `backstage` and, run `yarn install` and deploy via `yarn dev`

## Entities

Entities are important in backstage and it is configured in the `./catalog` folder. The entities represent the data model used in the backstage catalog. The `User` entity is also used for sign-in with Gitlab. Entities follow a format similar to kubernetes: `kind:namespace/name` where kind is the type.

## Environment Variables

These are the environment variables used in the backstage config files (`backstage/app-config.yaml` and `backstage/app-config.production.yaml` (used in the `Dockerfile`)):

For `docker-compose`, you can create a `.env` file as below:

```env
# Required in both development and production
POSTGRES_PASSWORD=<DB_PASSWORD>
POSTGRES_HOST=<DB_HOSTNAME>
POSTGRES_PORT=<DB_PORT>
POSTGRES_USER=<DB_USERNAME>

# Required in production
AUTH_GITLAB_CLIENT_ID=<GITLAB_APP_ID>
AUTH_GITLAB_CLIENT_SECRET=<GITLAB_APP_SECRET>
AUTH_GITLAB_URL=<GITLAB_URL>
GITLAB_HOSTNAME=<GITLAB_HOSTNAME_EXCLUDING_PROTOCOL>
```

## Authentication

Currently, authentication with Gitlab is only enabled in `production` mode. `production` mode is enabled in the Docker image and not with `yarn dev`.

1. You will need to setup an application in GitLab to enable the authentication feature. Follow the instructions [here](https://backstage.io/docs/auth/gitlab/provider/). After which, the above `AUTH_...` env variables will need to be set.

2. Lastly, you will need to modify the User entity to match your GitLab username in the file `catalog/users/user.yaml` and edit the git location of `catalog.locations[0].target` in `app-config.production.yaml` for `Users`.

## Kubernetes Secrets

The following secrets will need to be created (create a new yaml file and run `kubectl apply -f <FILENAME>`):

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
    name: backstage-db-secret
stringData:
    POSTGRES_PASSWORD: ''
    POSTGRES_USER: ''
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
    name: backstage-app-secret
stringData:
    AUTH_GITLAB_CLIENT_ID: ''
    AUTH_GITLAB_CLIENT_SECRET: ''
    K8S_SVC_ACC_TOKEN: '' # retrieved from `kubectl create token <SERVICE_ACC_NAME> --duration=1000000m`
    K8S_CONFIG_CA_DATA: '' # retreived from `C:\users\<USER>\.kube\config` in `clusters[].certificate-authority-data`
```
