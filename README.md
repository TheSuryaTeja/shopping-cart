# Dockerizing the Shopping-Cart Application

## env variable changes

- Edit src/application.properties
- replace localhost with the container name of mysql db container

```
db.connectionString = jdbc:mysql://mysql:3306/shopping-cart
```

## Creating dockerfiles

- Create dockerfile.app for building application image

```bash
FROM tomcat:9
ADD target/shopping-cart-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/shopping-cart-0.0.1-SNAPSHOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]

```

- Create docerfile.mysql for building mysql image

```bash
FROM mysql:latest
ADD databases/mysql_query.sql docker-entrypoint-initdb.d/mysql_query.sql
ENV MYSQL_ROOT_PASSWORD root
EXPOSE 3306

```

> Both the images have been built and tested locally first to ensure there is no error and pushed to the repo.

- Create a Jenkinsfile

```bash
pipeline {
    agent any
    tools{
        maven 'jenkins-maven'
    }
    stages{
        stage('Build Maven'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/TheSuryaTeja/shopping-cart.git']]])
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t iamsurya/shopping-cart-app -f Dockerfile.app .'
                    sh 'docker build -t iamsurya/shopping-cart-mysql -f Dockerfile.mysql .'
                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhub-pwd')]) {
                   sh 'docker login -u iamsurya -p ${dockerhubpwd}'

                    }
                   sh 'docker push iamsurya/shopping-cart-app'
                   sh 'docker push iamsurya/shopping-cart-mysql'


                }
            }
        }

    }
}

```

## Jenkins Setup

```bash
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11

```

- Volume mount "jenkins_home" is used to retain data even if container gets deleted

- Browse [http://localhost:8080/](http://localhost:8080/) and complete initial jenkins setup

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/init.png?raw=true)

- In the Dashboard section create a new item as pipeline

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/pipeline.png?raw=true)

- Add the github url of project repo in Source Code Management section of configure tab

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/git.png?raw=true)

- Add pipeline script or

- We can also add pipeline script from scm since we already pushed it to git

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/pipescm.png?raw=true)

- Create your dockerhub password as "dockerhub-pwd" credential before building

##Output

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/build_success.png?raw=true)

![alt text](https://github.com/TheSuryaTeja/shopping-cart/blob/master/images/result.png?raw=true)

- Exposed 8081 port since 8080 is already being used by jenkins by default

- Same images can be deployed in kubernetes with deployment or statefulset

```
        stage('Deploy to k8s'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml',kubeconfigId: 'k8sconfigpwd')
                }
            }
        }
```

## Author

- Surya Teja
- Mail - **heysuryateja@gmail.com**
- Connect on [Linkedin](https://www.linkedin.com/in/suryateja2000/)
