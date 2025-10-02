pipeline {
  agent any

  environment { 
    registry = "dark5980/devops" 
    registryCredential = 'dockercri' 
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://gitlab.com/udaykumar5980/springboot-build-pipeline.git'
      }
    }

    stage('Stage I: Build') {
      steps {
        echo "Building Jar Component ..."
        dir('/tmp/springboot-build-pipeline') {
          sh "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64; mvn clean package"
        }
      }
    }

    stage('Stage II: Code Coverage') {
      steps {
        echo "Running Code Coverage ..."
        dir('/tmp/springboot-build-pipeline') {
          sh "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64; mvn jacoco:report"
        }
      }
    }

    stage('Stage III: SAST') {
      steps { 
        echo "Running Static application security testing using SonarQube Scanner ..."
        dir('/tmp/springboot-build-pipeline') {
          withSonarQubeEnv('sonar') {
            sh 'mvn sonar:sonar -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.projectName=devopsproject'
          }
        }
      }
    }

    stage('Stage IV: QualityGates') {
      steps { 
        echo "Running Quality Gates to verify the code quality"
        dir('/tmp/springboot-build-pipeline') {
          script {
            timeout(time: 1, unit: 'MINUTES') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          }
        }
      }
    }

    stage('Stage V: Build Image') {
      steps { 
        echo "Build Docker Image"
        dir('/tmp/springboot-build-pipeline') {
          script {
            docker.withRegistry('', registryCredential) {
              def myImage = docker.build(registry)
              myImage.push()
            }
          }
        }
      }
    }

    stage('Stage VI: Scan Image') {
      steps { 
        echo "Scanning Image for Vulnerabilities"
        sh "trivy image --scanners vuln --offline-scan dark5980/devops > trivyresults.txt"
      }
    }

    stage('Stage VII: Smoke Test') {
      steps { 
        echo "Smoke Test the Image"
        sh "docker run -d --name smokerun -p 9091:9091 dark5980/devops"
        sh "sleep 90; ./check.sh"
        sh "docker rm --force smokerun"
      }
    }
  }
}
