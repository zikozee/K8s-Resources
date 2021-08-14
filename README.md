# K8s-Specification

https://minikube.sigs.k8s.io/docs/start/

minikube start
kubectl cluster-info

kubectl run nginx --image nginx
kubectl get pods  -->>>>>>  list pods
kubectl get pods -o wide  --> provides more info
kubectl describe pod <pod-name>

kubectl delete pod <podname>


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
It ensures the number of replica defined is running at all time
More like it monitors and when any goes down it spins it back up again

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
	
	To EDIT
	========
	kubectl edit deployment <deployment_name> 

	Update the existing ReplicaSet and then delete all PODs, 
	so new ones with the correct image will be created.
	
	More Info
	=========
	kubectl describe deployment <deployment_name> 
	
	To delete
	========
	kubectl delete deployments	<deployment_name>			


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
3. Loadbalancer :-> porvisions a load balancer 	e.g distributing load on the frontend tiers



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
		type: frontnd
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
	name: mmyapp-pod
	labels:
		app: myapp
		type: frontnd
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
	
	To Access Services on Host
	==========================
	minikube service myapp-service --url
	
AWESOME PART

Either   
 A single port on a single node
 OR Multiple pod on a single node
 Or multiple pod on multiple nodes   which all have the same label matching that on the service,
 
 K8s creates a service the same way without any additional input from us
 even when pods are removed or extras created, it automatically adapts



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
Only supported Cloud Platforms like Google cloud, AWS or Azurewe can levergage native load balancer
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