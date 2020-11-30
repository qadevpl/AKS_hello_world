# AKS_hello_world
Status of develop app:
- [x] 1. Create a Windows Container Image including IIS [Dockerfile], details: Dockerfile https://github.com/qadevpl/AKS_hello_world/Dockerfile,
- command to build: **docker build -t helloworld .**
- command to run: **docker run -it --rm --name  helloworldapp helloworld**
- [x] 2. Including a hello world .NET application into IIS (can be part of the container), details: **app code base on .net core web app:** **https://github.com/qadevpl/AKS_hello_world/tree/main/aspnetapp**
- [x] 3. Push Container Image to App Registry, details: link ARM template script building ACR component: **https://github.com/qadevpl/AKS_hello_world/tree/main/infra/ARM_template/ACR**
- [x] 4. Deploy AKS, details: link ARM template script building AKS component: **https://github.com/qadevpl/AKS_hello_world/tree/main/infra/ARM_template/AKS**
- **AKS include 2 type of node linux and windows**
- **AKS include configuration Azure CNI neccessery to build 2 type of nodes**
- [x] 5. Deploy Container to AKS, details: link to deployment yaml file: **https://github.com/qadevpl/AKS_hello_world/blob/main/helloworld_apps_deployment.yaml**
- ***deploy command:***
- **az acr login --name acru5jb5ydbxa2ee**
- **docker login acru5jb5ydbxa2ee.azurecr.io**
- **docker tag helloworld acru5jb5ydbxa2ee.azurecr.io/helloworld**
- **docker push acru5jb5ydbxa2ee.azurecr.io/helloworld**
- ### Run container in local laptop:
- **docker pull acru5jb5ydbxa2ee.azurecr.io/helloworld**
- **docker run -it --rm -p 8080:80 acru5jb5ydbxa2ee.azurecr.io/helloworld http://localhost:8080/**
- ### Run container in AKS:
- **az aks get-credentials --resource-group omada_example_apps --name omadacluster -a**
- **kubectl apply -n apps -f .\helloworld_apps_deployment.yaml**
- [ ] 6. Make IIS application available through reverse proxy
- [ ] 7. Open web application in browser (that can be manual)


## PROBLEM with step 6. and 7.
After deploy core web app on AKS cluster, created pod return error: 
> **Failed to pull image "acru5jb5ydbxa2ee.azurecr.io/helloworld": [rpc error: code = Unknown desc = a Windows version 10.0.19041-based image is incompatible with a 10.0.17763 host, rpc error: code = Unknown desc = Error response from daemon: Get https://acru5jb5ydbxa2ee.azurecr.io/v2/helloworld/manifests/latest: unauthorized: authentication required, visit https://aka.ms/acr/authorization for more information.]**

### steps to solve the problem:
1. upgrate AKS cluster from 1.18.1 to 1.19.3 - **error stil exists**
2. prepare nginx deployment for linux node, for eliminate problem with access to ACR
- deployment file for nginx for linux node: **https://github.com/qadevpl/AKS_hello_world/blob/main/nginx_apps_deployment.yaml**
- linux node works, NGINX is available by through reverse proxy on address: **http://20.191.44.116/**

### next steps to fix problem:
1. prepare docker image not from laptop win 10 system with os version **10.0.19041-based** but in win server 2019 with os version: **10.0.17763** or compatibile with this version
2. change AKS win node version and redeploy node

## refactoring solution/next step:
1. refactor ARM template and add more parametrization
2. prepare powershell script for running ARM template
3. join powershell script to azure devops
4. prepare helm template for deployment, services, ingress yaml
5. prepare **Azure key vault** storage for login and password

## open question:
**How do you consider to make the application HA?**
step to implement HA for app on AKS:
1. AKS deployment across multiple availability zones - zure availability zones protect resources from data center-level failures by distributing them across one or more data centers in an Azure region.
2. Azure Active Directory integration - AKS clusters can be integrated with Azure Active Directory so that users can be granted access to namespaces in the cluster or cluster-level resources using their existing Azure AD credentials
3. Pod traffic control through network policy implementation - Network policies can be used to define a set of rules that allow or deny traffic between pods based on matching labels.
4. Monitoring AKS cluster - implement monitoring on azure monitor or externall tool like Prometheus


