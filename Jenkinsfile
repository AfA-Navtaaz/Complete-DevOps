node {
    stage('Git Checkout')
    {
        git branch: 'main', url: 'https://github.com/AfA-Navtaaz/Complete-DevOps.git'
    }
    stage('Sending Dockerfile to Ansible over SSH')
    {
         sshagent(['ansible']) 
        {
         sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254'
         sh 'scp /var/lib/jenkins/workspace/complete/* ubuntu@172.31.8.254:/home/ubuntu/'
        }
    }
    stage('Docker build Image')
    {
        sshagent(['ansible'])
        {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
        }
    }
     stage('Docker Image Tagging')
    {
        sshagent(['ansible'])
        {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 docker image tag $JOB_NAME:v1.$BUILD_ID navtaaz/$JOB_NAME:v1.$BUILD_ID'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 docker image tag $JOB_NAME:v1.$BUILD_ID navtaaz/$JOB_NAME:latest'
        }
    }
      stage('Pushing Image To DockerHub')
    {
        sshagent(['ansible'])
        {
            withCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) 
            {
             sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 sudo docker login -u navtaaz -p ${dockerhub_password}"
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 sudo docker image push navtaaz/$JOB_NAME:v1.$BUILD_ID'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 sudo docker image push navtaaz/$JOB_NAME:latest'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 sudo docker image rm navtaaz/$JOB_NAME:v1.$BUILD_ID'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 sudo docker image rm $JOB_NAME:v1.$BUILD_ID'
            }
        }
    }
      stage('Running Image')
    {
        sshagent(['ansible'])
        {
            withCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) 
            {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 docker rm -f complete'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.254 docker run -p 8090:80 -d --name complete navtaaz/complete:latest'
            }
        }
    }
     stage('Sending File to Minikube')
    {
        sshagent(['ansible'])
        {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.44.136'
             sh 'scp /var/lib/jenkins/workspace/complete/* ubuntu@172.31.44.136:/home/ubuntu/'
        }
    }
    stage('Deployment on Minikube')
    {
        sshagent(['ansible'])
        {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.44.136 cd /home/ubuntu'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.44.136 kubectl apply -f Deployment.yaml -f Service.yaml'
        }
    }
}
