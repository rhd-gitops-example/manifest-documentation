# Bootstrapping Manifests

## Prerequisites

You need access to a recent OpenShift (4.3) cluster.

You will need to have completed the installation of the following additional
components:

 * OpenShift Pipelines Operator
 * Sealed Secrets [installed](https://github.com/bitnami-labs/sealed-secrets#installation)
 * ArgoCD [installed](https://operatorhub.io/operator/argocd-operator) this
   _MUST_ be installed to a project called `argocd`.

You will need to be logged in with administrator privileges.

During installation, we check that the Tekton resources are installed, also, we
generate sealed secrets using the Sealed Secrets operator.

## Generating credentials for pushing images

The default CI pipeline builds an image from your source application, and pushes
it to a repository, it will require credentials for this.

See [here](./quay_io.md) for a guide to creating the required credentials, if you're
using Quay.io.

## Manifest Model

The manifest models environments, applications and services.

An environment can have many applications, which in-turn can reference many
services.

Services are "per-environment", and multiple applications _within_ an
environment can reference the same service, and override the configuration with
Kustomize.

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

## Exploring the manifest

The bootstrap process generates a fairly large number of files, including a
manifest describing your first application, and configuration for a complete CI
pipeline and deployments from ArgoCD.

```yaml
environments:
- name: tst-dev
  pipelines:
    integration:
      binding: github-pr-binding
      template: app-ci-template
  apps:
  - name: taxi
    services:
    - name: taxi-svc
      source_url: https://github.com/<username>/taxi.git
      webhook:
        secret:
          name: github-webhook-secret-taxi-svc
- name: tst-stage
- name: tst-cicd
  cicd: true
- name: tst-argocd
  argo: true
```

The bootstrap creates four environments, `dev`, `stage`, `cicd` and `argocd`
with a user-supplied prefix.

The name of the app and service are derived from the last component of your
`app-repo-url` e.g. if your bootstrap with `--app-repo-url
https://github.com/myorg/myproject.git` this would bootstrap an app called
`myproject` and a service called `myproject-svc`.

## Environment configuration

The `tst-dev` environment created is the most visible configuration.

### configuring pipelines

```yaml
environments:
- name: tst-dev
  pipelines:
    integration:
      binding: github-pr-binding
      template: app-ci-template
  apps:
  - name: taxi
    services:
    - name: taxi-svc
      source_url: https://github.com/<username>/taxi.git
      webhook:
        secret:
          name: github-webhook-secret-taxi-svc
```

The `pipelines` key describes how to trigger a Tekton PipelineRun, the
`integration` binding and template are processed when a _Pull Request_
is opened.

This is the default pipeline specification for the `tst-dev` environment, you
can find the definitions for these in these two files:

 `./environments/<prefix>cicd/base/pipelines/07-templates/app-ci-build-pr-template.yaml`
 `./environments/<prefix>cicd/base/pipelines/06-bindings/github-pr-binding.yaml`

By default this triggers a PipelineRun of this pipeline

 `./environments/<prefix>cicd/base/pipelines/05-pipelines/app-ci-pipeline.yaml`

These files are not managed directly by the manifest, you're free to change them
for your own needs, by default they use [Buildah](https://github.com/containers/buildah)
to trigger build, assuming that the Dockerfile for your application is in the root
of your repository.

### configuring services

```yaml
apps:
- name: taxi
  services:
  - name: taxi-svc
    source_url: https://github.com/<username>/taxi.git
    webhook:
      secret:
        name: github-webhook-secret-taxi-svc
```

This defines an app called `taxi`, which has a single service called `taxi-svc`.

The configuration for these is written out to:

 `./environments/<prefix>dev/services/taxi-svc/base/config/`
 `./environments/<prefix>dev/apps/taxi/base/config/`

The `taxi` app's configuration references the services configuration.

The `source_url` references the upstream repository for the service, this
coupled with the `webhook.secret` is also used to drive the CI pipeline, with a
hook confgured, changes to this repository will trigger the template and
binding), the secret is used to authenticate incoming hooks from GitHub.

## Bringing the bootstrapped environment up

```shell
$ git init .
$ git add .
$ git commit -m "Initial commit."
$ git remote add origin <insert gitops repo>
$ git push -u origin master
$ oc apply -k environments/tst-dev/env/base
$ oc apply -k environments/tst-argocd/config
$ oc apply -k environments/tst-cicd/base
```

## Changing the initial deployment

## Your first CI run

## Changing the default CI run

## Adding an additional application

