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
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/GiridharTupuri/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    } 
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/GiridharTupuri/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
      stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@3.17.204.5:/opt/tomcat/webapps/webapp.war'
              }      
           }       
    }
   stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.22.119.5 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.17.204.5:8080/webapp/" || true'
        }
      }
    }  
  stage ('NMAP Port Scanner') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.22.119.5 "docker run --rm uzyexe/nmap -p 80 3.17.204.5" || true'
        }
      }
    }  
  }
}
