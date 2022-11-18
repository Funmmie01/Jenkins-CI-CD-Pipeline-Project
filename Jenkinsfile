def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]



pipeline {
    agent any
    
    
    environment{
        
        WORKSPACE = "${env.WORKSPACE}"
        
    }

    
    
    
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }
    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/Funmmie01/Jenkins-CI-CD-Pipeline-Project.git'
                sh 'mvn --version'
               

            }
        }
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
                
            }
            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        stage('Unit Test'){
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage ('SonarQube scanning'){
            steps {
                withSonarQubeEnv('SonarQube') {
                sh """
                mvn sonar:sonar \
                  -Dsonar.projectKey=jenkins \
                  -Dsonar.host.url=http://172.31.94.131:9000 \
                  -Dsonar.login=41a25873ee3d4dad358a18720cef932f52f62e1d
                """
                }
            }
        }
        stage("Quality Gate"){
              steps{
               waitForQualityGate abortPipeline: true
                 }
    }
         stage("Upload artifact to Nexus"){
            steps{
             sh 'mvn clean deploy -DskipTests'
         }
    }
    
         stage('Deploy to DEVING') {
          environment {
            HOSTS = "dev"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
         
          stage('Deploy to STAGE env') {
          environment {
            HOSTS = "stage"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
         
         
          stage('Approval') {
          steps {
           input('Do you want to proceed?')
          }
        }

         
          stage('Deploy to PROD env') {
          environment {
            HOSTS = "prod"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
    
    
    }
    
    
    post { 
        always { 
            echo 'I will always say Hello again!'
            slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }

//slackSend channel: '#mbandi-cloudformation-cicd', message: "Please find the pipeline status of the following ${env.JOB_NAME ${env.BUILD_NUMBER} ${env.BUILD_URL}"
