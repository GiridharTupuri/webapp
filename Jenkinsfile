pipeline {
  agent any 
  tools {
    maven 'M2_HOME'
  }
  stages {
    stage ('Initialize-Maven') {
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
    stage ('Source-Composition-Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/GiridharTupuri/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }
    stage ('Build-War-File') {
      steps {
      sh 'mvn clean package'
       }
    }
      stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@3.23.79.64:/opt/tomcat/webapps/webapp.war'
              }      
           }       
    } 
    stage ('Check-Website-Health') {
           steps {
           sshagent(['zap']) {
             sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.23.113.217 "curl -Is http://3.23.79.64:8080/webapp/ | head -n 1" || true'
          }
        }
    }
   stage ('Dynamic-Application-Security-Testing') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.23.113.217 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.23.79.64:8080/webapp/" || true'
        }
      }
    }  
  stage ('NMAP-Port-Scanner') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.23.113.217 "docker run --rm uzyexe/nmap -F -A 3.23.79.64" || true'
        }
      }
    } 
  stage ('Karata_API_Test_Execution') {
      steps {
      build "KarataFramework_Executer"
       }   
  }
  }
post {
      success {
          slackSend channel: '#ci-cdjenkinspipelineexecutionbuild',
                    color: 'good',
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
      }
      failure {
          slackSend channel: '#ci-cdjenkinspipelineexecutionbuild',
                    color: 'danger',
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
        }
    }  
  }
