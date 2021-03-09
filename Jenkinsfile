

pipeline {
  agent any
  stages{
    stage("temp"){
      steps{
        echo "Hello gowtham"
        sh '''
        echo " " >> reports/abc.xml
        '''
      }
    
    }
  }
  post { 
        always { 
            junit 'reports/abc.xml'
        }
    }
  
}
