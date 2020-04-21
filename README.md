# Bootstrapping Manifests

## Prerequisites

You need access to a recent OpenShift (4.3) cluster.

You will need to have completed the installation of the following additional
components:

 * OpenShift Pipelines Operator
 * Sealed Secrets [installed](https://github.com/bitnami-labs/sealed-secrets#installation)
 * ArgoCD [installed](https://argoproj.github.io/argo-cd/getting_started/#1-install-argo-cd)

You will need to be logged in with administrator privileges.

During installation, we check that the Tekton resources are installed, also, we
generate sealed secrets using the Sealed Secrets operator.

## Generating credentials for pushing images

The default CI pipeline builds an image from your source application, and pushes
it to a repository, it will require credentials for this.

See [here](./quay_io.md) for a guide to creating the required credentials, if you're
using Quay.io.

## Bootstrapping the Manifest

```shell
$ odo manifest bootstrap \
  --app-repo-url https://github.com/<username>/taxi.git \
  --app-webhook-secret testing \
  --gitops-repo-url https://github.com/<username>/gitops.git \
  --gitops-webhook-secret testing2 \
  --image-repo quay.io/<username>/taxi \
  --dockercfgjson ~/Downloads/<username>-auth.json --prefix tst-
```

| Option                  | Description |
| ----------------------- | ----------- |
| --app-repo              | This is the source code to your first application. |
| --app-webhook-secret    | Creates a secret used to validate incoming hooks. |
| --gitops-repo-url       | This is where your configuration and manifest live. |
| --gitops-webhook-secret | This is used to validate incoming hooks. |
| --image-repo            | Where should we configure your builds to push to? |
| --dockercfgjson         | This is used to authenticate image pushes to your image-repo. |
| --prefix                |  This is used to help separate user namespaces. |

## Exploring the Manifest

## Bringing the bootstrapped environment up

## Changing the initial deployment

## Your first CI run

## Changing the default CI run

## Adding an additional application

