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
    
  
  stage ('Static analysis') {
    steps { 
      withSonarQubeEnv('sonar') {
        sh 'mvn sonar:sonar'
	
        }
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
          sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.86.253.216:/prod/apache-tomcat-8.5.98/webapps/webapp.war'
        }
      }
  }

	 stage ('Dynamic analysis') {
            steps {
           sshagent(['zap']) {
                sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.86.153.21 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://3.86.253.216:8080/webapp -x zap_report  || true" '
		   // sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.91.196.22 "sudo docker run -t owasp/zap2docker-stable zap-baseline.py -t http://44.203.175.196:8080/webapp" '
		
              }      
           }       
    }	  
	  
}
}



