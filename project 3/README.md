# INTEGRATION SLACK INTO JENKINS

In this project we will integrage slack into our Jenkins CI pipeline

1. Search for slack app jenkins and select Jenkins CI | Slack App Directory

![Capture1](https://user-images.githubusercontent.com/18073289/195660473-fd3e8272-a841-4786-aed1-37efd665fabb.PNG)

2. Click on add to slack

![Capture2](https://user-images.githubusercontent.com/18073289/195661031-93e39c26-69d4-4610-8e29-ce1d7c97904e.PNG)

3. Create a new workspace or select an already existing one and click on add jenkins CI integration

![Capture3](https://user-images.githubusercontent.com/18073289/195661204-9626ab42-351a-4e3c-afa9-71e69358f181.PNG)

4. Go down to step 3 and copy the token there

![Capture14](https://user-images.githubusercontent.com/18073289/195662523-c07e4e15-48bb-4387-b3c7-aec7bad9960c.PNG)

5. Login to Jenkins and go to Manage Credentials

![Capture4](https://user-images.githubusercontent.com/18073289/195661672-1239955a-232c-484c-bb55-334e08a3614b.PNG)

6. Go to credential under Stores scoped jenks

![Capture5](https://user-images.githubusercontent.com/18073289/195662008-dc9b85ae-2b2f-4f1e-96e2-981d8af45b44.PNG)

7. Global credentals >> add credentials

![Capture7](https://user-images.githubusercontent.com/18073289/195662200-e47192a3-60f0-4065-b39c-d36314e79a94.PNG)

8. Select secret text, add id and save

![Capture8](https://user-images.githubusercontent.com/18073289/195662868-3a5a123e-4c18-4e38-b256-ab15be844cc0.PNG)

9. Go to Manage Jenkins > Manage Plugin

![Capture9](https://user-images.githubusercontent.com/18073289/195663278-cbf26df3-ce2b-4b5d-a12c-791a27b7cd0f.PNG)

10 Go go available > search for slack Notification

![Capture10](https://user-images.githubusercontent.com/18073289/195663477-9f255f79-992b-424d-bc70-aa0f123c2cee.PNG)

11. Select >> install without restart

![Capture10](https://user-images.githubusercontent.com/18073289/195664167-e6cca8cf-193a-4e9d-97a1-e79c5dd82a32.PNG)

12. Go to manage Jenkins >> Configure System

![Capture12](https://user-images.githubusercontent.com/18073289/195665972-d91ae8f7-e1cb-454f-bef0-c3cbd1df7b04.PNG)

13. Scroll down to slack section >> Enter workspace, select credentials and type the channel >> save

![Capture13](https://user-images.githubusercontent.com/18073289/195666051-6e80fd52-e7e7-47e2-b651-b42a5418f4bb.PNG)

14. Go to the Jenkins job >> Configure >> Enter the following code
```
  post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#cicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
    
 ```








