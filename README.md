# High Availability for Stateful GKE Workloads
Following is the app architecture that we will deploy on the GKE cluster. 
![Wordpress Application Stack](Wordpress%20Architecture.png)
## Step 1 - setup GKE cluster
Create a **regional** GKE cluster with two node pools
- us-central1-a: Specify zone **us-central1-a**
- us-central1-b: Specify zone **us-central1-b**

Use ```e2-highcpu-8``` with only 1 node each. 
Creating node pools with these names will help you identify the actual node pool easily for a given node. 
Cluster should look like shown below.
<img width="2027" alt="regional-cluster-with-zonal-pools" src="https://user-images.githubusercontent.com/32221454/171470312-c7926315-55e8-4a22-9cee-868a21f35985.png">

## Step 2 - create storage class
We will use [Compute Engine persistent disk CSI Driver](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/gce-pd-csi-driver) to create our storage class. 

**Note** - If you want to use storage size less than 200Gi then you need to ```pd-ssd``` persistent disk, otherwise you can use ```pd-standard```.

Use following YAML for the storage class definition.
```
kubectl apply -f regional-pd-sc.yaml
```

## Step 3 - Deploy Wordpress app on Zone A
```
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
Or use K9s if you have the tool.
You can notice that the wordpress application pods are currently deployed on ```us-central1-a``` nodepool. That is because of the ```nodeAffinity``` in the deployments. 
<img width="1789" alt="wordpress-pods" src="https://user-images.githubusercontent.com/32221454/171472504-4b829107-e91f-4445-9d42-bcb01ef5ea13.png">


## Step 5 - Kill a node pool
To demonstrate failure of a zone just delete a Node Pool. Since our app is deployed on zone A we will delete that node pool. 
```
gcloud container node-pools delete us-central1-a --cluster=<Your-cluster-name>
```
Or you can delete the node pool ```us-central1-a``` from GCP console as well. 

Please note, you've to delete it from the **GKE cluster > Nodes** page. 

**_NO NOT_ directly delete the VM or VM group.**

## Step 6 - Watch the app come back up and test
```
kubectl get po -o wide -w
```
As soon as the pod status shows running you can try accessing the Wordpress app again using the same LB IP as used earlier. 


