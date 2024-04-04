pipeline {
    agent any
    tools{
        nodejs 'node16'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {

      stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }

      
           stage('Git-Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Piyush6042/Game-2048.git'
            }
        }
        

        

         stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-qube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game-2048 \
                    -Dsonar.sources=. \
                    -Dsonar.projectKey=2048-Game '''
                }
            }
        }
        stage("SonarQube Quality Gate"){
       
             steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
         }

     }
     
       
      

      stage("OWASP Dependency Check"){
           steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
         stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                    
                }
            }
        }


        stage("Build and Push to Docker Hub"){
               steps{
                   
                echo 'login into docker hub and pushing image....'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubPassword', usernameVariable: 'dockerhubUser')]){
                     sh "docker build -t game-2048:latest -f backend/Dockerfile ."
                     sh "docker tag game-2048 ${env.dockerhubUser}/game-2048:latest"
                     sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPassword}"
                     sh "docker push ${env.dockerhubUser}/game-2048:latest"


               }
           }
        }


         stage("TRIVY Docker Scan"){
            steps{

                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubPassword', usernameVariable: 'dockerhubUser')]){
                     
                    sh "trivy image ${env.dockerhubUser}/game-2048:latest" 

               }
            }
        }
        
         stage('Deploy to Kubernetes Cluster') {
            steps {
                script {withKubeConfig([credentialsId: 'K8', serverUrl: '']) {
                        sh ('kubectl apply -f deployment.yaml')
                    }
                }
            }
        }
    }
}
