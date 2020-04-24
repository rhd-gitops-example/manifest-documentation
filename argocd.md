# Installation of ArgoCD through the Operator Hub

## Step 1

You will need to create the `argocd` namspace, you can easily do this with `oc create namespace argocd`.

![Create argocd namespace](img/argoop/create-argocd-namespace.png)

## Step 2

Find the ArgoCD Operator in the Operator Hub.

![ArgoCD in the Operator Hub](img/argoop/argocd-in-hub.png)

## Step 3

Create a subscription to the operator, this _must_ be in the `argocd` namespace.

![Creating subscription](img/argoop/create-operator-subscription.png)

You should now see the installed operators

![Installed operators](img/argoop/installed-operators.png)

## Step 4

Create an ArgoCD installation

You'll need to setup ArgoCD within the operator, to do this, requires creating
an ArgoCD resource.

![Create resource](img/argoop/create-argocd-resource.png)

This should indicate that you have a deployed ArgoCD operator.

![Resource list](img/argoop/argocd-resources.png)

## Step 5

Check that the pods have come up

ArgoCD may take some time to fully come up, it has a number of pods that need
to startup.

![ArgoCD pods](img/argoop/list-pods.png)
