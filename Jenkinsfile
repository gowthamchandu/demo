def version = ""
def buildInfo = ""
def rtDocker = ""
def server = ""
def ARTIFACT = "blackduck_scan"
def APINAME = "blackduck"
def LAMBDA = "Y"
def DOCKER = "Y"

pipeline {
  agent {
            label 'aws-ec2-terraform-worker-khoshal'
          }
  options{
    disableConcurrentBuilds()
  }
  
  // We prepare worker nodes to be able to work
  stages {
    stage("System preparation") {
      when {
        branch "feature/HTP*"
      }
      steps {
        sh "./bin/build_pre.sh"
      }
    }

    stage('SonarQube Analysis') {
      environment {
        scannerHome = tool 'tower'
      }
      steps {
        echo 'Analysis started...'
        withSonarQubeEnv('sonarfico') {
          sh "./bin/sonar.sh"
        }
      }
    } 

    // build lambdas and containers and push to artifactory or S3 with prefix
    stage("Build") {
      when {
        branch "feature/HTP*"
      }
      environment {
          JFROG_CREDS = credentials('svc_tower_password')
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
      }
      steps {
       sh " eval \$(aws ecr get-login --no-include-email | sed 's|https://||')"
	script {
	  if ( DOCKER == "Y" ) {
	    docker.build("towercontrol.repo.dev.aws.fico.com/${ARTIFACT}:${env.version}",'fargatetask')
	  }
	}
	sh "./bin/upload-artifacts.sh ${JFROG_CREDS} ${ARTIFACT} ${DOCKER} ${LAMBDA}"
      }
    }

//    stage("Sonar Qube Scan") {
//      when {
//          branch "feature/HTP*"
//      }
//      tools {
//        sonarQube 'SonarQube Scanner 2.8'
//      }
//      steps {
//        withSonarQubeEnv('SonarQube Scanner') {
//          sh 'sonar-scanner'
//        }
//      }
//    }




    // Images binaries security scan
    stage("Security Scan") {
      when {
          branch "feature/HTP*"
      }
      environment {
          BD_TOKEN = credentials('BD_TOKEN')
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
      }
      steps {
        //sh "./bin/security.sh ${BD_TOKEN} ${ARTIFACT} ${env.version} "
	//synopsys_detect "--detect.project.name=${ARTIFACT} --detect.project.version.name=pipeline --detect.code.location.name=${ARTIFACT}_v${env.version}"
	sh "echo a"
      }
    }

    // Virtual environment creation and deployment
    stage("Deploy to virtual Environment") {
      when {
        branch "feature/HTP*"
      }
      steps {
        script {
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
        }
        build job: "/Control_Tower/Terraform_Execute_jenkins",
	            parameters: [[$class: 'StringParameterValue',name: 'Worker_Node_To_Build', value: 'aws-ec2-terraform-worker-khoshal'],
		      [$class: 'StringParameterValue',name: 'Git_Branch', value: "origin/feature/control_tower_dev"],
		      [$class: 'StringParameterValue',name: 'Terraform_Action', value: 'apply'],
		      [$class: 'StringParameterValue',name: 'global_resources', value: 'module.control_tower.module.blackduck'],
		      [$class: 'StringParameterValue',name: 'regional_resources', value: 'none'],
		      [$class: 'StringParameterValue',name: 'prefix_terraform', value: "$version"]],
		    propagate: true,
		    wait: true
      }
    }

    // Virtual environment creation and deployment
    stage("Unit Test") {
      when {
        branch "feature/HTP*"
      }
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
	dir("ctt_store_reference") {
          sh 'PYTHONPATH=./:${PYTHONPATH} coverage run -m pytest --junitxml=../reports/ctt_store_reference-coverage.xml || true'
        }
	dir("fargatetask") {
          sh 'PYTHONPATH=./:${PYTHONPATH} coverage run -m pytest --junitxml=../reports/fargatetask-coverage.xml || true'
        }
        
      }

    }

    stage("Integration Test") {
      when {
        branch "feature/HTP*"
      }
      steps {
        sh "./bin/integration.sh"
      }
    }

    // Virtual environment is deleted after test
    stage("Delete virtual Environment") {
      when {
        branch "feature/HTP*"
      }
      steps {
        script {
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
        }
        build job: "/Control_Tower/Terraform_Execute_jenkins",
                    parameters: [[$class: 'StringParameterValue',name: 'Worker_Node_To_Build', value: 'aws-ec2-terraform-worker-khoshal'],
                      [$class: 'StringParameterValue',name: 'Git_Branch', value: "origin/feature/control_tower_dev"],
                      [$class: 'StringParameterValue',name: 'Terraform_Action', value: 'destroy'],
//                       [$class: 'StringParameterValue',name: 'Terraform_Action', value: 'plan'],
                      [$class: 'StringParameterValue',name: 'global_resources', value: 'module.control_tower.module.blackduck'],
                      [$class: 'StringParameterValue',name: 'regional_resources', value: 'none'],
                      [$class: 'StringParameterValue',name: 'prefix_terraform', value: "$version"]],
                    propagate: true,
                    wait: true
      }
    }


    stage("Promotion to develop") {
      when {
        branch "feature/HTP*"
      }
      environment {
      JFROG_CREDS = credentials('svc_tower_password')
        version = sh (
          script: 'git log --no-merges -1 --pretty=%h',
          returnStdout: true
        ).trim()
      }
      steps {
	sh "./bin/promotion.sh develop ${ARTIFACT} ${DOCKER} ${LAMBDA} ${JFROG_CREDS}"
      }
    }

    stage("Deploy to development") {
      when {
        branch "feature/HTP*"
      }
      steps {
        script {
          version = sh (
            script: 'git log --no-merges -1 --pretty=%h',
            returnStdout: true
          ).trim()
        }
        build job: "/Control_Tower/Terraform_Execute_jenkins", // test commit
                    parameters: [[$class: 'StringParameterValue',name: 'Worker_Node_To_Build', value: 'aws-ec2-terraform-worker-khoshal'],
                      [$class: 'StringParameterValue',name: 'Git_Branch', value: "origin/feature/control_tower_dev"],
                      [$class: 'StringParameterValue',name: 'Terraform_Action', value: 'apply'],
                      [$class: 'StringParameterValue',name: 'global_resources', value: 'module.control_tower.module.blackduck'],
                      [$class: 'StringParameterValue',name: 'regional_resources', value: 'none'],
                      [$class: 'StringParameterValue',name: 'prefix_terraform', value: "develop"]],
                    propagate: true,
                    wait: true
      }
    }

    stage("Promote to Staging") {
      when {
        branch "master"
      }
      steps {
	sh "./bin/promotion.sh staging ${ARTIFACT}"
      }
    }

    stage("Deploy to Staging") {
      when {
        branch "master"
      }
      steps {
        build('/MassRollout/staging/ctt_controller_${ARTIFACT}')
      }
    }

    stage("Unit Tests in Staging") {
      when {
        branch "master"
      }
      steps {
        build('/MassRollout/staging/ctt_controller_${ARTIFACT}')
      }
    }

    stage("Integration Tests in Staging") {
      when {
        branch "master"
      }
      steps {
        build('/MassRollout/staging/ctt_controller_${ARTIFACT}')
      }
    }

    stage("Performance Tests in Staging") {
      when {
        branch "master"
      }
      steps {
        build('/MassRollout/staging/ctt_controller_${ARTIFACT}')
      }
    }
    stage("Promote to prod") {
      when {
        branch "master"
      }
      steps {
        input 'Promote image to prod?'
	sh "./bin/promotion.sh production ${ARTIFACT}"
      }
    }

    stage("Deploy to Prod") {
      when {
        branch "master"
      }
      steps {
        build('/MassRollout/prod/ctt_controller_${ARTIFACT}')
      }
    }

  }
  post { 
        always { 
            junit 'reports/*.xml'
        }
    }
  
}
