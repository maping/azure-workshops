# Lab 3: Deploy the Order Capture API
You need to deploy the Order Capture API (azch/captureorder). This requires an external endpoint, exposing the API on port 80 and needs to write to MongoDB.

>Hint: The Order Capture API exposes the following endpoint for health-checks: http://[PublicEndpoint]:[port]/healthz

Save the YAML below as captureorder-deployment.yaml.
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: captureorder
spec:
  selector:
      matchLabels:
        app: captureorder
  replicas: 2
  template:
      metadata:
        labels:
            app: captureorder
      spec:
        containers:
        - name: captureorder
          image: azch/captureorder
          imagePullPolicy: Always
          readinessProbe:
            httpGet:
              port: 8080
              path: /healthz
          livenessProbe:
            httpGet:
              port: 8080
              path: /healthz
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          env:
          - name: TEAMNAME
            value: "team-azch"
          - name: MONGOHOST
            value: "orders-mongo-mongodb.default.svc.cluster.local"
          - name: MONGOUSER
            value: "orders-user"
          - name: MONGOPASSWORD
            value: "orders-password"
          ports:
          - containerPort: 8080
```
And deploy it using
```console
$ kubectl apply -f captureorder-deployment.yaml
```
Save the YAML below as captureorder-service.yaml
```YAML
apiVersion: v1
kind: Service
metadata:
  name: captureorder
spec:
  selector:
    app: captureorder
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
  ```
  And deploy it using
```console
$ kubectl apply -f captureorder-service.yaml
```
Retrieve the External-IP of the Service
```console
$ kubectl get service captureorder -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
104.40.239.17
```

Ensure orders are successfully written to MongoDB
```console
$ curl -d '{"EmailAddress": "email@domain.com", "Product": "prod-1", "Total": 100}' -H "Content-Type: application/json" -X POST http://104.40.239.17/v1/order
{
  "orderId": "5c70e47fce70b300019a79a3"
}
```
