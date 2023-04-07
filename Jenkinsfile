pipeline {
agent any 
tools {
maven 'maven'
}

stages {
stage("Git Checkout"){
steps{
git 'https://github.com/shaileshbora/star-agile-health-care.git'
 }
 }
stage('Build the application'){
steps{
echo 'cleaning..compiling..testing..packaging..'
sh 'mvn clean install package'
 }
 }
 
stage('Publish HTML Report'){
steps{
  publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Medicure_Project/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
 }
}
stage('Docker build image') {
              steps {
                
                  sh'sudo docker system prune -af '
                  sh 'sudo docker build -t shaileshbora/medicure_project:1.0 .'
              
                }
            }
stage('Push Image to DockerHub') {
      steps {
     withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
      sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
      sh 'sudo docker push shaileshbora/insure_me_project1:1.0'
      }
      }
      }
 stage (' setting up Kubernetes with terraform '){
            steps{

                dir('terraform_files'){
                sh 'terraform init'
                sh 'terraform validate'
                sh 'terraform apply --auto-approve'
                sh 'sleep 20'
                }
               
            }
        }
stage('deploy the application to kubernetes'){
steps{
  sh 'sudo chmod 600 ./terraform_files/project.pem'    
  sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/project.pem medicure-deployment.yml ubuntu@172.31.21.225:/home/ubuntu/'
  sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/project.pem medicure-service.yml ubuntu@172.31.21.225:/home/ubuntu/'
script{
  try{
  sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/project.pem ubuntu@172.31.21.225 kubectl apply -f .'
  }catch(error)
  {
  sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/project.pem ubuntu@172.31.21.225 kubectl apply -f .'
  }
}
}
}
}
}
