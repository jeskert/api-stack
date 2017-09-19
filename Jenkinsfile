pipeline {
    agent any
    stages {
        stage ('Checkout project') {
            steps {
                git url: 'https://github.com/zjx-immersion/api-stack.git'
            }
        }
        stage ('Check env') {
            steps {
                sh "java -version"
            }
        }
        stage ('Build and Unit Test') {
            steps {
                sh "./gradlew clean build"
                dir("build") {
                    sh "ls -a"
                }
                junit 'build/test-results/test/TEST-*.xml'
                stash excludes: 'src/', includes: '**', name: 'source'
            }
        }
        stage ('API Test') {
            steps {
                sh "./gradlew apiTest"
                dir("build") {
                    sh "ls -a"
                }
            }
        }
        stage ('package') {
            steps {
                sh "./gradlew bootRepackage"
                sh "ls -a"

                archive includes: "build/libs/*.jar"
                archive includes: "build/*"
            }
        }
        stage ('Artifact') {
            steps {
                step([$class: 'ArtifactArchiver', artifacts: '**/build/libs/*.jar', fingerprint: true])
            }
        }
        stage ('docker clear') {
            steps {
                script {
                    sh 'chmod +x ./src/main/docker/clear.sh'
                    POM_VERSION = sh(script: "./src/main/docker/clear.sh", returnStdout: true)
                    echo "${POM_VERSION}"
                }
                sh "./gradlew buildDocker"
            }
        }
        stage ('Deploy') {
            steps {
                sh "echo 'Run API Server in Container'"
                sh "docker run --name=api-server -d -p 5000:8082 zhongjx/api-stack "
                deleteDir()
            }
        }
    }
}

def archiveUnitTestResults() {
    step([$class: "JUnitResultArchiver", testResults: "build/**/TEST-*.xml"])
}

def archiveCheckstyleResults() {
    step([$class         : "CheckStylePublisher",
          canComputeNew  : false,
          defaultEncoding: "",
          healthy        : "",
          pattern        : "build/reports/checkstyle/main.xml",
          unHealthy      : ""])
}