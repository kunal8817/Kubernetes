# Steps for creating Dashboard

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Deploying the Dashboard UI
The Dashboard UI is not deployed by default. To deploy it, run the following command:
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```
Once you run the above command it will deply the required services under the newly created namespace "kubernetes-dashboard".
```sh
$ kubectl get ns
NAME                   STATUS   AGE
default                Active   5d16h
kube-node-lease        Active   5d16h
kube-public            Active   5d16h
kube-system            Active   5d16h
kubernetes-dashboard   Active   3d1h
```
## Accessing DashBoard
When you access Dashboard on an empty cluster, you'll see the welcome page. This page contains a link to this document as well as a button to deploy your first application. In addition, you can view which system applications are running by default in the kube-system namespace of your cluster, for example the Dashboard itself.

Now, in order to access the dashboard we can either change the service name to NodePort or you can do the port-forwarding using the kubectl.

###### Using PORT-FORWARDING Method
Now for using the port-forwarding metthod we need to check the port of the kubernetes-dashboard service. Run the below command.
```sh
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard
```
Check the entry of the TargetPort.
Once you have the TargetPort run the below command ```kubectl -n kubernetes-dashboard port-forward <your pod name> 8000:8443``` where 8000 is the port on the local host machine and the TargetPort which you grabbed from above (8443 in this case).
Now you can navigate to ```https://localhost:8000```

###### Using NodePort Method
1) We can run the dashboard by changing the service type to NodePort in the service yaml file. Use below command to edit the service file.
```kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard```
2) Change the type under the spec section from ClusterIP to NodePort.
3) Changing just the service type will work but it will take some random nodeport mostly between 30000-32767.
4) If you want to specify any particular port then you can add the port under spec>port section by the name ```nodePort: 32323```. Save the file.
5.) Run the command ```kubectl -n kubernetes-dashboard get all```. The Type should be showing as NodePort.
You can access the dashboard using ```<https://<your worker node IP>:32323```

Now you need to provide the TOKEN for accessing the kubernetes Dashboard.
Use below steps to get the TOKEN.
1) Run the command ```kubectl -n kubernetes-dashboard get svc```.
2) Describe the service account for grabbing the Token secret. ```kubectl -n kubernetes-dashboard describe sa kubernetes-dashboard```
3) This TOKEN secret wil actuall have the actual token to access the dashboard.
 ```kubectl -n kubernetes-dashboard describe secret <paste the secret from above command>```.
4) Copy the TOKEN and paste it in the Dashboard. You will be able to login to the dashboard but you won't be able to see anything. So thats why we need to create the service account and then assign a cluster-role.
5) Now you need to create an service account yaml file with the cluster-admin role binding.
6) You can create a service account yaml file from here.
```https://github.com/kunal8817/Kubernetes/blob/main/Dashboard/sa_cluster_admin.yaml```
7) Apply the yaml file using command ```kubectl apply -f sa_cluster_admin.yaml```
8) The service account and cluster role binding will be created in kube-system namespace.
9) Run the command ```kubectl -n kube-system get sa``` to get the list of all the service account.
10) Now describe the sa to get the TOKEN ID ```kubectl -n kube-system describe sa dashboard-admin```
11) Copy the TOKEN secret.
12) Describe the above secret to get the TOKEN. Run the command ```kubectl -n kube-system describe secret <paste the secret from above command>```
13) Once you use this TOKEN to login to your dashboard, you should be able to see the services, Namespaces and all the other workloads.
