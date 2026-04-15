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

