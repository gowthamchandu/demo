

pipeline {
  agent any
  stages{
    stage("temp"){
      steps{
        echo "Hello gowtham"
      }
    
    }
  }
  post { 
        always { 
            junit 'reports/abc.xml'
        }
    }
  
}
