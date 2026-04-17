3 tier => frontend (UI), backend (Interface between UI and DB) and DB (Storage)

3 tier application used for this project is : Refer - https://github.com/iam-veeramalla/Jerney

Jerney application is anyother blogging application on the internet, it is a light weight blogging application. 
Application features:
- can post something it is trending on the internet.

End Goal:
---------
- running appln locally
- Automate infra provisioning, following IaC
- Containerize appln (1. create docker file, 2. create docker images (for front end + backend + DB))
- Build and run appln using docker compose (docker compose can run on windows./mac/linux machines)
- create k8s manifeast to deploy appln on cluster.
- impliment CI/CD (all above steps must automatically implimented) 
- more importantly consider maintaining security on all above steps you perform.

Note: As devops engineer, in every action you perform, you should take care of security (devsecops)

Running Jerney Application Locally:
- usually, developers provide instructions on how to run the appln they deveoped locally in form of README / Confluence / Sharepoint
For this project we find instructions Jerney/deploy/setup.sh [tried to follow instructions in setup.sh file and run the command locally on EC2 instance, as a result we see following:]



Automate Kubernetes cluster infra provisioning, using Terraform IaC:



Containerizing Jerney Appln: Jerney/frontend/Dockerfile
Dockerfile for programming language used for developing Jerney Appln i.e nodejs
Stage 1:
- base image
- choose working dir (/app)
- copy dependencies (package.json)
- install dependencies
- copy all source code files
- build appln (o/p: dist, jar, etc.,)
Stage 2:
- light weight base image (if dealing with front end we go with nginx light weight image)
- copy artifact/binary from Stage 1-build step
- use non-root user and entry point for your cmd.

Note: Multistage docker file might have morethan 2 stages when team deside to have and run frontend + backend + db as a single microservice.

Jerney/backend/Dockerfile
Note: backend appln here is very light weight hence /Dockerfile is written in such as way that build and run takes place while deployment itself instead building and copying appln build on to next stage for running in the dockerfile. 

here we are not creating docker images for DB because DB docker images are already avaiable on Dockerhub. Ex: search postgress in DockerHUb you find official docker images for postgress DB.

Dockercompose file - why?
In our this project, we have 3 containers frontend, backend and DB (might be chance can be more than 3)
and 
we need to make sure they all connected and can communicate each other, making sure frontend, backend and DB are on the same network, look at volume requirements, environment variable requirements etc., all these can be done using dockercompose file.


k8s manifeast to deploy appln on cluster:
- create depolyment manifeast + PVC attached to it + SC (storage class - where you want to created storage for your appln ex: EBS/EFS/FSX etc.,)

For jerney appln, we will have
frontend -> deployment + service manifeast
backend -> depolyment + service + network policy (backend only be accessed by frontend) + secrets (for database credentails)
DB -> depolyment + service + PVC + storage class

Here instead of creating all above mentioned manifeast, instructor is just creating one .yml to deploy on eks cluster -> jerney/devops (branch) -->/k8s/jerney.yaml

CI/CD:
-using github actions
-adding linting check step
-adding sca(software composition analysis) step 
-build docker image and create frontend/backend image
-scanning docker image using trivy (security best pactice)
-perform linting on Dockerfile (to check security infeciences)
-checkov for security scanning for manifests (ex: k8s/terrafotm manifests)
-update k8s manifests file with lastest versions of docker images and commit

Jerney/.github/workflows/ci-cd.yml

