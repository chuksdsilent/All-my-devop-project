# CREATING CI IN JENKINS

1. Login to Jenkins and Click on New Item or Create new Job

![Capture5](https://user-images.githubusercontent.com/18073289/195613778-0300256f-1308-40d9-876d-48130c3f94da.PNG)

2. Type the name of the Job and Select Pipeline

![Capture6](https://user-images.githubusercontent.com/18073289/195614342-de94d881-ace0-4d44-a172-b4741b142b7b.PNG)

3. Go down to pipeline section and paste your groove code and save
```
pipeline {
    agent any

    stages {
        stage('Fetch Code') {
            steps {
                git branch: 'vp-rem', url: 'https://github.com/devopshydclub/vprofile-repo.git'
            }
        }
        stage('BUILD') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
            post {
                success {
                    echo 'Generate Analysis Result'
                }
            }
        }
    }
}
```

4. Click on Build Now to build your code

![Capture8](https://user-images.githubusercontent.com/18073289/195615515-f172031a-8eb0-497e-a07f-61da914faa49.PNG)



