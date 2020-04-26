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
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@3.134.86.92:/opt/tomcat/webapps/webapp.war'
              }      
           }       
    } 
    stage ('Check Website health') {
           steps {
           sshagent(['zap']) {
             sh 'ssh -o  StrictHostKeyChecking=no ec2-user@18.188.81.79 "curl -Is http://3.134.86.92:8080/webapp/ | head -n 1" || true'
          }
        }
    }
   stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@18.188.81.79 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.134.86.92:8080/webapp/" || true'
        }
      }
    }  
  stage ('NMAP Port Scanner') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@18.188.81.79 "docker run --rm uzyexe/nmap -F -A 3.134.86.92" || true'
        }
      }
    } 
     }
}
    post {
        always {
                slackSend channel: '#cicdpipeline-tomcatdeployment',
                    color: 'good',
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
        
        }
    }
 
