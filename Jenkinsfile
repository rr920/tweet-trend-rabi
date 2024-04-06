def registry = 'https://reporabi01.jfrog.io/'
def imageName = 'reporabi01.jfrog.io/rabi-docker-local/ttrend'
def version   = '2.1.4'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
    }

    stages {
        stage("Build") {
            steps {
                echo "--------------------Build Started---------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "-------------------Build Completed--------------------"
            }
        }

        stage("Test") {
            steps {
                echo "-------------------Unit Test Started------------------"
                sh 'mvn surefire-report:report'
                echo "------------------Unit Test Completed----------------"
            }
        }

        // stage("SonarQube Analysis") {
        //     environment {
        //         scannerHome = tool 'rabi-sonar-scanner'
        //     }
        //     steps {
        //         withSonarQubeEnv('rabi-sonarqube-server') {
        //         sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }

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
                              "target": "ttrend-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5" ]
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
        
        stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

        stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'jfrog-cred'){
                    app.push()
                echo '<--------------- Docker Publish Ended --------------->'

                }         
            }
        }

        stage("Deplying to EKS") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            sh './deploy.sh'
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
     } 
    }   
    }
}

