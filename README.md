# Helm

## Prerequisites
1. To be able to install helm charts we need a kubernetes cluster we can install these charts on.
Make sure you've installed the latest version of [Minikube](https://minikube.sigs.k8s.io/docs/start/).
1. Afterwards use your OS' package manager to [install Helm](https://helm.sh/docs/intro/install/#through-package-managers).
1. We will be creating resources on your local filesystem so you might want to reserve a directory somewhere to use as a working directory during the exercises.

## What is Helm

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



For now remove everything in the `templates` directory except for the `tests` directory, the `_helpers.tpl` file and the `NOTES.txt` file.
Also empty the `values.yaml` file. We will be adding back the relevant pieces during the following excersices.

## 
