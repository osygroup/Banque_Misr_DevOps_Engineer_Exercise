# Banque_Misr_DevOps_Engineer_Exercise
Submission for Banque Misr DevOps Engineer Exercise


In this project, a voting app "Cats Vs Dogs" application is deployed to an Azure Kubernetes Service cluster. Existing development tools such as Docker Compose are used to locally build and test the application.

This project takes into consideration the following:

- The infrastructure (running containers) and required services provisioning as well as application deployment must be automated and can be triggered with a click on a button or a command.
- Username and password for database must be encrypted.
- The database must persist the data after restarts.
- The application can be scaled automatically to handle increased loads.
- Changes to the application’s code must be automatically scanned/tested before they are merged to the master branch


 
The application is Two-tier architecture consists of two layers: Client Tier (named "Azure-Vote-Front") and Database/Data Tier (named "Azure-Vote-Back).

The Client Tier is a Python Flask app, while the Data Tier is a Redis Database.

The voting app's stack can be deployed on Docker locally with the use of docker-compose, and also on kubernetes using Github Workflows.

To deploy the app on Docker locally, clone this repo on your machine:

git clone [https://github.com/Azure-Samples/azure-voting-app-redis.git](https://github.com/osygroup/Banque_Misr_DevOps_Engineer_Exercise.git)

Change into the cloned directory:

cd azure-voting-app-redis

Then use the docker-compose.yaml file to create the container image, download the Redis image, and start the application:

docker-compose up -d


