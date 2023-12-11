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
git clone https://github.com/haquocdat543/gitlab-ci.git
cd gitlab-ci
```
### 2. Initialize
```
cd backend
terraform init
terraform apply --auto-approve
```
```
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v5.30.0
Terraform has been successfully initialized!
You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
var.key_pair
  Enter a value:
```
Enter your `key pair` name. If you dont have create one. [keypair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)

Outputs:
```
Sonarqube-Server = "ssh -i ~/Window2.pem ubuntu@54.92.66.231"
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

First, you need to create gitlab token . [gitlab-token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)

Then login with command :
```
glab auth login
```
Then `cd` to your `gitlab-repo`
```
glab variable set SONAR_TOKEN <sonarqube-token>
glab variable set SONAR_HOST_URL <sonarqube-server-url>
```
### 3. Dockerhub config

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
### 4. Install Gitlab-Runner
ssh to `Sonarqube-server` 
```
cd
sh ./gitlab-runner.sh
```

Go back to gitlab `Settings` > `CI/CD` > `Runners` > `Expand` > `three dots` beside `New Project Runner` > `Show runner installation...`

Below the `Command to register runner` > Copy it and run on `Sonarqube-server` 
* 1. Enter
* 2. Enter
* 3. Whatever you want
* 4. Whatever you want
* 5. ( Optional )
* 6. shell

and then run gitlab-runner
```
cd
sh ./bashlog.sh
sudo gitlab-runner run
```
Add file `.gitlab-ci.yml` with content
```
stages:
    - npm
    - sonar
    - trivy file scan
    - docker
    - trivy image scan
    - run container

Install dependecy:
    stage: npm    
    image:
        name: node:16
    script:
        - npm install    

sonarqube-check:
  stage: sonar
  image: 
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script: 
    - sonar-scanner
  allow_failure: true
  only:
    - main

Trivy file scan:
  stage: trivy file scan
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy fs . 

Docker build and push:
  stage: docker
  image:
    name: docker:latest
  services:
    - docker:dind   
  script:
    - docker build -f Dockerfile . -t haquocdat543/gitlab-v1:latest
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker push haquocdat543/gitlab-v1:latest

Scan image:
  stage: trivy image scan
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image haquocdat543/gitlab-v1:latest

deploy:
  stage: run container
  tags:
    - gitlab
  script:
    - docker run -d --name youtube -p 8080:8080 haquocdat543/gitlab-v1:latest


```

Go back to `gitlab-repo` repo and make a commit to start build
```
git add .
git commit -m "Add sonar-project.properties"
git push
```

Go back to gitlab > `Build` > `Pipelines` to see result 

Copy `<sonarqube-server-ip>:8080` to access vue app

### 5. Destroy
```
terraform destroy --auto-approve
```
