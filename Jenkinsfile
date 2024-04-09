def registry = 'https://ravikant1983.jfrog.io'
def imageName = "ravikant1983.jfrog.io/rkg1983-docker.local/rkg1983"
def version = "2.1.2"


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
        tools {
            jdk 'java17'
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

    stage("Quality Gate"){
    steps {
        script {
        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
}
    }
  }


           stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "rkg1983-libs-release-local//{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }   

        stage ("Docker Build"){
            steps {
                script {
                    echo "...........Docker Build Started........."
                    app = docker.build(imageName+":"+version)
                    echo "...........Docker Build Completed........."

                }
            }
        }

        stage ("Docker Publish"){
            steps{
                script{
                    echo ".........Docker publish started..... "
                    docker.withRegistry(registry, 'jfrog-cred'){
                    app.push()
                    }
                    echo ".........Docker publish Completed...."
                }
            }
        }

      }
    }
