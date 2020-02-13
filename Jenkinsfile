pipeline {

    agent any


    parameters { 
        string(defaultValue: "https://github.com", description: 'Whats the github URL?', name: 'URL')
   
   //In this pipeline you will add a string parameter to which you can add a URL. When you run the build it will ask for the parameter
    }



    stages {
        stage('Checkout Git repository') {
           steps {
                git branch: 'master', url: "${params.URL}"
           
     //To use this parameter use "${params.URL}": This pipeline will clone the github repo in the first stage and print the URL in the next (echo) stage:      
            }
        }

        stage('echo') {
           steps {
                echo "${params.URL}"
            }
        }
  
    task'Code compile'
    echo'Code compile'
    // Get maven home path
    def mvnHome = tool name: 'maven', type: 'maven'
    sh "${mvnHome}/bin/mvn clean compile"
    sleep 3
  
    task'Package'
    echo 'Build package'

    sh "${mvnHome}/bin/mvn clean package -DskipTests=true"
    sleep 3
  
    task'NexusDeploy'
    echo 'Store to Artifactory'
    sh "${mvnHome}/bin/mvn deploy -DskipTests=true"
    sleep 3

    task'BuildImage'
    echo'Build Docker Image'

   sh "sudo docker build -t hello-world-img ."
   sleep 2

   task 'Push to Docker Hub'
   sh "sudo docker tag hello-world-dev innovectutre/hello-world-img:latest"
   sh 'docker push innovectutre/hello-world-img:latest'
   sleep 10
  
   task'Deploy-DEV'
   echo'Deploy to DEV Environment'

   sh 'sudo docker run --name innovectutre -p 8088:8080 --rm -t -d hello-world-img:latest'
   sleep 10
   
    sshagent(['tomcat']) {
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.002 "cd /home/ubuntu && rm -rf innovectutre && mkdir innovectutre"'
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.002 "cd /home/ubuntu/java_app && sudo docker pull innovectutre/hello-world-img:latest"'
    sleep 120
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.002 "cd /home/ubuntu/java_app && sudo docker run --name helloworld_dev -p 8082:8080 --rm -t -d innovectutre/hello-world-img"'
   
    }
    task'Integration Test'
    echo'Selenium-Integration Test'
    sh "${mvnHome}/bin/mvn failsafe:integration-test -Dskip.surefire.tests"
    sleep 3
   
    task'Functional Test'
    echo'Selenium-Functional Test'
    build job: 'seleniumtest'
    sleep 3
  
 }

   stage('STAGE') {

   task'BuildImage'
   echo'Build Docker Image'

   sh "sudo docker build -t hello-world-img ."
   sleep 120

   task 'Push to Docker Hub'
   sh "sudo docker tag Hello-World-stage innovectutre/hello-world-img:latest"
   sh 'docker push innovectutre/hello-world-img:latest'

   sleep 120
   task'Deploy-STAGE'
   echo'Deploy to STAGE Environment'

    
    sh 'sudo docker run --name innovectutre -p 8081:8080 --rm -t -d hello-world-img'

    sleep 20
   
    sshagent(['tomcat']) {
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.001 "cd /home/ubuntu && rm -rf java_app && mkdir java_app"'
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.001 "cd /home/ubuntu/java_app && sudo docker pull innovectutre/hello-world-img"'
    sleep 120
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.001 "cd /home/ubuntu/java_app && sudo docker run --name helloworld_statge -p 8083:8080 --rm -t -d javastage-img"'
    }

    task'Regression Test'
    echo'Selenium-Regression Test'
    build job: 'seleniumtest'
    sleep 3
   
    task'Performance Test'
    echo'JMeter LoadBalancing Testing'
    sh '/home/ubuntu/apache-jmeter-5.0/bin/jmeter.sh -n -t jmeter_example.jmx -l Jmeter_test_Report.jtl'
    sleep 3

    
  
  }
 
   stage('PROD') {

   task'BuildImage'
   echo'Build Docker Image'

   sh "sudo docker build -t hello-world-img ."
   sleep 120

   task 'Push to Docker Hub'
   sh "sudo docker tag hello-world-prod innovectutre/hello-world-img:latest"
   sh 'docker push innovectutre/hello-world-img:latest'
   
   sleep 120
   task'Deploy-PROD'
   echo'Deploy to PROD Environment'

    sleep 5
   
    sshagent(['tomcat']) {
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.000 "cd /home/ubuntu && rm -rf java_app && mkdir java_app"'
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.000 "cd /home/ubuntu/java_app && sudo docker pull innovectutre/hello-world-img"'
    sleep 120
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@00.00.000.000"cd /home/ubuntu/java_app && sudo docker run --name helloworld_prod -p 8084:8080 --rm -t -d hello-world-img"'
    }
 
  
  }
 }

   

}
