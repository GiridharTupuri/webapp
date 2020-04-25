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
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
      stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@18.220.45.176:/opt/tomcat/webapps/webapp.war'
              }      
           }       
    }
  }
}
