pipeline {
  agent any
  parameters {
    // string(name: 'DOCKER_BASE', defaultValue: 'ghidra-base')
    string(name: 'DOCKER_BUILD', defaultValue: 'ghidra-build')
    string(name: 'DOCKER_TEST', defaultValue: 'ghidra-test')
    string(name: 'DOCKER_BASE', defaultValue: 'ghidra-base')
  }
  stages {
    stage('Build') {
      agent any
      steps {
        sh "docker build -t ${params.DOCKER_BASE} --target base ."
        sh "docker build -t ${params.DOCKER_BUILD} --target build ."
      }
    }
    stage('Test') {
      agent any
      steps {
        sh "docker build -t ${params.DOCKER_TEST} --target test ."
      }
    }
    stage('Deploy') {
      agent any
      script {
        def containerId = sh(script: "docker run -d ${params.DOCKER_NAME} tail -f /dev/null", returnStdout: true).trim()
        sh "docker cp ${containerId}:/ghidra/build/dist/ghidra.zip ."
        sh "docker stop ${containerId}"
        sh "docker rm ${containerId}"
      }
    }
    stage('Publish') {
      agent any
      steps {
        sh 'echo nice'
      }
    }
  }
  post {
        success {
            // Archive the extracted artifact and attach it to the Jenkins build
            archiveArtifacts artifacts: 'ghidra.zip', allowEmptyArchive: true
        }
        
        cleanup {
            // Clean up Docker resources
            script {
                docker.image("${params.DOCKER_BUILD}").remove()
                docker.image("${params.DOCKER_TEST}").remove()
            }
        }
    }
}