# Skupper FTP

[![main](https://github.com/skupperproject/skewer/actions/workflows/main.yaml/badge.svg)](https://github.com/skupperproject/skewer/actions/workflows/main.yaml)

#### XXX

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Configure separate console sessions](#step-1-configure-separate-console-sessions)
* [Step 2: Log in to your clusters](#step-2-log-in-to-your-clusters)
* [Step 3: Set up your namespaces](#step-3-set-up-your-namespaces)
* [Step 4: Install Skupper in your namespaces](#step-4-install-skupper-in-your-namespaces)
* [Step 5: Check the status of your namespaces](#step-5-check-the-status-of-your-namespaces)
* [Step 6: Link your namespaces](#step-6-link-your-namespaces)
* [Step 7: Deploy the FTP service](#step-7-deploy-the-ftp-service)
* [Step 8: Expose the FTP service](#step-8-expose-the-ftp-service)
* [Step 9: Test the FTP service](#step-9-test-the-ftp-service)
* [Cleaning up](#cleaning-up)
* [Next steps](#next-steps)

## Overview

XXX

## Prerequisites

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* The `skupper` command-line tool, the latest version ([installation
  guide][install-skupper])

* Access to two Kubernetes namespaces, from any providers you choose,
  on any clusters you choose

[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[install-skupper]: https://skupper.io/start/index.html#step-1-install-the-skupper-command-line-tool-in-your-environment

## Step 1: Configure separate console sessions

Skupper is designed for use with multiple namespaces, typically on
different clusters.  The `skupper` command uses your
[kubeconfig][kubeconfig] and current context to select the namespace
where it operates.

[kubeconfig]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

Your kubeconfig is stored in a file in your home directory.  The
`skupper` and `kubectl` commands use the `KUBECONFIG` environment
variable to locate it.

A single kubeconfig supports only one active context per user.
Since you will be using multiple contexts at once in this
exercise, you need to create distinct kubeconfigs.

Start a console session for each of your namespaces.  Set the
`KUBECONFIG` environment variable to a different path in each
session.

Console for _west_:

~~~ shell
export KUBECONFIG=~/.kube/config-west
~~~

Console for _east_:

~~~ shell
export KUBECONFIG=~/.kube/config-east
~~~

## Step 2: Log in to your clusters

The methods for logging in vary by Kubernetes provider.  Find
the instructions for your chosen providers and use them to
authenticate and configure access for each console session.  See
the following links for more information:

* [Minikube](https://skupper.io/start/minikube.html#logging-in)
* [Amazon Elastic Kubernetes Service (EKS)](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)
* [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster)
* [Google Kubernetes Engine (GKE)](https://skupper.io/start/gke.html#logging-in)
* [IBM Kubernetes Service](https://skupper.io/start/ibmks.html#logging-in)
* [OpenShift](https://skupper.io/start/openshift.html#logging-in)

## Step 3: Set up your namespaces

Use `kubectl create namespace` to create the namespaces you wish to
use (or use existing namespaces).  Use `kubectl config set-context` to
set the current namespace for each session.

Console for _west_:

~~~ shell
kubectl create namespace west
kubectl config set-context --current --namespace west
~~~

Console for _east_:

~~~ shell
kubectl create namespace east
kubectl config set-context --current --namespace east
~~~

## Step 4: Install Skupper in your namespaces

The `skupper init` command installs the Skupper router and service
controller in the current namespace.  Run the `skupper init` command
in each namespace.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

**Note:** If you are using Minikube, [you need to start `minikube
tunnel`][minikube-tunnel] before you install Skupper.

Console for _west_:

~~~ shell
skupper init
~~~

Console for _east_:

~~~ shell
skupper init --ingress none
~~~

Here we are using `--ingress none` in `east` simply to make
local development with Minikube easier.  (It's tricky to run two
minikube tunnels on one host.)  The `--ingress none` option is
not required if your two namespaces are on different hosts or on
public clusters.

## Step 5: Check the status of your namespaces

Use `skupper status` in each console to check that Skupper is
installed.  As you move through the steps below, you can use `skupper
status` at any time to check your progress.

Console:

~~~ shell
skupper status
~~~

Sample output:

~~~
Skupper is enabled for namespace "west" in interior mode. It is not connected to any other sites. It has no exposed services.
The site console url is: http://10.98.13.241:8080
The credentials for internal console-auth mode are held in secret: 'skupper-console-users'
~~~

## Step 6: Link your namespaces

Creating a link requires use of two `skupper` commands in conjunction,
`skupper token create` and `skupper link create`.

The `skupper token create` command generates a secret token that
signifies permission to create a link.  The token also carries the
link details.  The `skupper link create` command then uses the link
token to create a link to the namespace that generated it.

**Note:** The link token is truly a *secret*.  Anyone who has the
token can link to your namespace.  Make sure that only those you trust
have access to it.

First, use `skupper token create` in one namespace to generate the
token.  Then, use `skupper link create` in the other to create a link.

Console for _west_:

~~~ shell
skupper token create ~/west.token
~~~

Console for _east_:

~~~ shell
skupper link create ~/west.token
skupper link status --wait 30
~~~

If your console sessions are on different machines, you may need to
use `scp` or a similar tool to transfer the token.

## Step 7: Deploy the FTP service

XXX

Console for _east_:

~~~ shell
kubectl apply -f ftp-service
~~~

## Step 8: Expose the FTP service

XXX

Console for _east_:

~~~ shell
skupper expose deployment/ftp-service --port 2020 --port 2121
~~~

## Step 9: Test the FTP service

XXX

Console for _west_:

~~~ shell
kubectl run ftp-put --rm -it --image=docker.io/curlimages/curl -- -T /etc/services ftp://example:example@ftp-service:2121
kubectl run ftp-get --rm -it --image=docker.io/curlimages/curl -- ftp://example:example@ftp-service:2121/services
~~~

## Cleaning up

To remove Skupper and the other resources from this exercise, use the
following commands.

Console for _west_:

~~~ shell
skupper delete
~~~

Console for _east_:

~~~ shell
skupper delete
kubectl delete deployment/ftp-service
~~~

## Next steps

Check out the other [examples][examples] on the Skupper website.
