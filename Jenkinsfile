

pipeline {
  agent any
  
  // We prepare worker nodes to be able to work

  post { 
        always { 
            junit 'reports/*.xml'
        }
    }
  
}
