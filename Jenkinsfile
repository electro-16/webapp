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
        sh ' trufflehog3 -f json https://github.com/electro-16/webapp.git -o trufflehog_output.json || true '
      }
    }
    // stage ('Source Composition Analysis') {
    //   steps {
    //      sh 'rm owasp* || true '
    //      sh 'wget "https://raw.githubusercontent.com/electro-16/webapp/master/owasp-dependency-check.sh" '
    //      sh 'chmod +x owasp-dependency-check.sh'
    //      sh 'bash owasp-dependency-check.sh'
    //      sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml' 
    //   }
    // }

   
    stage('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
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
          sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@44.204.71.64:/prod/apache-tomcat-8.5.98/webapps/webapp.war'
        }
      }
  }
}
}
