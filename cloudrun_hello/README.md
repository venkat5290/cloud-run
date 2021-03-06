######## CLOUD RUN #########

Deploying a simple python flsk hello world app to cloud run

Pre requisites :

export PROJECT_ID=$(gcloud config get-value project)

echo $PROJECT_ID

gcloud config set run/region asia-south1
  
Enable the required APIS

(1) Create a directory and change directory into it

mkdir cloudrun_demo

cd cloudrun-demo

directory structure of app can be found in the image.

(2) create a main.py file and requirements.txt for flask app
  
(3) Containerizing the flask app using dockerfile:

PORT varibale in cloud run:

The container must listen for requests on 0.0.0.0 on the port to which requests are sent.  
By default, requests are sent to 8080, but you can configure Cloud Run to send requests to the port of your choice.  
Cloud Run injects the PORT environment variable into the container.  
Inside Cloud Run container instances, the value of the PORT environment variable always reflects the port to which requests are sent. It defaults to 8080.


(4) Building the container

gcloud builds submit --tag gcr.io/${PROJECT_ID}/cloudrun_demo 

gcloud builds list


(5) Deploying to cloud run:

paramerters : allow-unauthenticated

region - us-central1

gcloud run deploy --image gcr.io/${PROJECT_ID}/cloudrun_demo --platform managed --allow-unauthenticated  --region us-central1 


input prompts:

Service name: hello


Finally we get a servcie url  
Open the service url and Hello World appears on the browser


After some modifications:

Built a new image:

gcloud builds submit --tag gcr.io/${PROJECT_ID}/cloudrun_demo 

Deploy the container to a service

gcloud run deploy --image gcr.io/${PROJECT_ID}/cloudrun_demo --platform managed --allow-unauthenticated  --region us-central1


gcloud config set run/region us-central1

 gcloud run services list --platform managed

gcloud run revisions list --service hello-run

gcloud run revisions describe REVISION


Testing:

i=10
 while [ $i -ge 1 ]; do curl https://hello-run-si3dcgkqha-uc.a.run.app/;sleep 1;((i=i-1));done


clean up:

delete the cloud run service

 gcloud run services delete hello-run --platform managed


delete the container images from registry :

gcloud container images list

gcloud container images delete gcr.io/neat-vent-281306/cloudrun_demo ->  delets the latest tag
gcloud container images delete gcr.io/neat-vent-281306/cloudrun_demo:v2 --force-delete-tags ==>  delets the version v1


