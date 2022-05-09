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
We will be using [Kubernetes For Everyone](https://github.com/Wesbest/KubernetesForEveryone) course as an example of an application we want to deploy for different environments. 

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

When getting the deployments you should see 2 seperate dpeloyments
```sh
$ kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
customer1-release-k8sdemo   1/1     1            1           81m
customer2-release-k8sdemo   1/1     1            1           69m
```



For now remove everything in the `templates` directory except for the `tests` directory, the `_helpers.tpl` file and the `NOTES.txt` file.
Also empty the `values.yaml` file. We will be adding back the relevant pieces during the following excersices.

## 
