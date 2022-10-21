# DOCKER JENKINS CI/CD

1. Install docker in jenkins server with the following command
```
apt update
apt install docker.io
```
2. Add jenkins user to docker
```
sudo usermod -aG docker jenkins
```

3. Restart jenkins
```
systemctl restart jenkins
```

## CREATE ECR REPOSITORY
4. Login to aws and go to ECR

![Capture3](https://user-images.githubusercontent.com/18073289/195808005-55201efe-11bd-47b2-b407-e3ebd778dbc4.PNG)

5. Get Started

![Capture4](https://user-images.githubusercontent.com/18073289/195808145-bf103faa-8581-4afa-978b-bceb135bd45d.PNG)

6. Type in the repository name and create

![Capture5](https://user-images.githubusercontent.com/18073289/195808385-0370a3a9-bf8b-4d44-8208-32e5287338b8.PNG)

## CREATE IAM ROLE FOR JENKINS

7. Go to IAM role to create a new user

![Capture16](https://user-images.githubusercontent.com/18073289/195808961-059075bc-bb14-43f9-a25c-09469af7d983.PNG)

8. Go to users

![Capture17](https://user-images.githubusercontent.com/18073289/195809136-089a76c9-1149-44bd-b804-23b92338faa7.PNG)
 
9. Add Users

![Capture18](https://user-images.githubusercontent.com/18073289/195809310-25a8e470-9f14-4b27-a11c-c550e57bcfe8.PNG)

10.  Add Name and select access key programmatic Access
![Capture19](https://user-images.githubusercontent.com/18073289/195809415-67ec7c01-c083-482c-80db-79b3d51dee47.PNG)

11. Select Attach existing policies and search for ecr

![Capture20](https://user-images.githubusercontent.com/18073289/195809744-6c7b33dc-e6d2-4724-94bc-5e539711e833.PNG)

21. Create User

![Capture21](https://user-images.githubusercontent.com/18073289/195810098-d94e28a9-15f3-4a4c-9235-b0956cdf45cc.PNG)


22. Go to your terinal and install aws cli on the jenkins server with the following command

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

23. Configure aws on the jenkinks server with the following command

```
aws configure

```
Then follow the prompt

![Capture7](https://user-images.githubusercontent.com/18073289/195811672-2e1da01b-cbda-40f5-88a4-5eef4af4858d.PNG)

## CONFIGURE JENKINS

24. Login to Jenkins and go to manage Jenkins then select manage plugin

You need to install the following plugins

* nexus artifact uploader
* sonarqube scanner
* build timestamp
* pipline maven integration
* pipeline utility steps

![Capture16](https://user-images.githubusercontent.com/18073289/195813070-8837178a-7433-46e8-a0ee-abf555a92299.PNG)

25. Login to Jenkins and go to manage Jenkins then Manage Credentials >> System >> Global Credentials >> Add New Credentials

![Capture11](https://user-images.githubusercontent.com/18073289/195813819-63326824-082b-4636-9d95-8148794238f8.PNG)

26. Select aws credentials and fill in the access key and secret key

![Capture12](https://user-images.githubusercontent.com/18073289/195814168-59cd471b-2bfa-4df0-88c3-e5cfacda881a.PNG)


27. Go to the Jenkins Job >> configure >> scroll down to the pipeline and paste the following code.
```
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    environment {
        registryCredential = 'ecr:us-east-1:aws-cred'
        appRegistry = "xxxxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/myappimg"
        vprofileRegistry = "https://xxxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com"
        
    }
    stages{
        stage('Fetch code') {
          steps{
             git branch: 'docker', url: 'https://github.com/xxxxxxxxxxxxx/vprofile-project.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
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
         
    }
      post {
            always {
                echo 'Slack Notifications.'
                slackSend channel: '#cicd',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            }
        }
       
}
```
