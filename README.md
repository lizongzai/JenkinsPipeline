## Build Docker Image Using Jenkins Pipeline & Push to AWS ECR





> In this tutorial we will learn how to install Jenkins, Docker, install required Jenkins plugins for docker, run automatic Docker Build with Dockerfile using Jenkins Pipeline, pushing the docker Image to AWS ECR





#### **Configuring the System & Environment**

**Step 1:** First Let's install Jenkins, in this tutorial we are using AWS Linux AMI 2, and for that is are the installations steps documentation:

https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS

Install Docker:

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

#### `Download and install Jenkins`

##### To download and install Jenkins:

- To ensure that your software packages are up to date on your instance, use the following command to perform a quick software update:

  ```bash
  [ec2-user ~]$ sudo yum update –y
  ```

- Add the Jenkins repo using the following command:

  ```bash
  [ec2-user ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  ```

- Import a key file from Jenkins-CI to enable installation from the package:

  ```bash
  [ec2-user ~]$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  [ec2-user ~]$ sudo yum upgrade
  ```

- Install Java:

  ```bash
  [ec2-user ~]$ sudo amazon-linux-extras install java-openjdk11 -y
  ```

- Install Jenkins:

  ```bash
  [ec2-user ~]$ sudo yum install jenkins -y
  ```

- Enable the Jenkins service to start at boot:

  ```bash
  [ec2-user ~]$ sudo systemctl enable jenkins
  ```

- Start Jenkins as a service:

  ```bash
  [ec2-user ~]$ sudo systemctl start jenkins
  ```

- You can check the status of the Jenkins service using the command:

  ```bash
  [ec2-user ~]$ sudo systemctl status jenkins
  ```

##### Configure Jenkins:

Jenkins is now installed and running on your EC2 instance. To configure Jenkins:

- Connect to http://<your_server_public_DNS>:8080 from your favorite browser. You will be able to access Jenkins through its management interface:

  ![image-20220525112504599](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525112504599.png)

  

- As prompted, enter the password found in **/var/lib/jenkins/secrets/initialAdminPassword**. Use the following command to display this password:

  ```bash
  [ec2-user ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```

- The Jenkins installation script directs you to the **Customize Jenkins page**. Click **Install suggested plugins**.

  ![image-20220525112642372](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525112642372.png)

  ![image-20220525112732535](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525112732535.png)

  

- Once the installation is complete, **Create First Admin User**, click **Save and Continue**.

  ![image-20220525113841662](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525113841662.png)

  ![image-20220525113920071](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525113920071.png)

- On the left-hand side, click **Manage Jenkins**, and then click **Manage Plugins**.

  ![image-20220525114224097](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525114224097.png)

  

- Click on the **Available** tab, and then enter **Amazon EC2 plugin** at the top right.

- Select the check box next to **Amazon EC2 plugin**, and then click **Install without restart**.

  ![image-20220525114313105](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525114313105.png)

  

- Click **Add a new cloud**, and select **Amazon EC2**. A collection of new fields appears.

- Fill out all the fields. (Note: You will have to Add Credentials of the kind AWS Credentials.)

Configure Jenkins:

After completing this tutorial, be sure to delete the AWS resources that you created so that you do not continue to accrue charges.

#### Delete your EC2 instance

1. In the left-hand navigation bar of the Amazon EC2 console, choose **Instances**.
2. Right-click on the instance you created earlier and select **Terminate**.



#### `To install Docker on an Amazon EC2 instance`

1. Launch an instance with the Amazon Linux 2 AMI. For more information, see [Launching an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html) in the *Amazon EC2 User Guide for Linux Instances*.

2. Connect to your instance. For more information, see [Connect to your Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html) in the *Amazon EC2 User Guide for Linux Instances*.

3. Update the installed packages and package cache on your instance.

   ```bash
   sudo yum update -y
   ```

4. Install the most recent Docker Engine package.

   ```bash
   sudo amazon-linux-extras install docker
   ```

5. Start the Docker service.

   ```bash
   sudo service docker start
   sudo systemctl enable docker
   ```

6. Add the `ec2-user` to the `docker` group so you can execute Docker commands without using `sudo`.

   ```bash
   sudo usermod -a -G docker ec2-user
   ```

7. Log out and log back in again to pick up the new `docker` group permissions. You can accomplish this by closing your current SSH terminal window and reconnecting to your instance in a new one. Your new SSH session will have the appropriate `docker` group permissions.

8. Verify that you can run Docker commands without `sudo`.

   ```bash
   sudo docker info
   ```

   

**Step 2:** **Add Jenkins user to Docker group**

```bash
sudo usermod -a -G docker jenkins
```



**Step 3:** **Restart Jenkins service**

```bash
sudo service jenkins restart
```



**Step 4:** **Reload system daemon**

```bash
sudo systemctl daemon-reload
```



**Step 5:** **Restart Docker service**

```bash
sudo service docker restart
```



**Step 6:** **Install Jenkins plugins for Docker**

1. Docker plug-in 
2. Docker pipeline plug-in

```bash
sudo yum install -y git
```



**Step 7:** **Create AWS ECR Repo in AWS**

![image-20220525115020439](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115020439.png)



**Step 8:** **Create IAM role and** With `AmazonEC2ContainerRegistryFullAccess` policy and **attach with the Jenkins EC2 Instance**

![image-20220525114946608](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525114946608.png)



![image-20220525115147399](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115147399.png)

![image-20220525115214437](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115214437.png)



**Creating Jenkins Pipeline**

**Step 1 – Create a pipeline in Jenkins, with your project name**

![image-20220525115532652](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115532652.png)

![image-20220525115601722](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115601722.png)



![image-20220525115615580](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525115615580.png)



`click on Pipeline Syntx`

![image-20220525120047689](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525120047689.png)



`Generate Pipeline Script`

![image-20220525120107996](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525120107996.png)





**Step 2: Use below pipeline code**

> In place of “**YOUR_ACCOUNT_ID_HERE**” paste your AWS Account ID
>
> In place of “**CREATED_AWS_ECR_CONTAINER_REPO_REGION**” copy created ECR repo region id
>
> “**IMAGE_REPO_NAME**” set your ECR repository name
>
> “**IMAGE_TAG**” mention your desired tag



```
pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="YOUR_ACCOUNT_ID_HERE"
        AWS_DEFAULT_REGION="CREATED_AWS_ECR_CONTAINER_REPO_REGION" 
        IMAGE_REPO_NAME="ECR_REPO_NAME"
        IMAGE_TAG="IMAGE_TAG"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/sd031/aws_codebuild_codedeploy_nodeJs_demo.git']]])     
            }
        }
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
    }
}
```



**Step 3:** Click on Build to see build happening properly and Docker image getting published to AWS ECR

![image-20220525120459500](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525120459500.png)



![image-20220525120405062](C:\Users\86181\AppData\Roaming\Typora\typora-user-images\image-20220525120405062.png)




