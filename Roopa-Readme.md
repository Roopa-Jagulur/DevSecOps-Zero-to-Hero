Chartgpt
--------
Implimenting secuirty in all activities that devops engineer do.

1. OWASP threat dragon is used by organization in real time ?

Yes—OWASP Threat Dragon is used in real organizations, but mostly for design-time threat modeling, not continuous real-time monitoring.
It helps teams identify security risks early in development, rather than detecting attacks in live systems.

2. from when orgainzisation started using  OWASP threat dragon for real time projects?

Organizations have been using OWASP Threat Dragon since around 2018–2019, when it was introduced by OWASP.
It’s mainly adopted in modern DevSecOps workflows for design-time security, not specifically for real-time runtime projects.

git
----
securing git repo's, by following below steps:
- .gitignore - git will not track files that are in .gitignore file.
- Native Git Pre-Commit Hooks (Custom Scripts)  Pre-commit hook - track sensitive info stored in files through patterns. If pattern match with any of the files then hook will block the commit. .git/hooks/pre-commit.sh
- Block commits with Gitleaks, pre-commit framework (works for all (.gitignore) the repositories) : installing pre-commit framework and adding .pre-commit-config.yml (which connects to git leaks) and run $pre-commit install (will add pre-commit file for you under .git/hooks/pre-commit)
- Gitleaks -> Repository & History Scanning $ gitleaks detect - command on files already committed to check any sensitive data been pushed on a repo/repos.
- Gitleaks in GitHub Actions : you as devops engineer you create cicd on checking for any sensitive data on all pull request on repo creating repo/.github/workflows/gitleaks.yml
- Branch Protection Rules: protect main branch -> branches -> add branch ruleset -> enforce branch protection status, including ruleset that checks for match sensitive patterns for all pull requests to the target repo (like release/feature etc.,) 
- RBAC github/ collabators / add only required people to access repo with required permissions.
- Mandatory Reviews and CODEOWNERS: github/ Rules/ rulesets/ add options like require two reviews atleast, require reviews from specific team, require review from codeowners (adding codeowners name will notify codeowners for reviewing changes when pull request raised).
- Dependabot: looks for packages used for application (ex: pom.xml, go.mod etc.,) and checks for packages version for vernerabilities constantly compare package version with vernerabilty DB. If a package version is identified as vernerability version, in this case, Dependabot create pull request and update version in repo to avoid using vernerable version. (repo/.github/dependabot.yml)

Refer this repo 02-Git/README.md for more detailed explianation of above topics.

IaC Security
------------
Best pactices: do not hard code credentails - considering all above git best practices. Now, comes terraform
ex: terraform .tf code to create s3 bucket and check that .tf for any security vernurabilities, to do those checks on terraform file we use checkov $ checkov -d . (now, checkov runs checks on current terraform directory)

Refer this repo 03-IaC/01-Checkov.md for more detailed explianation of above topics.

How do you configure your terraform test suite? include your answer with, we have integrated checkov to our ci file to valiadate terraform configuration everytime when changes pushed to terraform repo.

Vault - why vault?
when terraform run locally, steps are?
hobby project -> variables we pass locally through .env -> execute terraform and create aws resource

when terraform files run as part of real time production?
terraform files run through CI/CD which are maintained in git repo -> clone -> update -> push -> trigger ci and create aws resource. Here ci (ex: github actions system run CI file) accountability credentials are used to create resource on AWS, not with individual user crendentails.

pros following above method: git used as single source of truth and even changes can be tracked easily, change review will take place, backups created, auditing can also be done.

Now, how do we pass credentials to CI pipeline?
note: github->secrets and variable-> provide Actions secrets (secure but not completely secure for longlived credentials)

Vault (centerlized crendential manager) used to use short lived passwords which remain for 5/10 mins to run CI pipelines, after 5/10 mins password change.

flow: Terraform+vault -> prod env -> clone -> update -> push -> Github actions run CI pipeline -> as part of pipeline run, pipeline request vault for short lived credentials (10mins) to connect to AWS -> connect AWS create resource.

Note: Hasicorp Vault is sloving longlived credential problem introducing short lived credentials.

github uses OIDC to request vault (we initially provide AWS crendentials) using those credentails Vault create IAM_users which is short lived / temporary credentials and provide it to github through OIDC protocol/jwt (java web tokens).

Vault can be installed on EC2 instance / K8S.

Refer this repo 03-IaC/02-Vault.md for more detailed explianation on how to create EC2 instance and install Vault. 

Vault production setup on EC2 + Github
- install vault on ec2 
- run vault
- configure vault, by enabling AWS engine (configure connection using your AWS IAM user keys and create role that need to create s3 bucket)
- enable OIDC (Open Id COnnect - is a simple login protocol/system that lets you signin into different apps using existing account) 
Note: all above stes performed on Valut EC2 instance

Terraform code modified to use valut temporary credentials to create s3 bucket on AWS.

GitHub Workflow - create CI github actions workflow to trigger on push action to repo where file is mentioned to 'fetch keys from vault to create s3 bucket on aws, creating temperory IAM user and run terraform init, plan and apply.

Container Security
------------------
- running containers as non-root users
- multistage docker file
- using distroless images in docker file
- importance of .dockerengine
- docker run

Refer this repo 04-Container-Security/README.md for more detailed explianation and demo for above topics.

For demo purpose, using app.js (a simple java application) and package.json (dependency file for app.js)

- running container as root user will grant access conatiner user to host machine, which means host machine is compromised and also all the containers running on that host is also compromised. Hence run container using non-root user.

# Create non-root user in your dockerfile adding below line
RUN groupadd -r appuser && useradd -r -g appuser appuser

# and 

# Switch to non-root user
USER appuser

👉 The app now runs as a **non-root user**.

⚠️ Still not secure:
- Image is large (due to image containing/retaining system and application packages that do not need during container run time
Note: for Building image require lot package but while running application on container doesn't need all of them that were used at build stage) Note: packages used today would be vernurable tomorrow - so cannot turst.
- Build tools remain
- OS utilities exist

To make your docker image secure and will also reduce docker image size. so, consider implimenting multistage docker file having build and run stages where you just copy binary required to run the docker image on container not any unrequired packages.

There will still be more unsed packages during container run time even on slim images used at docker file run stage, those can also be removed by using distroless images- reduce images size and also removes system related binaries within the container.

instead light weight slim packages, we use distroless image

ex:
from
# -------- Runtime Stage --------
FROM node:25-slim

to 
# -------- Runtime Stage --------
FROM gcr.io/distroless/nodejs20-debian12


## STEP 3: Add `.dockerignore` - create .dockerignore in folder where your dockerfile has and put all the files that do not require them to be copied to dockerimage by 'COPY . .' command in dockerfile.

## STEP 6: Harden the Runtime (Defense in Depth)

Even secure images need runtime protection.

---

### Hardened Run Command

bash command:
docker run \
  --read-only \ (container run as readonly)
  --tmpfs /tmp \ (if need write, can be written on /tmp)
  --cap-drop ALL \ 
  --security-opt no-new-privileges \ (above and this command, do not all to perform any previliage change operations)
  --pids-limit 100 \ (to limit processes)
  --memory 256m \ ( limit container memory)
  --cpus 0.5 \ (limit container cpu)
  -p 3000:3000 \ (expose port)
  secure-app

  Securing kubernetes
  -------------------
- what are namespaces (first step of security)
- RBAC - different components 
- network policies (ex: how to restrict access to DB i.e deployed in your k8s)
-  Advanced policy enforcement (Ex: how do you setup and manage your k8s cluster as per your company's compliance / security - kyverno)
- secrets in k8s (how do you store sensitive information within youe k8s cluster)
- git for secerets -External secerets operator integratig with vault provider (how do you store your k8s seceret references using Version control systems like git)

Refer this repo 05-Kubernetes/README.md for more detailed explianation and demo for above topics.

namespace (logical saperation)- offers isolation helps to manage single k8s across different teams in a company.

namespace - resource quota (mem, cpu)
          - resouce limit (resources per container need to use)   


RBAC:
admin create service account 
admin create role (permissions - r, w, execute) ex: dev role, test role etc.
admin bind role to service account using role binding for access services info within namespace
admin bind role to service account using cluster role for access all services info within cluster (all namespace within cluster)

Network policies: by default any pod in cluster can talk to any other pod by default.

using network policies you can define access to your pods using ingress and egress ex: restrict frontend appln talking to DB except backend appln service.

At $ kubectl run <podname> : as part of this command run we create labels and annotations as part of this command run to define network policies.

At $ kubectl expose pod <podname>: as part of this command run we create service name to make service reachable even after IP change due to container / pod restart.

note: AWS EKS is setup with VPC CNI and VPC CNI already has support for the network policies.

Kyverno (policy as code): enforce policy to a cluster.

If need define k8s cluster as per company/organization compliance, can be defined using Kyverno -> admission -> validation and mutation 

validation ex: stop when image has latest as tag/ pods cannot defined without resource quota / limits etc.,

mutation ex: stop creating pod/ services / config maps when there is no labels or annotations defined / create lables / annotation automatically.

Secret Management:
creating secrets using kubernetes secret manifest (.YAML) used to store and manage secrets securely in a kubernetes cluster and we invoke them as env variables.

Storing kubernetes secrets in git or any where is still not safe as kubernetes secrets are base64 encoded not encrypted hence using External Secrets Operator is safe play.

External secrets operators avaialable are Vault, AWS secrets manager, Azure key vault where you store all your secrets and you use just their references on kubernetes cluster. This way secrets are safe.

Instead using kubernetes secret manifeast (.YAML) we use External secret manifeast (.YAML) where we reference external secret operator we want and from where scerets must be synced.

Application Security
--------------------
Refer this repo 06-Application-Security/README.md for more detailed explianation and demos.

*****









