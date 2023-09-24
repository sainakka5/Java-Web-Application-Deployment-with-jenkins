PROJECT 4 : DevOps CI/CD Pipeline for a Java web application using Jenkins, SonarQube, Docker, and Kubernetes.
STEPS TO CREATE CI/CD Pipeline for a Java web application using Jenkins :
1.	Create an EC2 instance in AWS console, I create an instance with name “Jenkins_project”.
  ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/9a6187ea-c3fd-49ea-9450-bb9c78dbdb7e)

        
2.	Install the Jenkins and its requirements in that instance using Jenkins installation code in Jenkins documentary for centos 7.

        sudo yum install wget -y
        sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
        sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
        yum install fontconfig java-11-openjdk -y
        yum install jenkins -y
        service jenkins start
        systemctl enable jenkins
        service jenkins status

3.	After execution of the above script, enable the http (8080) port in your Jenkins server security group then login into Jenkins with your ip address like, give your <public_ip:8080> on your browser and give the authentication password, which you can find it in path 
 < /var/jenkins_home/secrets/initialAdminPassword > of Jenkins server give that password and login to the Jenkins webserver.
4.	Create a repository in GitHub and give the java-based web application files in that repo and copy the URL of that repository. I had saved in my GitHub repository                  https://github.com/sainakka5/java_web_appilication_code
   ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/05f482f5-df75-4467-a4bd-b769b8653a03)

 
5.	Create a pipeline in Jenkins and give the maven code to build the java application in the pipeline syntax as below code
            ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/3447fcaf-3811-48a6-8108-528ea67cc8f0)

6.	In this script I had given a java application build with maven tool, and tested with SonarQube and it will build a docker image of that application in server with Jenkins.
   
                      pipeline {
                          agent any
                          tools {
                              maven "mvn"
                          }
                      
                          stages {
                              stage('Checkout') {
                                  steps {
                                      git branch: 'main', url: 'https://github.com/sainakka5/java_web_appilication_code.git'
                                  }
                              }
                              
                              stage('Build ') {
                                  steps {
                                      sh "mvn compile"
                                      // Add your build steps here
                                  }
                              }
                              
                              stage('Test') {
                                  steps {
                                      sh 'mvn test'
                                  }
                              }
                              stage('Deploy') {
                                  steps {
                                      sh 'mvn install'
                                      // Add your deployment steps here
                                  }
                              }
                      
                              stage('sonar_code_analysis') {
                                  environment {
                                      scannerHome = tool 'SONARQUBE'
                                  }
                                  steps {
                                   withSonarQubeEnv('SONARQUBE'){
                                       sh "${scannerHome}/bin/sonar-scanner \
                                        -Dsonar.login=5b7342b84cdb97dfb3ff1d9854a7b57dc4ffdf40 \
                                        -Dsonar.host.url=https://sonarcloud.io \
                                        -Dsonar.organization=java-webapp \
                                        -Dsonar.projectKey=java-webapp_ \
                                        -Dsonar.java.binaries=./ "
                                      }
                                  }
                              }
                              stage('Build Docker Image') {
                                  steps {
                                      script {
                                          def dockerImageName = 'dockerfile'
                                          def dockerfilePath = '.'
                                          // Build the Docker image
                                          sh "docker build -t ${dockerImageName} -f ${dockerfilePath}/dockerfile ."
                                      }
                                  }
                              }
                          }
7.	After script was build the stages and application docker image creation was shown as below figure’
    ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/f6d7e7c9-b920-4251-ba4f-a0c2be910ea7)

8.	The docker image was created in our server. With that docker image we have to deploy the application into the EKS cluster.
![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/192ae264-8ab0-41ca-81d9-2a42c438508e)

9.	For this create a repository in dockerhub account and copy the url or path of that repo and push the local docker image to the dockerhub repository by using these commands 
          docker login
          docker tag your-image-name:tag <your-dockerhub-username>/<your-repository-name>:tag 
         docker push <your-dockerhub-username>/<your-repository-name>: tag
 Create an eks cluster by give the configuration in one file and run the command  
         eksctl create cluster -f <filename>
for this we should install eksctl, kubectl, awscli in the server. We can find the script in AWS documentary for these packages.
                        ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/46214ebc-a513-40d5-b1fc-5ba90549d44e)
                       ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/65cfe498-45d8-4e95-be4e-386926c58b3f)
             ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/a3c3ba00-5b3d-4cae-be14-7222a950f68e)
            ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/d9efecfb-f8ac-43c0-9fa6-e22b40ada7c5)
         ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/4186c4fc-ddc6-44ed-ab6d-39795f63461c)
        ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/c96d233b-2246-4da1-bc0f-97d7a966d153)
                
10.	After creation of cluster and nodes deploy the application by create two files deploy.yaml  and service.yaml having the below script in it.
           ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/fd05eec7-75f8-4dd8-9162-e1bf75490904)
                    
11.	At the place of image keyword in deploy file give the copied path of the dockerhub repo path and deploy the files with command
kubectl apply -f deploy.yaml        and         kubectl apply -f service.yaml
          ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/8c477a3f-fdcc-4fff-8c7d-75a70d4e1435)

12.	Pods of nodes and Application was deployed. And we can check by the command “kubectl get all”
              ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/f61facd8-65b4-4b2a-9448-81b9e252aba5)

We got running status so the application was running successfully in nodes
13.	We can check this by checking load balancer in AWS we get all information of the running application as follows.
      ![image](https://github.com/sainakka5/Java-Web-Application-Deployment-with-jenkins/assets/136338958/7338c3e5-4adc-41bb-9dd4-651ec39e5fe8)

We can go into the application with the help of url present in load balancer

14.	Once you have completed the individual steps, you will have a fully automated CI/CD pipeline that triggers on code changes. When code is pushed to the version control system, Jenkins will automatically build the WAR file, test the code with SonarQube, transfer the WAR file to the Docker server, build the Docker image, and finally deploy the application to Kubernetes.
15.	We will be evaluated based on the functionality and correctness of their CI/CD pipeline, adherence to best practices, documentation, and their ability to troubleshoot and handle issues that may arise during the development process. Extra credit may be given for implementing advanced features or optimizations
