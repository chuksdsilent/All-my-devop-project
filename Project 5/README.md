# DOCKER CI/CD DEMOSTRATION

## CONTINUOUS INTEGRATION
1. install Jenkins and maven using the following command
```
sudo apt update
sudo apt install openjdk-11-jdk -y
sudo apt install maven -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

2. Login to jenkins >> manage Jenkins >> Manage Credentials >> system(stores scoped to jenkins) >> global credentials >> Add credentials

![Capture1](https://user-images.githubusercontent.com/18073289/196464275-66a3485e-4112-4791-a1ab-15c222819716.PNG)


3. setup docker engine in jenkins with the following command
```
apt install docker.io
usermod -aG jenkins docker
```

4. install the following plugins plugins in jenkins
configure jenkins >> manage plugins
* docker
* pipeline AWS steps
then install without restarting

![Capture2](https://user-images.githubusercontent.com/18073289/196464345-1395419a-575f-44ee-9313-893696c2a0a5.PNG)


## CONTINUOUS DEPLOYMENT

5. Login to AWS >> Create a ECS Cluster

![Capture3](https://user-images.githubusercontent.com/18073289/196464538-562f0c5b-358e-49b9-ae8c-62bb1d3dbd97.PNG)


6. Create a Task Definition

![Capture5](https://user-images.githubusercontent.com/18073289/196465526-cb91040c-91c1-4177-b48c-fe060b2d5648.PNG)



7. Open the ecs cluster >> services >> deploy a new service

![Capture6](https://user-images.githubusercontent.com/18073289/196465563-70f978df-b028-458e-aa3c-26c098c1672a.PNG)



10. Go to security group and allow traffic from port 8080



11. Go to target group >> check health tab >> override port 8080 and change threshold to 2


![Capture7](https://user-images.githubusercontent.com/18073289/196465747-ca2c6ae4-f794-4c6d-a122-1957e7b7d2c0.PNG)

![Capture8](https://user-images.githubusercontent.com/18073289/196465783-3738995c-551e-4bab-87f3-8a24a1eda66e.PNG)


12 copy the load balancer dns and check site on your browser

![Capture9](https://user-images.githubusercontent.com/18073289/196466155-43cf8bd2-de7c-4cc8-ad3e-2e90d43572e6.PNG)


13. copy the cluster and service name and paste on the code below also make sure your aws credentials is correct


```
pipeline {
    agent any
    environment {
        registryCredential = 'ecr:us-east-2:awscreds'
        appRegistry = "951401132355.dkr.ecr.us-east-2.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://951401132355.dkr.ecr.us-east-2.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileappsvc"
    }
  stages {
    stage('Fetch code'){
      steps {
        git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
      }
    }


    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }

    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('build && SonarQube analysis') {
            environment {
             scannerHome = tool 'sonar4.7'
          }
            steps {
                withSonarQubeEnv('sonar') {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

     }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
     }
     
     stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awscreds', region: 'us-east-2') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

  }
}
```








