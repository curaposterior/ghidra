pipeline {
  agent any
  parameters {
    string(name: 'DOCKER_BUILD', defaultValue: 'ghidra-build')
    string(name: 'DOCKER_TEST', defaultValue: 'ghidra-test')
    string(name: 'DOCKER_BASE', defaultValue: 'ghidra-base')
    string(name: 'DOCKER_DEPLOY', defaultValue: 'ghidra-deploy')
  }
  stages {
    stage('Build') {
      agent any
      steps {
        sh "docker build -t ${params.DOCKER_BASE} --no-cache --target=base ."
        sh "docker build -t ${params.DOCKER_BUILD} --target=build ."
      }
    }
    stage('Test') {
      agent any
      steps {
        sh "docker build -t ${params.DOCKER_TEST} --target=test ."
      }
    }
    stage('Deploy') {
      agent any
      steps {
        sh "docker build -t ${params.DOCKER_DEPLOY} --target=deploy ."
      }
    }
    stage('Publish') {
      agent any
      steps {
        script {
          def containerId = sh(script: "docker run --rm -d ${params.DOCKER_BUILD} tail -f /dev/null", returnStdout: true).trim()
          sh "docker cp ${containerId}:/ghidra/build/dist/ghidra_*.zip ."
          sh "mv ghidra_*.zip ghidra.zip"
          sh "docker stop ${containerId}"
        }
      }
    }
  }
  post {
        success {
            archiveArtifacts artifacts: 'ghidra.zip', allowEmptyArchive: true
        }
        
        cleanup {
            script {
                docker.image("${params.DOCKER_BASE}").remove()
                docker.image("${params.DOCKER_BUILD}").remove()
                docker.image("${params.DOCKER_TEST}").remove()
            }
        }
    }
}