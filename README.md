Steps:

Github --->AWS-->Code pipeline (Code build+code deploy)-->install kubernaties cluster -->loadbalancer.


Steps:

1. Docker file.
2. Build spec.yaml
3. code pipeline - Code build + code deploy


--------------------------------------------------

1. Docker file 

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY dist/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


--------------------------------------------------
docker run -itd -p 3000:80 imagename


http://13.235.19.235:8080/	

Developer → GitHub → Jenkins → Build → Test → Push Image → Deploy to Kubernetes

Pushing my docker file into Github 

1. My docker file is in server
2. I am using ebelow commands to push my docker file in docker hub
1.I set my repository by using below command
git remote set-url origin https://github.com/subbasri17/Brain-Tasks-App.git 
git remote -v
git config -l
3.git add .
4.git commit -m "This is my first commit"
5.git pull origin main --rebase
6. git push origin main




---------------------------------------------------------------------------------------------------------------------------------------------------------
Code Build and code deploy screenshots attached.
Steps followed:
STEP 1: Create Ubuntu Linux server
STEP 2:
sudo apt update sudo apt install ruby-full sudo apt install wget
Wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
systemctl status codedeploy-agent
STEP 3: Create role for EC2
Go to aws IAM roles
Create Roles
Use case select EC2
Choose policy ec2roleforcodedeploy Codedeployfullaccess
Then attach role to ec2 go ec2 —-> security —-> modifyIAM
STEP 4 : Create Service role for Deploy stage
Select use case codedeploy
Once role created attach below permision
STEP 5:
Go codepipeline
Create Pipeline
Name
Create Git connection choose your repo and branch:

Next Create Build stage choose AWS build
Under env choose appropriate Image ubuntu or amazon linux
Create project mention path buildspec.yaml
Next Skip the test stage
Next Choose AWS deploy provider
Create application
Create deploymentgroup in that attach role for code deploy which you created above
Then save it
Trigger Pipilne check the logs if its failing or not.
---------------------------------------------------------------------------------------------------------------------------------------------------------


=-----------------------------------------------------------------------------------------------
eks-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-tasks-app
  labels:
    app: brain-tasks-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: brain-tasks-app
  template:
    metadata:
      labels:
        app: brain-tasks-app
    spec:
      containers:
        - name: webserver
          image: aarushisuba/webserver1:latest
          ports:
            - containerPort: 80
			
=-----------------------------------------------------------------------------------------------
service.yaml

apiVersion: v1
kind: Service
metadata:
  name: brain-tasks-service
spec:
  type: LoadBalancer
  selector:
    app: brain-tasks-app
  ports:
    - port: 80
      targetPort: 80


------------------------------------------------------------------------------------------------------
Dockerfile
build spec
deployment.yaml
service.yaml
dist/ 
------------------------------------------------------------------------------------------------------

buildspec.yaml

version: 0.2

env:
  variables:
    IMAGE_NAME: aarushisuba/webserver1
    AWS_DEFAULT_REGION: ap-south-1
    EKS_CLUSTER: brain-cluster

phases:

  install:
    commands:
      - echo Installing kubectl
      - curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.27.1/2023-09-14/bin/linux/amd64/kubectl
      - chmod +x kubectl
      - mv kubectl /usr/local/bin/

  pre_build:
    commands:
      - echo Login to DockerHub
      - echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

      - echo Updating kubeconfig
      - aws eks --region $AWS_DEFAULT_REGION update-kubeconfig --name $EKS_CLUSTER

      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)

  build:
    commands:
      - echo Build started on `date`
      - docker build -t $IMAGE_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest

  post_build:
    commands:
      - echo Pushing Docker images
      - docker push $IMAGE_NAME:$IMAGE_TAG
      - docker push $IMAGE_NAME:latest

      - echo Deploying to EKS
      - kubectl apply -f eks-deployment.yaml
      - kubectl apply -f service.yaml

artifacts:
  files:
    - '**/*'

	
-------------------------------------------------------------------------------------------------------
Cloud watch :

1. SNS created -Email notification service.
2.Cloud watch metrics created for instance. 
3.Set up threshold and enabled notification.

---------------------------------------------------------------------
