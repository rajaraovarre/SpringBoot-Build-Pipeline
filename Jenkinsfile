pipeline {
  agent { label 'build' }
   environment { 
        registry = "adamtravis/democicd" 
        registryCredential = 'dockerhub' 
   }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'release', credentialsId: 'GitlabCred', url: 'https://gitlab.com/learndevopseasy/devsecops/springboot-build-pipeline.git'
      }
    }
  
   stage('Stage I: Build') {
      steps {
        echo "Building Jar Component ..."
        sh "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64; mvn clean package "
      }
    }

   stage('Stage II: Code Coverage ') {
      steps {
	    echo "Running Code Coverage ..."
      }
    }

   stage('Stage III: SCA') {
      steps { 
        echo "Running Software Composition Analysis using OWASP Dependency-Check ..."
      }
    }

   stage('Stage IV: SAST') {
      steps { 
        echo "Running Static application security testing using SonarQube Scanner ..."
      }
    }

   stage('Stage V: QualityGates') {
      steps { 
        echo "Running Quality Gates to verify the code quality"
      }
    }
   
   stage('Stage VI: Build Image') {
      steps { 
        echo "Build Docker Image"
        script {
               docker.withRegistry( '', registryCredential ) { 
                 myImage = docker.build registry + ":rel$BUILD_NUMBER" 
                 myImage.push()
                }
        }
      }
    }

   stage('Stage VII: Scan Image ') {
      steps { 
        echo "Scanning Image for Vulnerabilities"
        }
    }
          
   stage('Stage VIII: Smoke Test ') {
      steps { 
        echo "Smoke Test the Image"
        }
    }

    stage('Stage IX: Trigger Deployment'){
      steps { 
       script {
        TAG = "rel$BUILD_NUMBER"
         echo "Trigger CD Pipeline"
          build wait: false, job: 'WEZVATECH-SAMPLE-DEPLOYMENT-PIPELINE', parameters: [string(name: 'IMAGETAG', value: TAG), string(name: 'environment', value: 'uat')]
       }
      }
    }


  }
}
