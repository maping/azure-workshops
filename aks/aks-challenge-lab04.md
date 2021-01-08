# Lab 4: Deploy the frontend using Ingress
You need to deploy the Frontend (azch/frontend). This requires an external endpoint, exposing the website on port 80 and needs to write to connect to the Order Capture API public IP.

The frontend requires certain environment variables to properly run and track your progress. Make sure you set those environment variables.
```
CAPTUREORDERSERVICEIP="<public IP of order capture service>"
```

Save the YAML below as frontend-deployment.yaml
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
      matchLabels:
        app: frontend
  replicas: 1
  template:
      metadata:
        labels:
            app: frontend
      spec:
        containers:
        - name: frontend
          image: azch/frontend
          imagePullPolicy: Always
          readinessProbe:
            httpGet:
              port: 8080
              path: /
          livenessProbe:
            httpGet:
              port: 8080
              path: /
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          env:
          - name: CAPTUREORDERSERVICEIP
            value: "_PUBLIC_IP_CAPTUREORDERSERVICE_"
          ports:
          - containerPort: 8080
```
And deploy it using
```console
$ kubectl apply -f frontend-deployment.yaml
```
Verify that the pods are up and running
```console
$ kubectl get pods -l app=frontend
```
