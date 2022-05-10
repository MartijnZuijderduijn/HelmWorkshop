# Helm

## Prerequisites
1. To be able to install helm charts we need a kubernetes cluster we can install these charts on.
Make sure you've installed the latest version of [Minikube](https://minikube.sigs.k8s.io/docs/start/).
1. Afterwards use your OS' package manager to [install Helm](https://helm.sh/docs/intro/install/#through-package-managers).
1. We will be creating resources on your local filesystem so you might want to reserve a directory somewhere to use as a working directory during the exercises.

## What is Helm
Helm is three things. It is a Packaging system, Configuration management tool and Templating engine for Kubernetes manifest files.

If Kubernetes were an operating system, Helm would be the package manager. Ubuntu uses apt, CentOS uses yum, and Kubernetes uses Helm. If kubernetes were a website, Helm would be the template engine not unlike you can use ASP.net to [template html pages](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0&tabs=visual-studio#the-home-page). 

Helm makes it easy to manage different configurations for an application or to package your own set of kubernetes manifests and make it available to other teams or use it as a build artifact.

### Why do I need helm
Kubernetes objects are challenging to manage. With helpful tools, the Kubernetes learning curve becomes smooth and manageable. Helm automates maintenance of YAML manifests for Kubernetes objects by packaging information into charts and advertises them to a Kubernetes cluster.

Helm keeps track of the versioned history of every chart installation and change. Rolling back to a previous version or upgrading to a newer version is completed with comprehensible commands.

### How does Helm work?
![how does Helm work](/images/helm-overview.png)

Helm has a number of moving pieces:
- Go templates for yaml files

To make the configuration of a set of Kubernetes manifests dynamic, Helm has a set of [Go Templates](https://blog.gopheracademy.com/advent-2017/using-go-templates/). You can create flags, loops and other templating functions to dynamically build up your kubernetes manifests.

- A Package containing Go templates and a set of default values

Helm bundles a set of templates together inside a package called a Chart. Alongside the templates a Chart also contains a `values.yaml` file that contain the default values the chart should be installed with and a `Chart.yaml` containing some information about the chart like the name, version and maybe a list of dependend charts this chart needs.

- Optionally a Chart Repository

Helm is a packaging tool and you can package and push/pull your charts to and from a chart repository. 
Helm's equivalent is [Artifact Hub](https://artifacthub.io/packages/search?kind=0) but you can also host yourt own repository.
You do not actually have to use this feature. Helm is perfectly capable in deploying your local "unpackaged" charts. 
It becomes relevant if you want to use publicly available charts, use these as a dependency for ypour own chart or want to distribute your own chart to other teams.

- A CLI Tool used to interact with charts and kubernetes clusters

Helm comes with a CLI that is used to interact with charts by creating & validating Charts, managing Chart repositories and pulling from them and installing, upgrading and potentially rolling back of a so called release of a chart on a target kubernetes cluster. They way Helm installs a Chart on a cluster is using the configuration file of kubernetes normally located at `~/.kube/config`. It takes the current context if not specified, parses the templates in the chart using the default `values.yaml` and any other override `.yaml` files and applying the resulting plain yaml manifests on the Kubernetes api. 

- A Release inside your kubernetes cluster

The result of a Helm install is a Release. You can see it as a singular instance of your Helm Chart. These are all the plain Kubernetes resources that are running inside your cluster like Deployments, Services, Ingresses or any other Kubernetes resource. To be able to track the status of the release, Helm also installs some meta information about the Release inside you cluster in the form a Kubernetes Secret. This secret contains information about each individual install and upgrade event that occured. This enables Helm to rollback to previous revisions of the Release.

## Creating a Helm chart
Let's start exploring helm!

Create a new helm chart by running the following command:
```sh
$ helm create k8sdemo
Creating k8sdemo
```

Once created, open it in an editor like VS Code or your own favorite text editor or IDE.
You will see the following directory structure of the example project just created by Helm:
```
k8sdemo/
├── .helmignore       # Contains patterns to ignore when packaging Helm charts.
├── Chart.yaml        # Information about your chart
├── values.yaml       # The default values for your templates
├── charts/           # Charts that this chart depends on
└── templates/        # The template files
    ├── tests/        # The test files
    ├── _helpers.tpl  # helper template functions to capture common templating solutions for instance generating labels
    └── NOTES.txt     # Information to display on a successful install  
```

## Installing a Helm chart
Helm has created a boilerplate chart for you. We can use that boilerplate to explorer how to install a helm chart.
Install the boilerplate k8sdemo chart by running the following command. Pretend this is an application deployment for a specific customer.

```sh
$ helm install customer1-release ./k8sdemo
NAME: customer1-release
LAST DEPLOYED: Mon May  9 11:54:42 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=k8sdemo,app.kubernetes.io/instance=customer1-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

We can now use normal `kubectl` commands to explorer what has been deployed by the Helm chart.
```sh
$ kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
customer1-release-k8sdemo   1/1     1            1           6m47s

$ kubectl get services
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
customer1-release-k8sdemo   ClusterIP   10.96.142.170   <none>        80/TCP    7m53s
```

One of the powers of helm is that you can make isolated deployments using the same set of kubernetes manifests. 
We can use the boilerplate chart to also install a deployment for an imaginary customer2. Lets use another command this time. One that can easiliy be repeated in a CI/CD pipeline:
```sh
$ helm upgrade --install customer2-release ./k8sdemo
Release "customer2-release" does not exist. Installing it now.
NAME: customer2-release
LAST DEPLOYED: Mon May  9 12:05:58 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=k8sdemo,app.kubernetes.io/instance=customer2-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

When getting the deployments you should see 2 seperate deployments
```sh
$ kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
customer1-release-k8sdemo   1/1     1            1           81m
customer2-release-k8sdemo   1/1     1            1           69m
```

Let's imagine we have third customer that want's to use our app but they have some scaling requirements.
We can use the same boilerplate chart to provide a different number of replicas for the deployment.
We can quickly inspect the values we can pass to the chart by running:
```sh
$ helm show values ./k8sdemo
# Default values for k8sdemo.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1
...
```

as you can see we can supply a value for the number of replicas.
Lets create a new release for our thirs customer and supply the desired amount of replicas.

```sh
$ helm upgrade --install customer3-release ./k8sdemo --set replicaCount=3 
Release "customer3-release" does not exist. Installing it now.

NAME: customer3-release
...
```

if we get the pods...
```sh
$ kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
customer1-release-k8sdemo-5db97d748c-dsz6m   1/1     Running   0          4h34m
customer2-release-k8sdemo-667665f4b4-vn74x   1/1     Running   0          4h23m
customer3-release-k8sdemo-568597d6f-9frjm    1/1     Running   0          63s
customer3-release-k8sdemo-568597d6f-9sm2c    1/1     Running   0          63s
customer3-release-k8sdemo-568597d6f-pcz4j    1/1     Running   0          63s
```

we see we got the desired results without having to change any Kubernetes manifest files!

Clean up this excersice by deleting all releases
```sh
$ helm delete customer1-release
$ helm delete customer2-release
$ helm delete customer3-release
```

## Upgrading a Helm Chart 
Applications are dynamic things in code but also the way they are deployed. 
Create a Release using the k8sdemo chart using the name "customer-api-prod" using the defaults.
To view the status of a release run
```sh
$ helm status customer-api-prod
```

After a while we find out that we have some changing requirements for our application.
We want to have some redundancy for our pod so we have to worry less about outages of our application also we would like to have developers to ingress to our customer api application. Maybe we also want to specify the desired resources for our deployment so Kubernetes is able to schedule our pods better.

Luckily our chart has all the dynamic values to be able to support all these requirements.
If you inpect the `values.yaml` you can see that there are loads of options we can pass to our templates.

for now we are interested in the following configuration:
```yaml
replicaCount: 1
...

ingress:
  enabled: false
...

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
```

create a new `values.prod.yaml` file with the following content inthe `k8sdemo\` directory

```yaml
replicaCount: 2

ingress:
  enabled: true

resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

upgrade the release by running
```sh
$ helm upgrade --install customer-api-prod ./k8sdemo -f ./k8sdemo/values.prod.yaml
```

take your time to explore the changes in kubernetes using `kubectl`

if you run 
```sh
$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
customer-api-prod       default         2               2022-05-10 09:01:53.306922 +0200 CEST   deployed        k8sdemo-0.1.0   1.16.0 
```

you can see that we are now ar revision 2.
another way to view the revision is by running:
```
$ helm history customer-api-prod
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Mon May  9 16:51:32 2022        superseded      k8sdemo-0.1.0   1.16.0          Install complete
2               Tue May 10 09:01:53 2022        deployed        k8sdemo-0.1.0   1.16.0          Upgrade complete
```

clean up by deleting the release

## Building our own helm chart
Up until now we have been using the default boilerblate chart to explore the basics of installing and upgrading releases. 
In this exercise we will be exploring building our worn chart using the GO templating language.   

First remove everything in the `templates` directory except for the `tests` directory, the `_helpers.tpl` file and the `NOTES.txt` file.
Also empty the `values.yaml` file.

