# Banque_Misr_DevOps_Engineer_Exercise


In this project, a voting app "Cats Vs Dogs" application is deployed to an Azure Kubernetes Service cluster. Existing development tools such as Docker Compose are used to locally build and test the application.

This project takes into consideration the following conditions:  

- The infrastructure (running containers) and required services provisioning as well as application deployment must be automated and can be triggered with a click on a button or a command.
- Password for database must be encrypted.
- The database must persist the data after restarts.
- The application can be scaled automatically to handle increased loads.
- Changes to the application’s code must be automatically scanned/tested before they are merged to the master branch

 
The application is Two-tier architecture consists of two layers: Client Tier (named "Azure-Vote-Front") and Database/Data Tier (named "Azure-Vote-Back).

<p align="center">
  !<img src="https://user-images.githubusercontent.com/46828049/201533073-fc0e8d63-8ba2-4bd1-a4a2-0954c1253456.png">
</p>


The Client Tier is a Python Flask app, while the Data Tier is a Redis Database.

The voting app's stack can be deployed on Docker locally with the use of docker-compose, and also on kubernetes using Github Workflows.

Requirements for local deployment:
- Install Docker
- Install Kubectl

## Deployment on Docker

To deploy the app on Docker locally, clone this repo on your machine:  

git clone [https://github.com/Azure-Samples/azure-voting-app-redis.git](https://github.com/osygroup/Banque_Misr_DevOps_Engineer_Exercise.git)

Change into the cloned directory:  

cd azure-voting-app-redis  

Then use the docker-compose.yaml file to create the container image, download the Redis image, and start the application:

docker-compose up -d  

<p align="center">
  !<img src="https://user-images.githubusercontent.com/46828049/201534627-15e5bb7a-2f63-4b68-be80-db26a312fc28.png">
</p>


Note that from the docker-compose.yaml, the redis database password is visible, which can be changed to any value of choice. This is for development scenarios only.


## Deployment on Kubernetes (AKS)

The voting app stack can be deployed on a kubernetes cluster (AKS in this case) via Github workflows pipeline. The cluster can be provisioned using Ansible. There is aan Ansible Playbook to help create an AKS cluster in the root of the repo (azure_create_aks.yml). Install Ansible on your machine and run the playbook.   

The Github workflow kubernetes_deployment.yml (in .github/workflows directory) has all the steps listed to deploy the application. The credentials needed for the workflow to connect to the cluster are added as Github Actions secrets in order not to expose them. Other sensitive data can be passed to the pipelines via Github Actions secrets. Just configure and deploy the Voting application at the click of a button!  

As seen on the workflow file:
- The application can be triggered on push to the main branch, on creation of a pull request to the main branch, and on the click of a button (Workflow Dispatch)
- The Client Tier is tested on every run on the pipeline to ensure Continuous Testing.  

The kubernetes manifest files (in the kubernetes directory) are configured to achieve scalability, database persistence, and password encryption.  

For scalability of the frontend (Azure-Vote-Front), Horizontal Pod Autoscaler resource (hpa.yml) is used to achieve this. This resource is used to adjust the number of pods in a deployment depending on CPU utilization. In the azure-vote-front deployment, the front-end container already requests 0.25 CPU, with a limit of 0.5 CPU. HPA autoscales the number of pods in the azure-vote-front deployment. If average CPU utilization across all pods exceeds 50% of their requested usage, the autoscaler increases the pods up to a maximum of 10 instances. A minimum of 3 instances is then defined for the deployment.  

For database persistence, a Persistent Vomume (PV) resource is created, and is mounted on the Redis database using Persistent Volume Claim resource. This enables the data stored in the database to be persistent, even after destruction of the redis pod. The PV already exists in the cluster before the deployment of the stack. The Persistent Volume Claim is created on deployment of the stack (in the deployment.yaml file). 

For encrypting the Redis database's password, a Secret resource is created, and is referenced in the azure-vote-back deployment. This enables the protection of the database's password as the scret is encrypted.  

### To manually deploy the voting app on a kubernetes cluster:  
Connect to your kubernetes cluster as instructed by your cluster provider.  

Create a namespace for the application:  
kubectl create namespace vote_app  

Create secret for the database password:  
kubectl create secret generic redis --from-literal="REDIS_PASSWORD=pass1234" -n vote-app  

Create a yaml file for the Redis database's Persistent Volume of 2Gb with the following content:  

apiVersion: v1  
kind: PersistentVolume  
metadata:  
&nbsp;&nbsp;name: pv-volume  
&nbsp;&nbsp;labels:  
&nbsp;&nbsp;&nbsp;&nbsp;type: local  
spec:  
&nbsp;&nbsp;storageClassName: default  
&nbsp;&nbsp;capacity:  
&nbsp;&nbsp;&nbsp;&nbsp;storage: 2Gi  
&nbsp;&nbsp;accessModes:  
&nbsp;&nbsp;&nbsp;&nbsp;- ReadWriteMany  
&nbsp;&nbsp;hostPath:  
&nbsp;&nbsp;&nbsp;&nbsp;path: "/mnt/data"  
    

Create the Persistent Volume:
kubectl apply -f PV.yaml

Deploy the voting application:
kubectl apply -f kubernetes -n vote-app

The azure-vote-front kubernetes service is a Load Balancer, which helps us to reach the application over the internet.
In a few minutes, the application is deployed and you can visit the application on a browser via the Load Balancer's public IP.

<p align="center">
  !<img src="https://user-images.githubusercontent.com/46828049/201539122-e20e3007-8121-4ee3-b710-da98a64c8e1e.jpg">
 </p>


For this project, the application can be visited via http://20.84.228.230/
