# High Availability for Stateful GKE Workloads
## Step 1 - setup GKE cluster
Create a **regional** GKE cluster with two node pools
- us-central1-a: Specify zone **us-central1-a**
- us-central1-b: Specify zone **us-central1-b**

Use ```e2-highcpu-8``` with only 1 node each. 
Creating node pools with these names will help you identify the actual node pool easily for a given node. 

## Step 2 - create storage class
We will use Compute Engine persistent disk CSI Driver to create our storage class. 

**Note** - If you want to use storage size less than 200Gi then you need to ```pd-ssd``` persistent disk, otherwise you can use ```pd-standard```.

Use following YAML for the storage class definition.
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: regionalpd-storageclass
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-standard
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.gke.io/zone
    values:
    - us-central1-a
    - us-central1-b
```

## Step 3 - Deploy Wordpress app on Zone A
```
cd wordpress
kubectl apply -k ./
```

## Step 4 - Access the Wordperss app
```
kubectl get svc -w
```
Use the LB IP for wordpress service and access the Wordpress app. Setup your user and add a blog post.
In another window keep checking the status of the pods. 
```
kubectl get po -o wide -w
```

## Step 5 - Kill a node pool
To demonstrate failure of a zone just delete a Node Pool. Since our app is deployed on zone A we will delete that node pool. 
```
gcloud container node-pools delete us-central1-a --cluster=<Your-cluster-name>
```
Or you can delete the node pool ```us-central1-a``` from GCP console as well. 
Please note, you've to delete it from the GKE cluster > Nodes page. Don't directly delete the VM or VM group. 

## Step 6 - Watch the app come back up and test
```
kubectl get po -o wide -w
```
As soon as the pod status shows running you can try accessing the Wordpress app again using the same LB IP as used earlier. 


