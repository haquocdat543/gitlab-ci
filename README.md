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

Navigate to `Project` > `Manually`

* Project display name : Whatever you want > `Set Up` > `With Gitlab CI`

Add file `sonar-project.properties` to you repo with content :
```
sonar.projectKey=<your-project-key>
sonar.qualitygate.wait=true
```
Click `Continue`

Click `Generate a token` > `Copy`

Go back to gitlab `Settings` > `CI/CD` > `Variables` > `Expand` > `Add variable` > Enter key and value
```
SONAR_TOKEN <sonarqube-token>
SONAR_HOST_URL <sonarqube-server-url>
```
Click `Continue`

You can use [gitlab-cli](https://gitlab.com/gitlab-org/cli) instead
```
glab variable set SONAR_TOKEN <sonarqube-token>
glab variable set SONAR_HOST_URL <sonarqube-server-url>
```

Navigate to Dockerhub > `Finger print in top right corner` >  `Account Settings` > `Security` > `New Access Token` > Enter name > `Generate` > Copy it

Go back to gitlab `Settings` > `CI/CD` > `Variables` > `Expand` > `Add variable` > Enter key and value

```
DOCKER_USERNAME <your-dockerhub-name>
DOCKER_PASSWORD <your-docker-token>
```
You can use [gitlab-cli](https://gitlab.com/gitlab-org/cli) instead
```
glab varible set DOCKER_USERNAME <your-dockerhub-name>
glab varible set DOCKER_PASSWORD <your-docker-token>
```

### 3. Destroy
```
terraform destroy --auto-approve
```
