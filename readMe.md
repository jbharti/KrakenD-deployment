1.login to the cluster


2. create kubernetes secret of license file using this command 
kubectl create secret generic kranend-lickey --from-file=./LICENSE -n dev
kubectl get secret -n dev

3. Create Deployment 
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev

4. Create services
kubectl apply -f services.yaml -n dev
kubectl get service -n dev

5. Port forwarding to access URL
kubectl port-forward svc/krakend-service 8080:8000
URL - http://localhost:8080/__health

Delete Deployment:

kubectl delete -f services.yaml -n dev

kubectl delete -f deployment.yaml -n dev

kubectl delete secret kranend-lickey -n dev