pipeline {
  agent any 
  tools {
    maven 'Maven'
  }  
  stages {
   stage ('Initialize') {
      steps {
        sh '''
                  echo "PATH = ${PATH}"
                  echo "M2_HOME = ${M2_HOME}"
           '''
      }
    }
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog3 || true'
        sh 'trufflehog3 https://github.com/electro-16/webapp.git -f json > tufflehog3 '
      }
    }
   stage ('Build') {
     steps {
       sh 'mvn clean package'
     }
    }
    stage ('Deploy-To-Tomcat') {
      steps  {
        sshagent (['tomcat']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.87.48.212:/prod/apache-tomcat-8.5.98/webapps/webapp.war'
        }
      }
  }
}
}
