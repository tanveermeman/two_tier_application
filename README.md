Deploying a Flask API and MySQL server on Kubernetes
This repo contains code that

Deploys a MySQL server on a Kubernetes cluster
Attaches a persistent volume to it, so the data remains contained if pods are restarting
Deploys a Flask API to add, delete and modify users in the MySQL database
Prerequisites
Have Docker and the Kubernetes CLI (kubectl) installed together with Minikube (https://kubernetes.io/docs/tasks/tools/)
Getting started
Clone the repository
Configure Docker to use the Docker daemon in your kubernetes cluster via your terminal: eval $(minikube docker-env)
Pull the latest mysql image from Dockerhub: Docker pull mysql
Build a kubernetes-api image with the Dockerfile in this repo: Docker build . -t flask-api
Secrets
Kubernetes Secrets can store and manage sensitive information. For this example we will define a password for the root user of the MySQL server using the Opaque secret type. For more info: https://kubernetes.io/docs/concepts/configuration/secret/

Encode your password in your terminal: echo -n super-secret-passwod | base64
Add the output to the flakapi-secrets.yml file at the db_root_password field
Deployments
Get the secrets, persistent volume in place and apply the deployments for the MySQL database and Flask API

Add the secrets to your kubernetes cluster: kubectl apply -f flaskapi-secrets.yml
Create the persistent volume and persistent volume claim for the database: kubectl apply -f mysql-pv.yml
Create the MySQL deployment: kubectl apply -f mysql-deployment.yml
Create the Flask API deployment: kubectl apply -f flaskapp-deployment.yml
You can check the status of the pods, services and deployments.

Creating database and schema
The API can only be used if the proper database and schemas are set. This can be done via the terminal.

Connect to your MySQL database by setting up a temporary pod as a mysql-client: kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql --host mysql --password=<super-secret-password> make sure to enter the (decoded) password specified in the flaskapi-secrets.yml
Create the database and table
CREATE DATABASE flaskapi;
USE flaskapi;
CREATE TABLE users(user_id INT PRIMARY KEY AUTO_INCREMENT, user_name VARCHAR(255), user_email VARCHAR(255), user_password VARCHAR(255));
Expose the API
The API can be accessed by exposing it using minikube: minikube service flask-service. This will return a URL. If you paste this to your browser you will see the hello world message. You can use this service_URL to make requests to the API

Start making requests
Now you can use the API to CRUD your database

add a user: curl -H "Content-Type: application/json" -d '{"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>"}' <service_URL>/create
get all users: curl <service_URL>/users
get information of a specific user: curl <service_URL>/user/<user_id>
delete a user by user_id: curl -H "Content-Type: application/json" <service_URL>/delete/<user_id>
update a user's information: curl -H "Content-Type: application/json" -d {"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>", "user_id": <user_id>} <service_URL>/update
