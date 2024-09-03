node {

    def mvnHome = tool 'Maven3'
    stage ("checkout")  {
       checkout scmGit(branches: [[name: '*/Develop']], extensions: [], userRemoteConfigs: [[credentialsId: 'fee80334-c226-4b46-bc10-c5f9e156408f', url: 'https://github.com/satyavolu777/Devops_Aug24.git']])
    }

   stage ('build')  {
    sh "${mvnHome}/bin/mvn clean install -f MyWebApp/pom.xml"
    }
	
    stage ('Code Quality scan')  {
       withSonarQubeEnv('SonarQube') {
        sh "${mvnHome}/bin/mvn -f MyWebApp/pom.xml sonar:sonar"
        }
   }
   
     stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }             
 
  
   stage ('Code coverage')  {
       jacoco()
   }

   
   
   stage ('DEV Deploy')  {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '136cdd6f-db57-4497-a037-73f7e0a23959', path: '', url: 'http://ec2-13-235-149-20.ap-south-1.compute.amazonaws.com:8080')], contextPath: null, onFailure: false, war: '**/*.war'

    }

  stage ('Slack notification')  {
    slackSend channel: '#devops-prj', message: 'Job is successful, here is the info -  Job \'${env.JOB_NAME} [${env.BUILD_NUMBER}]\' (${env.BUILD_URL})', teamDomain: 'Devops_Aug24', tokenCredentialId: '980de285-d778-41de-9bad-07be28ba6729'
   }

   stage ('DEV Approve')  {
            echo "Taking approval from DEV Manager for QA Deployment"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve QA Deployment?', submitter: 'admin'
            }
     }

stage ('QA Deploy')  {
     echo "deploying into QA Env " 
deploy adapters: [tomcat9(credentialsId: '136cdd6f-db57-4497-a037-73f7e0a23959', path: '', url: 'http://ec2-13-235-149-20.ap-south-1.compute.amazonaws.com:8080')], contextPath: null, onFailure: false, war: '**/*.war'

}
}
