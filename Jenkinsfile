

pipeline {
  agent any
  
  // We prepare worker nodes to be able to work
  stages {
    stage("System preparation") {
      
      steps {
        sh "sudo pip install coverage boto3 pytest pytest-cov"
      }
    }
    // Virtual environment creation and deployment
    stage("Unit Test") {
      
      steps {
        script {
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
        }
        //sh "echo unittest"
        sh "./bin/unit-test.sh  ${ARTIFACT} ${APINAME}"
	sh "git submodule update --init --recursive"
        dir("ctt_get_scan_results") {
          sh 'PYTHONPATH=./:${PYTHONPATH} coverage run -m pytest --junitxml=../reports/ctt_get_scan_results-coverage.xml || true'
        }
      }

    }

    
  }
  post { 
        always { 
            junit 'reports/*.xml'
        }
    }
  
}
