# clo835-project-manifest

## This repo contains below manifest files.
|Manifest file name | Purpose|
|- |- |
|eks_config.yaml|manifest for eks cluster deployment|
|db-secret.yaml|secret manifest for mysql credes encoded in base64 for Deployment-db.yaml|
|secrets.yaml|secret manifest for the environment variables for Deployment-app.yaml and aws creds for s3 access|
|configMap-app.yaml|configmap manifest for s3 bucket and image name information |
|service-db.yaml| Cluster service manifest for database access from port 3306 |
|service-app.yaml| Loadbalancer servcie manifest for application access over the internet |
|mysql-pvc.yaml| Persitent Volume claim manifest provisioned with default provisioner |
|Deployment-db.yaml |mysql deployment manifest|
|Deployment-app.yaml|flask app deployment manifest|
|hpa.yaml |horizontal pod scaling manifest|
|sa-clo835.yaml|service account creation manifest|
|clu-role.yaml|cluster role creation manifest|
|clu-role-bin.yaml|cluster role binding with service account manifest|         
                    
## Depoloyment steps



### 1. Create an two node eks cluster

```bash
eksctl create cluster -f eks_config.yaml
```

### 2. Install the ebs-csi-driver to provision ebs for persistent volumes 
_87656022586_ should be updated with the relevent userid 
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster clo835 --service-account-role-arn arn:aws:iam::<_87656022586_>:role/LabRole --force
```

### 3. Create the new namespace 
```bash
kubectl create ns final
```
### 4. serviceaccount and clusterrole creation + clusterrolebinding to the service account

```bash
kubectl apply -f sa-clo835.yaml -n final
```
```bash
kubectl apply -f clu-role.yaml -n final
```
```bash
kubectl apply -f clu-role-bin.yaml -n final
```

### 5. Deploy the manifests according to the above table

```bash
kubectl apply -f db-secret.yaml -n final
```
```bash
kubectl apply -f secrets.yaml -n final
```
```bash
kubectl apply -f configMap-app.yaml -n final
```
```bash
kubectl apply -f service-db.yaml -n final
```
```bash
kubectl apply -f service-app.yaml -n final
```
```bash
kubectl apply -f mysql-pvc.yaml -n final
```
```bash
kubectl apply -f Deployment-db.yaml -n final
```
```bash
kubectl apply -f Deployment-app.yaml -n final
```

### 6. HPA testing
Deploy the metric server to run healt checks
```bash
k apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.1/components.yaml
```
Verify the metric server status
```bash
k get apiservice v1beta1.metrics.k8s.io -o json | jq '.status'
```

Deploy the Horizontal pod scaling manifest and run the busy box
```bash
k apply -f hpa.yaml -n final
k run -i --tty load-generator --image=busybox /bin/sh
```
Execute an infinity loop accessing the loadbalancer URL of the flask app to increase the cpu utilization.\
Make sure to update the lodabalancer entrupoint in below command
```bash
while true; do wget -q -O - http://<_LoadBalancerServiceURL_>; done
```
On a separte terminal, monitor the scaling process.
```bash
k get hpa -n final -w
```


