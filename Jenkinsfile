pipeline {
    agent {label 'maven'}

environment {
    PATH = "/opt/apache-maven-3.8.8/bin:$PATH"
}
    stages {
        stage("build"){
            steps {
                 echo "----------- build startedddddd----------"
                sh 'mvn clean deploy'
                 echo "----------- build complted ----------"
           }
         }
	         stage("test"){
            steps{
                echo "----------- unit test started ----------"
                sh 'mvn surefire-report:report'
                 echo "----------- unit test Complted ----------"
            }
        }

    stage('SonarQube analysis') {
	    {
        agent {
            docker { image 'openjdk:17-alpine' }
        }

	    
    environment {
      scannerHome = tool 'sonar-scanner'
    }
    steps{
    withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
      sh "${scannerHome}/bin/sonar-scanner"
    }
    }
  }

      }
    }
