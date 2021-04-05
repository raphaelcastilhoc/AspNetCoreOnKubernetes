BUILD DOCKER IMAGE

docker build -t aspnet-core-on-kubernetes-storage .

docker tag aspnet-core-on-kubernetes-storage raphaelcastilhoc/aspnet-core-on-kubernetes-storage

docker push raphaelcastilhoc/aspnet-core-on-kubernetes-storage:latest



BUILD KUBERNETES RESOURCES

kubectl apply -f kubernetes_api.yaml



ACCESS KUBERNETES DASHBOARD

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

kubectl proxy

(use this command to get token authentication)
kubectl describe secret -n kube-system


http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#!/login




CONFIG SCALING

We can play around with scaling out our pods using the Horizontal Pod Autoscaler (HPA). Horizontal Pod Autoscaler automatically
scales the number of pods based on CPU utilization.

To create an HPA, we first need to enable the K8s metrics server. HPA scales pods based on pod resource metrics retrieved from the resource metrics API.
The metrics API is provided by the metrics server. We can deploy the metrics server using the following command.


kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.1/components.yaml


For it to work locally, we also need to add --kubelet-insecure-tls to the args for the containers spec.
To edit the the spec, we need to run this command:


kubectl edit deploy -n kube-system metrics-server


Under spec.template.spec.containers, on the same level as name: metrics-server add
--kubelet-insecure-tls


The following command config the auto-scaling:


kubectl autoscale deployment aspnet-core-on-kubernetes-storage --cpu-percent=10 --min=2 --max=10


We can see the scaling on kubernetes dashboard or on docker dashboard


