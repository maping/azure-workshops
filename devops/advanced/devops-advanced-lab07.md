# Lab07 Deploy Java+Maven Azure Pipeline Docker Agent on AKS

## 1. Create Java+Maven Azure Pipeline Docker Agent

## 2. Create a Azure Container Registry

## 3. Create a Azure Kubernetes Service

## 4. Push Agent Docker Image to ACR 
```console
$ ssh azureuser@104.41.165.174
$ sudo docker tag devops-agent-ubuntu-java-maven:16.04 mwitacrkoreacentral.azurecr.io/devops-agent-ubuntu-java-maven:16.04
$ sudo docker login mwitacrkoreacentral.azurecr.io -u ******************** -p ******************
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/azureuser/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
$ sudo docker push mwitacrkoreacentral.azurecr.io/devops-agent-ubuntu-java-maven:16.04
```

## 5. Create Objects in AKS
```console
$ kubectl create namespace azure-devops-agents
namespace/azure-devops-agents created
$ kubectl create secret docker-registry mwitacrkoreacentralsecret --docker-server=******************** --docker-username=******************** --docker-password=******************** -n azure-devops-agents
secret/mwitacrkoreacentralsecret created
$ kubectl create secret generic azdevops --from-literal=AZP_URL=******************** --from-literal=AZP_TOKEN=************************** --from-literal=AZP_POOL=Self-Hosted-Docker-Agent -n azure-devops-agents
secret/azdevops created
```
>Attention：secret can not cross namespace。

Authorize AKS visit ACR 
```console
$ az aks update -n Quickstart-AKS -g MWIT-Quickstart-AKS-RG --attach-acr mwitacrkoreacentral
```

## 6. Deploy Ubuntu Java Maven Docker to AKS
```console
$ vim azdevops-agent-ubuntu-java-maven-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-java-maven
  labels:
    app: azdevops-agent-ubuntu-java-maven
spec:
  replicas: 1 #here is the configuration for the actual agent always running
  selector:
    matchLabels:
      app: azdevops-agent-ubuntu-java-maven
  template:
    metadata:
      labels:
        app: azdevops-agent-ubuntu-java-maven
    spec:
      containers:
      - name: devops-agent-ubuntu-java-maven
        image: mwitacrkoreacentral.azurecr.io/devops-agent-ubuntu-java-maven:16.04
        env:
          - name: AZP_URL
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_URL
          - name: AZP_TOKEN
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_TOKEN
          - name: AZP_POOL
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_POOL
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: docker-volume
      volumes:
      - name: docker-volume
        hostPath:
          path: /var/run/docker.sock
$ kubectl apply -f azdevops-agent-ubuntu-java-maven-deployment.yml -n azure-devops-agents
deployment.apps/ubuntu-java-maven created
$ kubectl get pod -n azure-devops-agents
NAME                                 READY   STATUS    RESTARTS   AGE
ubuntu-java-maven-5df984f78b-4lth9   1/1     Running   0          2m29s
$ kubectl scale deployment ubuntu-java-maven --replicas 2 -n azure-devops-agents
```
If deploy failed, delete Deployment
```console
$ kubectl delete -f azdevops-agent-ubuntu-java-maven-deployment.yml -n azure-devops-agents
deployment.apps "ubuntu-java-maven" deleted
```
Click "Self-Hosted-Ubuntu-Agent"，Click "Agents"，you can see a ubuntu-java-maven-xxxxxxxx Online.

## 7. Test Ubuntu-Java-Maven-Docker-Agent
Create a java-helloworld-web-app Project，Create a Pipeline，Use Self-Hosted-Docker-Agent as Agent Pool，Save and Queue.

# Reference：
1. https://robertoprevato.github.io/Self-hosted-Azure-DevOps-agents-running-in-Docker/
2. https://github.com/RobertoPrevato/AzureDevOps-agents
3. https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1

