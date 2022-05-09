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
helm create k8sdemo
```

Once created, open it in an editor like VS Code or your own favorite text editor or IDE.
You will see the following directory structure:
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

For now remove everything in the `templates` directory except for the `tests` directory, the `_helpers.tpl` file and the `NOTES.txt` file.
Also empty the `values.yaml` file. We will be adding back the relevant pieces during the following excersices.

## 
