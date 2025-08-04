
# lms docker deployment
![image](https://nordicapis.com/wp-content/uploads/Docker-API-infographic-container-devops-nordic-apis.png)

## install docker:

# LMS Minikube Deployment
![image](https://miro.medium.com/v2/resize:fit:1358/1*YpBo8EdrWiQv8LSNFGvBXg.jpeg)


## STEP-1: Launch Server
- Guide - https://minikube.sigs.k8s.io/docs/start/
- Requirements —-------t2.medium instance in AWS
- 2 CPUs or more
- 2GB of free ram memory
- 30GB of free disk space

## STEP-2: Install Softwares

### Update system
- sudo apt update
### Docker setup
- Visit: https://get.docker.com/

- curl -fsSL https://get.docker.com -o install-docker.sh
- sudo sh install-docker.sh
- sudo usermod -aG docker ubuntu
- newgrp docker


## docker network: 

- docker network create -d bridge lmsnetwork

## Database: 
- docker container run -dt --name lmsdb -p 5432:5432 --network lmsnetwork -e POSTGRES_PASSWORD=admin123 postgres

## Backend:

- cd ~/lms-app/api
- docker build -t lms-api .
- docker login -u username   ---> to store your image in docker hub
- docker push image-name     ---> pushing your image in docker hub
- docker container run -dt --name api -p 3000:3000 --network lmsnetwork -e DATABASE_URL=postgresql://postgres:admin123@pub-ip:5432/postgres -e PORT=3000 -e MODE=local lms-api
- browse backend: pub-ip:3000/api

## Frontend:

- cd ~/lms-app/webapp
- Add backend url in .env of webapp 
- docker build -t lms-web .
- docker container run -dt --name web -p 80:80 --network lmsnetwork lms-web
- docker -v 

### Kubectl setup
- Visit: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux
- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
- chmod +x kubectl
- sudo mv kubectl /usr/local/bin/kubectl
- kubectl version

### Minikube setup
- Visit: https://minikube.sigs.k8s.io/docs/start/
- curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
- sudo install minikube-linux-amd64 /usr/local/bin/minikube
- minikube version
- minikube status
- minikube start

## STEP-3: Create K8S Manifest files
- Code: git clone -b minikube https://github.com/muralialakuntla3/lms-app.git
- cd ~/lms/k8s

## STEP-4: Deploy K8S files

### Database:
- to encrypt the password
- **echo -n Qwerty123 | base64**
- **encrypted-pw**

#### Postgresql k8s deployment
- kubectl apply -f pg-secret.yml  
  - to check secret: kubectl describe secret postgres-secret
      
- kubectl apply -f pg-deployment.yml
- kubectl apply -f pg-cluster-ip.yml

### Docker login:
- generate PAT in docker hub
- My Account -> settings -> New Access Token
- login to your server
- docker login -u muralialakuntla3
- password: dckr_pat_T_****************

### Backend:
- cd ~/lms-app/api
- docker build -t muralialakuntla3/lms-api .
- docker push muralialakuntla3/lms-api
- kubectl apply -f be-configmap.yml
  - to check configmap: kubectl describe configmap backend-config-map
- kubectl apply -f be-deployment.yml
- kubectl apply -f be-service.yml

#### to check backend use port-forward cmd
- kubectl port-forward service/lms-be-service 32100:3000 --address 0.0.0.0
- check in browser: **pub-ip:32100
  
### Frontend:
#### Connect frontend with backend  : 
- cd ~/lms-app/webapp
- sudo vi .env
  - update backend-ip:32100 address

#### frontend deployment
- docker build -t muralialakuntla3/lms-web .
- docker push muralialakuntla3/lms-web
- kubectl apply -f fe-deployment.yml
- kubectl apply -f fe-service.yml

#### to check backend use port-forward cmd
- kubectl port-forward service/lms-fe-service 32200:80 --address 0.0.0.0
- check in browser: pub-ip:32200

## REACT JS v20- Presentation tier
## NODE JS v20- Application tier
## POSTGRES v16- Database
