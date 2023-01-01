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
For example, you can specify the allowed roles, and leave out application deletion permission.
See file no. 4 for an example project role. 
To work with project roles, you will also need to create JWT tokens using argocd command.

**Sync**: The argocd process of making the actual state (state on K8s cluster) same
as desired state (as defined in the source). Only out of sync applications are synced back.
You can enable automated sync to true. But this does not delete a resource removed at source.
For this, enable another sync property => prune = true.
There is another sync option called selfHeal. If set to true, any changes made to live 
cluster will trigger the sync process to bring cluster back to desired state.
You can set these properties in application yaml, CLI, or via browser.

Sync provides us hooks like PreSync, PostSync, SyncFail. For eg, we can use PostSync to send notifications.
Sync waves are further used to order the manifest files within one stage of sync operation.
-ve wave values are also allowed. Lower wave value gets executed first.

An example of an application that deploys Sync wave jobs:
https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/sync-phases-waves/application%20-%20sync%20waves.yaml

The waves that above uses:
https://github.com/mabusaa/argocd-example-apps/blob/master/sync-waves/manifests.yaml

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

**Private repo as source**

https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/#github-app-credential

This also covers 'Credential templates', to re-use credentials across repos if they share common pwd.

**Diffing customisation**

You can choose to ignore certain resources from being synced.
You can use JSON path (pointers), JQ path expressions, or sepcific managers in metadata.managedFields.
A good use case when to use diffing customisation is when you are using istio, to ignore clientConfig: caBundle.

**Destination Cluster**

Argocd can by default deploy to same cluster where it itself is hosted.
But to deploy to remote cluster, the remote cluster information
is exposed via a Secret, which has a special label of 'argocd.argoproj.io/secret-type:cluster'
Cluster data required: name, server (host) url, config (authN details), and namespaces in the cluster.

Eg. of an application pointing to a remote cluster: https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20remote%20cluster.yaml

Secret with cluster details: https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/clusters/staging-digitalocean.yaml

**App of apps patter**

Managing argocd applications using kubectl can become cumbersome. App of apps patter helps here, 
as only the root argocd application is deployed using kubectl, and this app has a source = directory path.
All new applications are then kept at the path specified, and they then get provisioned automatically.

Eg. of app of apps: https://github.com/mabusaa/argocd-course-app-of-apps


**Application Set**

It is a controller and a CRD that automates argocd application creation. 

ApplicationSets use 'Geberators' to generate parameters of application template. 
Types of generators:
1. List generator
2. Cluster generator
3. Git generator
4. Matrix generator
5. Merge generator
6. SCM provider generator
7. Pull request generator
8. Cluster decision resource generator

See sample file no. 6 for an example application set using a list generator.

Example of cluster generator:

https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/application%20set/cluster-generator-matching-labels.yaml

https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/application%20set/cluster-generator-all.yaml

Git directory generator: https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/application%20set/git-directory-generator.yaml

Matrix generator: https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/application%20set/matrix-generator.yaml

Pull request generator: https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/application%20set/pull-request-generator.yaml

