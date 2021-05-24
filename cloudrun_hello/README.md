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

files structure:

----dockerfile
---- app/
	 ---- requirements.txt
	 ---- main.py


(2) create a main.py file and requirements.txt for flask
  
main.py:  

import os

from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
	name = os.environ.get("NAME", "World")
    return "Hello {}!".format(name)

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))


requirements.txt

flask
gunicorn


(3) Containerizing the flask app using dockerfile:

FROM python:3.9-slim

# Allow statements and log messages to immediately appear in the Knative logs
ENV PYTHONUNBUFFERED True

# Copy local code to the container image.
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY /app .

# Install production dependencies.
RUN pip install -r requirements.txt

# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
# Timeout is set to 0 to disable the timeouts of the workers to allow Cloud Run to handle instance scaling.
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 --timeout 0 main:app



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


