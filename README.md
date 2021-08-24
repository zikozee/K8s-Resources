# K8s-Resources

https://minikube.sigs.k8s.io/docs/start/

minikube start
kubectl cluster-info

kubectl run nginx --image nginx
kubectl get pods  -->>>>>>  list pods
kubectl get pods -o wide  --> provides more info
kubectl describe pod <pod-name>

kubectl delete pod <podname>

watch n 0.1 <IP>:<PORT>/hello-world

kubectl get deployment [pod] [services] --sort-by=.spec.replicas (we can sourt by any property)
starting with apiVersion.   kind, metadata., .spec.
sample .metadata.labels.app ....


================
	DELETING ALL
================
kubectl delete all -l <label_key>=label_value
kubectl delete all -l app=hello-world-rest-api

==================
	Getting Events
==================
kubectl get events --sort-by=.metadata.creationTimestamp


COMPONENTS
kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok								manages where pods are depployed and which node has what space
etcd-0               Healthy   {"health":"true"}				distributed databases storing all our config
controller-manager   Healthy   ok								this guy send commands and receive commands from "node agents"
etcd-1               Healthy   {"health":"true"}


Minikube Dashbaord
===================
minikube dashboard  ====>> opens a web browser by default
minikube dashboard --url   ===>> emits a url 

drain pods in minikube: kubectl drain --force --ignore-daemonsets minikube


			=========================
					PODS
			=========================


YAML   K8S yaml file always contains   apiVersion, kind, metadata and spec
====================================================================
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:
		app: myapp
		type: front-end
spec:
	containers
		- name: nginx-container
		  image: nginx


we can then create with:::::: kubectl create[apply] -f pod-definition.yml
							====================================
apiVersion
==========
POD  ===> v1
Service ====> v1
ReplicaSet ====> apps/v1
Deployment ====> apps/v1


kind   -->POD, Service, ReplicaSet, Deployment
====

metadata  : name, labels  (this are direct children of meetadata)  2:23 PODS with YAML
========
name == String
labels == dictionary

=============
Check LOGS
=============
kubectl logs <pod-name> -f    ====> to follow the log
kubectl logs <pod-name>  ===> one time output


kubectl get pods
kubectl describe pod <pod_name>
kubectl delete pod <pod_name>
WRITE TO OUTPUT :: kubectl get pod <pod_name> -o yaml > pod.yaml

CHECK LOADS CONSUMED BY PODS
kubectl top pods   ::::::::::::: JSON
kubectl top pods --use-protocol-buffers  ::::::::::::: protocol-buffers
				=============================	
					REPLICATION CONTROLLER
				============================
It ensures the number of replica defined is running at all time
it helps in load balancing and scaling

ReplicationController OLD Technology
ReplicaSet   new technologies


==============
HOW TO DEFINE YML file
==============

apiVersion: v1
kind: ReplicaController
metadata:							::::: for the replicationController
	name: myapp-rc
	labels:
		apps: myapp
		type: frontnd
spec:
	template:
		metadata:					::::: for the pod 
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers
				- name: nginx-container
				  image: nginx
		
	replicas: 3	
		
		
==============
HOW TO CREATE AND CHECK
==============		
	kubectl create -f <rc_file_name>.yml
	
	To check
	========
	kube get replicationController
	
		
		
		
		
							=============================	
									REPLICASET - rs
							============================
It ensures the number of pods defined is running at all time
More like it monitors and when any goes down it spins it back up again
NOTE: even if we change versions and redeploy no changes are made as rs does not worry about versions only Deployment do

HOW TO DEFINE YML file
==============

apiVersion: apps/v1				:::: if you get it wrong, you'll get error: unable to recognise "replicaset-definition.yml": no matches for/, kind=Replicaset
kind: ReplicaSet
metadata:							::::: for the replicationController
	name: myapp-rs
	labels:
		type: front-end					**match1**
spec:
	template:
		metadata:					::::: for the pod
			name: myapp-pod
			labels:
				type: front-end			:::: labels key "type" and "value" used here must be same inlabels of rs (**match1**)
		spec:
			containers:
				- name: nginx-container
				  image: nginx
		
	replicas: 3	
	selector: 							
		matchLabels:
			type: front-end				**match1**				
		
==============
HOW TO CREATE AND CHECK
==============		
	kubectl create -f <rc_file_name>.yml
	
	To check
	========
	kubectl get replicaset	
	kubectl get rs	
	
	To EDIT
	========
	kubectl edit replicaset <replicaset_name> 
	kubectl edit rs <replicaset_name> 

	Update the existing ReplicaSet and then delete all PODs, 
	so new ones with the correct image will be created.
	
	More Info
	=========
	kubectl describe replicaset	<replicaset_name>
	kubectl describe rs	<replicaset_name>
	
	To delete
	========
	kubectl delete replicaset	myapp-rs			::: Also deletes all underlying PODS
					TYPE		NAME
	kubectl delete rs	myapp-rs			::: Also deletes all underlying PODS
					TYPE		NAME
	
DIFF BTW ReplicaController and REPLICASET
========================================
::: major difference btw rs and rc ->> 
this property enables the replicaet to manage consider already created pods with labels matching
the defined in the selector

=================
	HOW TO SCALE <depending on the replicas provided, we might be scaling up or down>
=================
**** Scaling can be used as a form of restart too

1.
edit the replicas key to say from 3 to 6
replicas: 6

then run kubectl replace -f <rs_file_name>.yml

2 kubectl scale --replicas=6 -f <rs_file_name>.yml

3. kubectl scale --replicas=6 replicaset myapp-rs
								TYPE		NAME
	kubectl scale --replicas=6 rs 		myapp-rs
								TYPE	NAME	





					================
						DEPLOYMENTS
					================

deployment definition.yml file is same as that of REPLICASET
ONLY DIFFERENCE is kind: Deployment
	deploy from command line
	========
	kubectl create deployment <deployment_name> --image=image_name

	expose from commandline
	========
	kubectl expose deployment <deployment_name> --type=LoadBalancer --port=8080

	To Create
	========
	kubectl create/apply -f <deployment_name>.yml
	
	To check
	========
	kubectl get deployment
	
	kubectl get rs (THIS WILL SHOW A NEW RELICASET IN THE NAME OF THE DEPLOYMENT)	
	kubectl get pods (THIS WILL SHOW all IN THE NAME OF THE deployment-replicaset)	
	
	TO see see deployment, rs and pods all at once
	kubectl get all

	To Get file output
	========
	to get yaml output
	kubectl get deployment hello-world-rest-api -o yaml

    ::::yaml, wide, custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-as-json,jsonpath-file,name,template,templatefile
	
	WRITE TO OUTPUT :: kubectl get deployment hello-world-rest-api -o yaml > deploymentt.yaml
	
	To EDIT
	========
	kubectl edit deployment <deployment_name> 

	Update the existing ReplicaSet and then delete all PODs, 
	so new ones with the correct image will be created.

	SET IMAGE TO ANOTHER IMAGE
    ==========================
	kubectl set image deployment deployment_name container_name=new_image_name
    container_name === ddeployment_name
	
	More Info
	=========
	kubectl describe deployment <deployment_name> 
	
	To delete
	========
	kubectl delete deployments	<deployment_name>	
   kubectl delete -f a.yaml,b.yaml,c.yaml,d.yaml,eyaml


	See Difference from a new deployment to an existing deployment
	======================================
	kubectl diff -f <deployment_name>.yaml


	Quick Fix to Reduce release downtime
    =========================================
	Under spec: 
		set a parameter minReadySeconds: 45					:::: so that the pods are given a chance to start up



=====================================
	UPDATE AND ROLLBACK
=====================================
when a deployment is done, its called rollout  this is versioned
subsequents deployment versions likewise to assist in rolling back

ROLLOUT STATUS
===============
kubectl rollout status deployment/myapp-deployment
kubectl rollout status deployment.apps/myapp-deployment

HISTORY
======
kubectl rollout history deployment/<deployment_name>
kubectl rollout history deployment.apps/<deployment_name>

Deployment Strategy
===================
Recreate: completely destroying and recreating
Rolling Update(default): for every pod destroyed the newer version is deployed

UPDATE OR EDIT
==============
After changes are made
kubectl apply -f <deployment-file>.yml

THE --record option ensures that we track the command used for updating
OR
kubectl edit deployment <deployment_name> nginx  --record

OR
kubectl set image deployment/<deployment_name> nginx=nginx:1.9.1  --record (THIS WILL LEAVE THE FILE UN_UPDATED)

ROLLBACK
===========
kubectl rollout undo deployment/<deployment_name>
kubectl rollout undo deployment.apps/<deployment_name>

kubectl rollout undo deployment <deployment_name> --to-revision=2

	

	
					
					=======================
							SERVICES --svc
					=======================
	Services enable communication btw different group of pods
	Service is only required for a pod or group of pods if they will be exposed
	
	e.g group of frontend, group of back end and external datasources
Services sits between them all to enable connectivity thereby enabling loose coupling

users --->> services --->> group of front-end
group of front-end --->> services --->> backend
backend --->> services --->> database

Types of Services
======
1. NodePort :-> makes the internal pod port accessible on the node (Node houses pod(s))
2. ClusterIP :-> the services creates a virtual ip inside the cluster to enable pod in different group talk to each other
3. Loadbalancer :-> provisions a load balancer 	e.g distributing load on the frontend tiers


*** when changing from LoadBalancer to ClusterIP, delete the service first and then recreate as cluster-ip

	==========
	NODE PORT:
	==========
The service is like server in itself with the ip called the cluster ip
NodePort <------Service(10.106.1.12 (cluster ip))  80-------- Pod target port	80

	The service maps the port on the pod to its own identical port
	then routes tha port to the Port on the Node say 30008
NodePorts are in the range: 30000 - 32727


apiVersion: v1
kind: Service
metadata:						
	name: myapp-service
	labels:
		apps: myapp
		type: front-end
spec:
	type: NodePort
		name: myapp-pod
	ports:						:::: This is an array so we can have many pod mapping
		- targetPort: 80		port on the pod if omiited, value of port is used
		  port: 80			::: Mandatory  service port
		  nodePort: 30008		::: can be ommited then a free port is used
	Selector:
		app: myapp
		type: front-end


==================================
TO CONNECT THE SERVICE TO THE POD
==================================
 we link the labels used in creating the pods and map it to the selector
 
 say we have sample pod as below, we see the labels used are the same as that in the selector

apiVersion: v1
kind: Pod
metadata:						
	name: myapp-pod
	labels:
		app: myapp
		type: front-end
spec:
	containers
	 - name: nginx-pod
	   image: nginx
 
 
	To Create
	========
	kubectl create/apply -f <service_name>.yml
	
	To check
	========
	kubectl get services
	
		To check more details
	=================
	kubectl describe service <service_name> 

	To Get file output
	========
	to get yaml output
	kubectl get svc hello-world-rest-api -o yaml

    ::::yaml, wide, custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-as-json,jsonpath-file,name,template,templatefile
	
	WRITE TO OUTPUT :: kubectl get svc hello-world-rest-api -o yaml > service.yaml
	
	To Access Services on Host
	==========================
	minikube service myapp-service --url
	
AWESOME PART

Either   
 A single pod in a single node
 OR Multiple pod in a single node
 Or multiple pod in multiple nodes   which all have the same label matching that on the service,
 
 K8s creates a service the same way without any additional input from us
 even when pods are removed or scaled up, it automatically adapts



 	==========
	CLUSTERIP:
	==========
apiVersion: v1
kind: Service
metadata:						
	name: backend
	labels:
		apps: myapp
		type: frontnd
spec:
	type: ClusterIP
		name: myapp-pod
	ports:					
		- targetPort: 80		the port which the group of backend is exposed
		  port: 80			
		 
	Selector:
		app: myapp
		type: backend
 
==================================
TO CONNECT THE SERVICE TO THE BACKEND
==================================
 we link the labels used in creating the backend pods and map it to the selector
 
 say we have sample pod as below, we see the labels used are the same as that in the selector 
apiVersion: v1
kind: Pod
metadata:						
	name: myapp-pod
	labels:
		app: myapp
		type: backend
spec:
	containers
	 - name: springboot
	   image: springboot
	   
	   
	
  	=============
	LOADBALANCER:
	=============
Only supported Cloud Platforms like Google cloud, AWS or Azure we can levergage native load balancer
all we have to do is set the the type of service to LoadBalancer	
													=============
if platform is not supported even if we set it to LoadBalancer, it will work as NodePort

apiVersion: v1
kind: Service
metadata:						
	name: backend
	labels:
		apps: myapp
		type: frontnd
spec:
	type: LoadBalancer
		name: myapp-pod
	ports:					
		- targetPort: 80		the port which the group of backend is exposed
		  port: 80			
		 
	Selector:
		app: myapp
		type: backend
		
		
		
		=================
		DEPLOYING EXAMPLE VOTING APP
		=================
to check both pods and service to gether
kubectl get pods,svc


		to create all at once in the directory
		==================================
		
		kubectl create -f .

OR MULTIPLE FILE TOGETHER
   kubectl create -f a.yaml,b.yaml,c.yaml
		
	===================================================
	ON Google cloud platofrom(GCP) (Google Kubernetes EngineGKE)
	=================================================
	- click the hamburger
	- go to kubernetes engine> cluster
	- create new cluster
	- give cluster a name and do some settings if needed
	- Use clousd shell
	- and deploy
	NOTE: change user service like voting-app-service and result-app-service to type LoadBalancer 
	remove nodePort
		
		
	=============================================
		Kubernetes On AWS (EKS)
	=============================================
=============================================
Ensure you have aws cli
also kubectl
Create an IAM role withe EKS CLuster policy -- https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html
Create IAM role for Eks Node -- https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html
Create key pair -- https://us-west-2.console.aws.amazon.com/ec2/home?region=us-west-2#KeyPairs:sort=keyName
Then login to aws > search EKS
create cluster name
choos custer role created
use default vpc or create new with subnets where network interfaces will be placed
Choose Cluster endpoint access
then create ::: takes some time

	***remember to set  type in client facing service file to LoadBalancer
	and remove nodePort
	
	
	login via console:  aws eks update-kubeconfig --region us-west-2 --name example-voting-app
	=========================
	if reqwuired login with access key found in my security credentials around where username is
	then
	aws configure
	acces key id : ***
	access secret: ****
	region :  region you deployed esle it will not work
	output formaat: 
	
	
	then do kubectl get nodes
	you can clone repo here
	the run commands
	
	***remember to set  type in client facing service file to LoadBalancer
	and remove nodePort
	

	=============================================
		Kubernetes On AZURE (AKS)
	=============================================
	
	
	Login to Azure
	Search AKS
	
	Walkthrough: https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal
	
	
	ensure when using this cmd:az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
	change your resource-group and clustername
	az aks get-credentials --resource-group azure-resource1 --name example-voting-app
	
	***remember to set  type in client facing service file to LoadBalancer
	and remove nodePort
	
	
	
	
							================================
								SETTING UP KUBEADMIN
							================================
1. Multiple system or VMs
2. install docker on all nodes
3. install kubeadm on all nodes
4. initialize one node as admin
5. int PODNETWORK across all nodes
6. join worker nodes to admin node



						========================================
							Converting docker-compose to a k8s deployment config
						========================================
https://kompose.io


					=============================================
					PERSISTENT VOLUME and PERSISTENT VOLUME CLAIM
					=============================================
To access a volume in  a cluster we use persistent volume: pv
For pods to access the pv, they use the persistent volume claim pvc


					=============================================
							CONFIG MAPS iN KUBERNETES   
					=============================================
kubectl create configmap <name_of_config_map>  --from-literal=<KEY>=<VALUE>
kubectl get configmap <name_of_config_map>
kubectl describe configmap <name_of_config_map>

sample   --> kubectl edit configmap <name_of_config_map>
=======
apiVersion: v1
data:
  RDS_DB_NAME: todos				******
  RDS_HOSTNAME: mysql             ******
  RDS_PORT: "3306"				******
  RDS_USERNAME: todos-user		******
kind: ConfigMap
metadata:
creationTimestamp: "2021-08-17T17:28:43Z"
name: todo-web-application-config
namespace: default
resourceVersion: "85093"
uid: a1ce1733-adc2-46e2-98d5-fe0f5b798944

					
					=============================================
							USING SECRETS WITH KUBERNETES   
					=============================================

kubectl create secret generic todo-web-application-secrets --from-literal=RDS_PASSWORD=dummytodos



					==============================================
						SERVICE DISCOVERY IN KUBRNETES
					==============================================
we can configure a name as metadata : say currency-conversion
and we can den do CURRENCY_CONVERSION_SERVICE_HOST as kubenetes creates it automatically
but don't use this


rather we use service name as this is always constant hence we can map this as an environment variable under 
containers for services that will call it

lecture 57 KUBERNETES, step 08 3:39

hear we are mapping in currency-conversion  which calls currency-exchange
env:
- name: <ENV_VARIABLE_EXPECTED>
- balue: http://<service-name>

Hence On using Ribbon CLient for LoadBalancing,
we updated to use ribbon rather than feign clients
and we have just set the name directly in the kuberntes deployment file
name: currency-conversion
as ribbon takes care of it



				==================================
					INGRESS LOAD BALANCER
				==================================
Take for instance we have 2 or more services running for 2 or more microservices
it means we will have to create 2 or more services with load balancers (if the microservices are internet facing)
This is alot loadbalancer 
instead we can create a single LoadBalancer called ingress which will can configure its redirection based on path
This way we can convert our services from LoadBalancers to NodePorts
                          =========================================

e.g look at project 05-currency-conversion-microservce-basic
and example the ingress.yaml 
you see how the paths are cleanly configured for the currency-conversion and currency-exchange microservice


				========================================
					USING RBAC TO ALLOW ACCESS SERVICE DISCOVERY
				========================================
see 02.rbac.yaml in 06-currency-conversion-microservice-cloud
we only enabled view access

				
				==========================================
					USING SPRING CLOUD KUBERNETES CONFIG TO LOAD CONFIG MAPS
				==========================================
remember we added the dependency for k8s to the 06-currency-conversion-microservice-cloud
the application/microservice will then by default be looking configmap and secret map based on the name you set
															 ====================
as  the application name e.g currency-conversion
so we can do the below

kubectl create configmap currency-conversion --from-literal=YOUR_PROPERTY=value --from-literal=YOUR_PROPERTY_2=value2


This was tested in mysql 06-..-k8s-configmap project,
removed all the read from configmap defined in deployment file
set spring.application.name=todo-web-application
kubectl create configmap --from-literal=RDS_DB_NAME=todos --from-literal=RDS_HOSTNAME=mysql  --from-literal=RDS_PASSWORD=dummytodos   --from-literal=RDS_PORT="3306" --from-literal=RDS_USERNAME=todos-user
we can define all the property in the config map, add kubernetes config and it will be picked up automatically
this was picked up automatically
TEST THIS OUT

				
				========================
					AUTO SCALING
				========================
1 cluster autoscaling
first increase number of nodes(server)

	gcloud container clusters create example-cluster \
	--zone us-central1-a \
	--node-locations us-central1-a,us-central1-b,us-central1-f \
	--num-nodes 2 --enable-autoscaling --min-nodes 1 --max-nodes 4

2 horizontal pod auto-scaling
another option: increase number of pods on high demands:    (only if resources are available)
kubectl autoscale deployment currency-exchange-service --max=3 --min=1 --cpu-percent=70

this means auto scale deployment for currency-exchange-service to go all the way to 3pods if the cpu utilization is 
greater than 70 percent 
SAMPLE  04-.. project we reconfigured the deployment.yaml
we also copied out the format of hpa using
kubectl get hpa currency-exchange -o yaml > 01-hpa.yaml


3 vertical pod auto-scaling
increasing amount of cpu/memory allocated to pod: increasing memory of pod

    1- Available in version 1.14.7-gke.10 or higher and in 1.15.4-gke.15 or higher
   #### Enable on Cluster
	gcloud container clusters create [CLUSTER_NAME] --enable-vertical-pod-autoscaling --cluster-version=1.14.7
	gcloud container clusters update [CLUSTER-NAME] --enable-vertical-pod-autoscaling

	2 #### Configure and setup VPA
	apiVersion: autoscaling.k8s.io/v1
	kind: VerticalPodAutoscaler
	metadata:
	  name: currency-exchange-vpa
	spec:
	  targetRef:
	    apiVersion: "apps/v1"
	    kind:       Deployment
	    name:       currency-exchange
	  updatePolicy:
	    updateMode: "Off"   ###### set to Auto  to auto-scale up or auto scale down
	```

	3 #### Get Recommendations
	kubectl get vpa currency-exchange-vpa --output yaml
    scale based on recommendatation or just set to updateMode to Auto in VPA


				======================
						TRACING
                ======================
Add dependencies as shown in 07 and 08 
remember to add sleuth which gives unique id

on GCP (Google cloud platform)
enable all stack driver (except error logging)
remember naming convention is now different
logging, tracing, monitoring, stackDriver APi

Under Monioring.  right hand corner, click trace
from which you can see all traces
see trace list and monitor down to method being called in controller clas
click on any of the requests to see how it trickles down

				================
				ERROR REPORTING
				================
enabled by default
search error reporting
we can see all error occurring in the application
once we resolve we can change the status to resolved

				===============
				Logging
				===============
We can check workloads in GKE
pick a service, select container logs
pick and id and search (clear filter and just paste an id )

trace id always remain the same across all microservices
however, span id, beside it changes as it moves from one microservice to another
