# gitlab-ci
This is a demonstration of using Gitlab ci

This link with two repositories :
* [gitlab-ci](https://gitlab.com/haquocdat543/ci)
## Prerequisites
* [git](https://git-scm.com/downloads)
* [terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [config-profile](https://docs.aws.amazon.com/cli/latest/reference/configure/)
## Start
### 1. Clone project
```
git clone https://github.com/haquocdat543/devops-infra.git
cd devops-infra
```
### 2. Backend
```
cd backend
terraform init
terraform apply --auto-approve
```
### 3. Config sonarqube server
Copy `<your-sonarqube-server-public-ip>:9000` and open it in your browser
Login with default username `admin` and default password `admin`

### 3. Destroy
```
terraform destroy --auto-approve
```
