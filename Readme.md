`Jenkins: v2.375.1` `terraform: v1.3.6`

## Run the project.
1. [Run jenkins with docker](https://hub.docker.com/_/jenkins)
```bash
docker run --name myjenkins -p 8080:8080 -p 50000:50000 -v $HOME:/var/jenkins_home jenkins/jenkins:lts-jdk11
```

2. Access http://localhost:8080 and authorized with admin password. Default path of password is `/var/jenkins_home/secrets/initialAdminPassword`.

3. [Install terraform with available version](https://developer.hashicorp.com/terraform/downloads)
```bash
# Enter container with root user.
docker exec -it --user root myjenkins bash

# Install terraform
apt update
apt-get install wget
apt-get install unzip

wget https://releases.hashicorp.com/terraform/1.3.6/terraform_1.3.6_linux_amd64.zip
unzip terraform_1.3.6_linux_amd64.zip

# Check version
terraform --version
```

4. [Follow the video to create git repo credential](https://www.youtube.com/watch?v=AG26QMUFzrw)

5. [Follow the tutorial to setup terraform plugin, AWS Credentials, Jenkins Pipeline](https://medium.com/nerd-for-tech/deploying-aws-resources-using-terraform-and-jenkins-pipeline-1c706f1a2e7c)
    - You might need to copy [his Jenkinsfile](https://github.com/troy-ingram/week-24-project) and modify by youself.

6. After building the pipeline, jenkins will load the Jenkinsfile and update the content.
