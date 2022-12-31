# argo-cd
Theory course on argocd

** What is Gitops, and where does argocd fits in? **

GitOps is an operational framework that takes DevOps best practices used for application 
development such as version control, collaboration, compliance, and CI/CD, and applies 
them to infrastructure automation. ie Gitops = IaC + MRs + CI/CD. 
MR here means merge requests (ie. change management, collaboration, PR review etc)

## Argocd

Argocd is a Gitops continuous delivery (CD) tool for Kubernetes. 
It works on pull mechanism, ie it periodically (default 3 mins) pulls K8s configs 
from a repo, and applies them changes to the K8s cluster.

**Application**: Argocd configuration is created as a yaml of type Application. 
An application defines the source to read configs from, and destination cluster and namespace.

An application has a state of in 'Sync' or 'Out of sync'.

Application source can be one of 4 types: Helm, Kustomize, Jsonnet, a directory of yaml files.

See file no 1 for sample application yaml. You can apply the yaml to create an application.
Second option is to create via argocd browser.
Third option is to create an application using CLI: 
'argocd app create app-1 --repo <http... .git>
--path guestbook --dest-server https://kubernetes.default.svc --dest -namespace default'

Check the second example yaml file in this project for an example of using Kustomize as a source.
With Kustomize, you also have the option to provide name prefix/suffix which will be
applied to all k8 resources created by argocd.

**Project**: It defines a logical grouping of Argocd Applications. If you dont define a 
project, then a default project is automatically created for your applications.

Check file no. 3 for an example of Project yaml.

You can use projects to restrict the source and destinations cluster/namespace
for the applications of the project.

You can also use 'Project Roles' to define permissions for the applications.
See file no. 4 for an example project role. 

**Sync**: The argocd process of making the actual state (state on K8s cluster) same
as desired state (as defined in the source)

**Argocd components:**

1. API/Web Server: Exposes an api for outside world interaction like UI/CLI/gRPC/REST.
2. Repo Server: Does 2 things: cloning git repos, and generating k8s manifest.
3. Application Controller: Interacts with k8s cluster api.
4. Dex: Identity service to integrate with external identity providers.
5. Redis: Used for caching.
6: ApplicationSet Controller: Automates generation of Argocd applications.
 
By default, API server is not exposed with external endpoint. So expose the api and the Argocd ui,
you do port forward: kubectl port-forward svc/argocd-server -n argocd 8080:443
Once done, you may open this url to access the argocd browser: https://localhost:8080/ 
The password to login into browser can be found by base 64 decoding secret named 
argocd-initial-admin-secret in argocd namescpace, created as part of argocd installation.
username to login would be admin.

## Argocd installation

Argocd provides 2 types of installation options:
1. Cluster admin privileges: when argoc has cluster admin access to deploy into the cluster where argocd itself is running.
2. Namespace level privileges: use this if you do not need to deploy applciations in same cluster that argocd runs it. 

You need to create a namespace for argocd: 'kubectl create ns argocd'

Then choose one of the below options:

1. Non-HA :

a. cluster-admin privileges: https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

b. namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/namespace-install.yaml


2. HA:

a. cluster-admin privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/install.yaml

b. namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/namespace-install.yaml

3. Light installation "Core"

https://github.com/argoproj/argo-cd/raw/stable/manifests/core-install.yaml

4. Helm chart: https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd


** Argocd cli Installation** instructions: https://argo-cd.readthedocs.io/en/stable/cli_installation/

Just as done for setting up browser above, do port forwarding, and login to cli using
admin user and secret pwd. To verify cli setup, run command 'argocd cluster list'




