pipeline {
  agent any

  tools {
  maven 'Maven3'
  }
  stages {
    stage ('Build') {
      steps {
      sh 'mvn clean install -f MyWebApp/pom.xml'
      sh 'echo Code Quality scan'{
      sh 'withSonarQubeEnv('SonarQube')' {
      sh "${mvnHome}/bin/mvn -f MyWebApp/pom.xml sonar:sonar"
       }
     }
   
     sh 'echo Quality Gate'{
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
       }
     }
     }	
    }	
   stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '136cdd6f-db57-4497-a037-73f7e0a23959', path: '', url: 'http://ec2-13-201-192-227.ap-south-1.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'#devops-prj', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
     stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat9(credentialsId: '136cdd6f-db57-4497-a037-73f7e0a23959', path: '', url: 'http://ec2-13-201-192-227.ap-south-1.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
        }
    }
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }
    stage ('Slack Notification for QA Deploy') {
      steps {
        echo "deployed to QA Env successfully"
        slackSend(channel:'#devops-prj', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
 }
}
