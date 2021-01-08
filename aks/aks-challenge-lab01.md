# Lab 1: Deploy Kubernetes with Azure Kubernetes Service

Get the latest available Kubernetes version
```console
$ az aks get-versions -l eastus -o table
KubernetesVersion    Upgrades
-------------------  -----------------------
1.12.5               None available
1.12.4               1.12.5
1.11.7               1.12.4, 1.12.5
1.11.6               1.11.7, 1.12.4, 1.12.5
1.10.12              1.11.6, 1.11.7
1.10.9               1.10.12, 1.11.6, 1.11.7
1.9.11               1.10.9, 1.10.12
1.9.10               1.9.11, 1.10.9, 1.10.12
```

Create a Resource Group
```console
$ az group create --name akschallenge --location eastus
```

Create AKS using the latest version and enable the monitoring addon
```console
$ az aks create --resource-group akschallenge --name <unique-aks-cluster-name> --enable-addons monitoring --kubernetes-version 1.12.5 --generate-ssh-keys --location eastus
```
>Important: If you are using Service Principal authentication, for example in a lab environment, you’ll need to use an alternate command to create the cluster with your existing Service Principal passing in the Application Id and the Application Secret Key.
```console
$ az aks create --resource-group akschallenge --name <unique-aks-cluster-name> --enable-addons monitoring --kubernetes-version 1.12.5 --generate-ssh-keys --location eastus --service-principal APP_ID --client-secret "APP_SECRET"
```
Install the Kubernetes CLI
```console
$ az aks install-cli
```

Authenticate
```console
$ az aks get-credentials --resource-group akschallenge --name <unique-aks-cluster-name>
```
List the available nodes
```console
$ kubectl get nodes
```
