node {

    def mvnHome = tool 'Maven3'
    stage ("checkout")  {
       checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '8a2a23c8-40a3-4976-b94c-aa73c9fb1e1f', url: 'https://github.com/akhil5672/my-application-demo']])
    }

   stage ('build')  {
    sh "${mvnHome}/bin/mvn clean install -f MyWebApp/pom.xml"
    }

     stage ('Code Quality scan')  {
       withSonarQubeEnv('Sonarqube') {
       sh "${mvnHome}/bin/mvn -f MyWebApp/pom.xml sonar:sonar"
        }
   }
  
   stage ('Code coverage')  {
       jacoco()
   }

   stage ('Nexus upload')  {
        nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: '18.218.31.0:8081/',
        groupId: 'myGroupId',
        version: '1.0-SNAPSHOT',
        repository: 'maven-snapshots',
        credentialsId: '16f44f50-2530-4d89-8dc4-96960d465874',
        artifacts: [
            [artifactId: 'MyWebApp',
             classifier: '',
             file: 'MyWebApp/target/MyWebApp.war',
             type: 'war']
        ]
     )
    }
   
   stage ('DEV Deploy')  {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '452e77a1-3214-4231-8469-63591565d19c', path: '', url: 'http://3.16.81.232:8080')], contextPath: null, war: '**/*.war'

    }

  stage ('Slack notification')  {
    slackSend(channel:'slack-integration-alerts', message: "Job is successful, here is the info -  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
   }

   stage ('DEV Approve')  {
            echo "Taking approval from DEV Manager for QA Deployment"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve QA Deployment?', submitter: 'admin'
            }
     }

stage ('QA Deploy')  {
     echo "deploying into QA Env " 
deploy adapters: [tomcat9(credentialsId: '452e77a1-3214-4231-8469-63591565d19c', path: '', url: 'http://3.16.81.232:8080')], contextPath: null, war: '**/*.war'

}

  stage ('QA notify')  {
    slackSend(channel:'slack-integration-alerts', message: "QA Deployment was successful, here is the info -  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
   }

stage ('QA Approve')  {
    echo "Taking approval from QA manager"
    timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD Deploy?', submitter: 'admin,manager_userid'
  }
}

stage ('PROD Deploy')  {
     echo "deploying into PROD Env " 
deploy adapters: [tomcat9(credentialsId: '452e77a1-3214-4231-8469-63591565d19c', path: '', url: 'http://3.16.81.232:8080')], contextPath: null, war: '**/*.war'

}
}
