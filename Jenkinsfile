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
    }
}

