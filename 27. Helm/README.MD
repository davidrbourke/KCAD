# Helm Charts

A package manager for Kubernetes. Can group together all the resource objects for an application in Kubernetes. You don't need to deploy each resource file individually, you can deploy or apply changes in one update. 

## Features
- A single command to install the whole app, and create all the objects
- Custom values for settings/variables that can apply across all the objects
- Upgrade the application with a single command
- A single command to rollback an update, Helm keeps track of versions and which resources have changed

## Install Helm
First must have a functional cluster, and kubectl configured to connect to the cluster. Various ways to install: https://helm.sh/docs/intro/install/  

## Helm Configuration
- The kubernetes definition deployment yaml files need to be converted to Helm templates.
- The values that can be options are converted to variables.
- Values are stored in a file called Values.yaml

Together the templates and the values file form a Helm Chart:
- Templates
- Values
- Chart.yaml

Examples can be found: https://artifacthub.io/
Or via searching: `helm search hub <item to search>`  
To search other repos, you must a repository, e.g., the bitnami repo:

1. Add the repo `helm repo add bitnami https://charts.bitnami.com/bitnami`
2. Search `helm search hub <item to search>`
3. List all loaded repos that can be searched `helm repo list`

## Install Chart
```
helm install [release-name] [chart-name]

# Install from repository
helm install release-1 bitnami/wordpress

# Install from local download directory
helm install release-2 ./wordpress
```
The helm chart package is downloaded from the repository locally  
Each time you run an install, a new release is created

```
# Uninstall a helm chart
helm uninstall <release-name>

# Only download a chart, not install
helm pull --untar bitnami/wordpress

# View the downloaded items, so you can edit and install
ls wordpress
```

## Example Template (for ConfigMap)
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "some harded value"
  drink: {{ .Values.favouriteDrink }}
```

### Entry in values.yaml
```
favouriteDrink: coffee
```

### Chart.yaml
```
helm create <chart-name>

# Outputs:
apiVersion: The chart API version (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
```

## Helm Charts file structure
```
<application-name-directory>/
  Chart.yaml
  values.yaml
  charts/
  templates/
```
