# Deployment in EKS Cluster

### Prerequisite
Create folder(Ex. Kranend), In which following deployment and license file should be present
- deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: krakend-deployment
spec:
  selector:
    matchLabels:
      app: krakend
  replicas: 2
  template:
    metadata:
      labels:
        app: krakend
        app.kubernetes.io/environment: dev
    spec:
      containers:
      - name: krakend
		image: 01123456789.dkr.ecr.us-east-1.amazonaws.com/krakend-ee:2.0.0        
        ports:
        - containerPort: 8080
        imagePullPolicy: IfNotPresent
        command: [ "/usr/bin/krakend" ]
        args: [ "run", "-d", "-c", "/etc/krakend/krakend.json","--accept-eula", "-p", "8080" ]
        env:
        - name: KRAKEND_PORT
          value: "8080"
        volumeMounts:
        - name: kranend-lickey
          mountPath: "/etc/krakend/LICENSE"
          subPath: LICENSE
      volumes:
      - name: kranend-lickey
        secret:
          secretName: kranend-lickey
```
- services.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: krakend-service
spec:
  type: NodePort
  ports:
  - name: http
    port: 8000
    targetPort: 8080
    protocol: TCP
  selector:
    app: krakend
```
- krakend.json
```
{
    "$schema": "http://www.krakend.io/schema/v3.json",
    "version": 3
}
```
- LICENSE

Lisence key should be there in this file. for more details Please read the [documentation](https://www.krakend.io/docs/enterprise/overview/introduction/) file

### Deployment steps
It is deployed in dev namespace of switalk-dev cluster. The steps are following.

1. login to the cluster
```
aws eks --region us-east-1  update-kubeconfig --name dev
```
2. create kubernetes secret of license file using this command
```
kubectl create secret generic kranend-lickey --from-file=./LICENSE -n dev
kubectl get secret -n dev
```

3. Create Deployment
```
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev
```

4. Create services
```
kubectl apply -f services.yaml -n dev
kubectl get service -n dev
```

5. Port forwarding to access URL
```
kubectl port-forward svc/krakend-service 8080:8000
URL - http://localhost:8080/__health
```

### Delete Deployment
```sh
kubectl delete -f services.yaml -n dev

kubectl delete -f deployment.yaml -n dev

kubectl delete secret kranend-lickey -n dev
```