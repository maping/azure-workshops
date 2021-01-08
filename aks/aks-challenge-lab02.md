# Lab 2: Deploy MongoDB
You need to deploy MongoDB in a way that is scalable and production ready. There are a couple of ways to do so.

>Attention: Be careful with the authentication settings when creating MongoDB. It is recommended that you create a standalone username/password and database.

>Important: If you install using Helm and then delete the release, the MongoDB data and configuration persists in a Persistent Volume Claim. You may face issues if you redploy again using the same release name because the authentication configuration will not match. If you need to delete the Helm deployment and start over, make sure you delete the Persistent Volume Claims created otherwise you’ll run into issues with authentication due to stale configuration. Find those claims using kubectl get pvc.

**Deploy an instance of MongoDB to your cluster. The application expects a database called akschallenge**

Install Helm on your developer machine
```console
$ brew install kubernetes-helm
```

If the cluster is RBAC enabled, you have to create the appropriate ServiceAccount for Tiller (the server side Helm component) to use.

Save the YAML below as helm-rbac.yaml
```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
And deploy it using
```console
$ kubectl apply -f helm-rbac.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
```
Initialize Tiller (ommit the --service-account flag if your cluster is not RBAC enabled)
```console
$ helm init --service-account tiller
Creating /Users/maping/.helm 
Creating /Users/maping/.helm/repository 
Creating /Users/maping/.helm/repository/cache 
Creating /Users/maping/.helm/repository/local 
Creating /Users/maping/.helm/plugins 
Creating /Users/maping/.helm/starters 
Creating /Users/maping/.helm/cache/archive 
Creating /Users/maping/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /Users/maping/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```
Install the MongoDB Helm chart

After you Tiller initialized in the cluster, wait for a short while then install the MongoDB chart, **then take note of the username, password and endpoints created. The command below creates a user called orders-user and a password of orders-password**
```console
$ helm install stable/mongodb --name orders-mongo --set mongodbUsername=orders-user,mongodbPassword=orders-password,mongodbDatabase=akschallenge
```
>Hint: By default, the service load balancing the MongoDB cluster would be accessible at orders-mongo-mongodb.default.svc.cluster.local

>Important: If you meet the following error: **error validating data: field spec.dataSource for v1.PersistentVolumeClaimSpec is required.** It means your are using an old helm. please use `brew update && brew upgrade kubernetes-helm` to upgrade helm client and `helm init --upgrade` to upgrade helm server.

You’ll need to use the user created in the command above when configuring the deployment environment variables.
