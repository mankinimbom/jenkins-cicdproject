pipeline {
  agent any
  stages {
    stage('ContinuesDownload') {
      steps {
        script{
          try {
            git 'https://github.com/mankinimbom/maven.git'
          } catch(Exception e1) {
            emailext body: 'Jenkins was unable to download the code from the GitHub repository', to: 'ankinimbommichaely@gmail.com', subject: 'Code Download failed'
            error("Git clone failed")
          }
        }
      }
    }
    stage('ContinuesBuild') {
      steps {
        script{
          try {
            sh 'mvn package'
          } catch(Exception e2) {
            emailext body: 'Jenkins was unable to build the code', to: 'ankinimbommichaely@gmail.com', subject: 'Code Build failed'
            error("Maven build failed")
          }
        }
      }
    }
    stage('UploadArtifactTonexus') {
      steps {
        script{
          try {
             nexusArtifactUploader (
            artifacts: [[
                artifactId: 'webapp', 
                classifier: '', 
                file: '/home/prodserver/workspace/workspace/pipelineproject/webapp/target/webapp.war', 
                type: 'war'
            ]], 
            credentialsId: 'nexus', 
            groupId: 'webapp', 
            nexusUrl: 'http://10.0.0.39:8081', 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'maven-snapshots', 
            version: '1.0-SNAPSHOT' )
          } catch(Exception e2) {
            emailext body: 'Jenkins was unable to upload the artifact to Nexus', to: 'ankinimbommichaely@gmail.com', subject: 'Upload Artifact failed'
            error("Upload Artifact failed")
          }
        }
      }
    }
    stage('SonarQubeAnalysis') {
      steps {
        script{
          try {
            withSonarQubeEnv(credentialsId: 'sonarqube') {
              sh 'mvn clean verify sonar:sonar'
            }
          } catch(Exception e) {
            emailext body: 'SonarQube analysis failed', to: 'ankinimbommichaely@gmail.com', subject: 'SonarQube Analysis failed'
            error("SonarQube Analysis failed")
          }
        }
      }
    }
    stage('DeployToQAT') {
      steps {
        script{
          try {
            deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://10.0.0.39:8080/')], contextPath: 'devops', war: '**/*.war'
          } catch(Exception e3) {
            emailext body: 'Jenkins was unable to deploy the code to tomcat QA', to: 'ankinimbommichaely@gmail.com', subject: 'Deploy to QA failed'
            error("Deployment to QA failed")
          }
        }
      }
    }
    stage('ContinuesTesting') {
      steps {
        script{
          try {
            git 'https://github.com/mankinimbom/testingproject.git'
            sh 'java -jar /home/prodserver/workspace/workspace/pipelineproject/testing.jar'
          } catch(Exception e4) {
            emailext body: 'Jenkins was unable to run the tests', to: 'ankinimbommichaely@gmail.com', subject: 'Code Testing failed'
            error("Testing failed")
          }
        }
      }
    }
    stage('ContinuesDeployToProd') {
     steps {
      script{
        try {
        // Wait for approval
        input message: 'Approve deployment?', ok: 'Approve', submitter: 'mike'
        
        // If approved, continue with deployment
        deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://10.0.0.39:8080/')], contextPath: 'devops', war: '**/*.war'
      } catch(Exception e5) {
        emailext body: 'Jenkins was unable to deploy the build artifact to PROD', to: 'ankinimbommichaely@gmail.com', subject: 'Code Deploy to Prod failed'
        error("Deployment to PROD failed")
      }
    }
  }
}
    stage('Check Monitoring Services') {
      steps {
        script{
          try {
            sh '''
            echo "Checking Prometheus..."
            curl -f http://10.0.0.39:9090/-/healthy
            echo "Checking Grafana..."
            curl -f http://10.0.0.39:3000/api/health
            '''
          } catch(Exception e) {
            emailext body: 'Monitoring services are not available', to: 'ankinimbommichaely@gmail.com', subject: 'Monitoring services check failed'
            error("Monitoring services check failed")
          }
        }
      }
    }
    stage('Notify Slack') {
      steps {
        catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
          slackSend (
            channel: "#random",
            color: "good",
            message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) ran successfully"
          )
        }
      }
      post {
        failure {
          slackSend (
            channel: "#random",
            color: "danger",
            message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) failed"
          )
        }
      }
    }
  }
}
