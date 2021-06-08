pipeline {
	agent {
		label 'BuildServer'
	      }
	environment {
              registry = "mansig11/hangout"
              registryCredential = 'hangout'		
	         }
    stages {
        stage('compile') {
	   steps {
                echo 'compiling..'
		git url: 'https://github.com/mansgupt11/hangout-project2-sourcecode.git'
		sh script: '/opt/apache-maven-3.6.3/bin/mvn compile'
           }
        }
        stage('codereview-pmd') {
	   steps {
                echo 'codereview..'
		sh script: '/opt/apache-maven-3.6.3/bin/mvn -P metrics pmd:pmd'
           }	  	
        }
        stage('unit-test') {
	   steps {
                echo 'unittest..'
	        sh script: '/opt/apache-maven-3.6.3/bin/mvn test'
                 }
	   post {
               success {
                   junit 'target/surefire-reports/*.xml'
               }
           }			
        }
        stage('codecoverate') {
	   steps {
                echo 'codecoverage..'
		sh script: '/opt/apache-maven-3.6.3/bin/mvn cobertura:cobertura -Dcobertura.report.format=xml'
           }
	   post {
               success {
	           cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false                  
               }
           }		
        }
        stage('package') {
	   steps {
                echo 'package..'
		sh script: '/opt/apache-maven-3.6.3/bin/mvn package'	
           }		
        }
        stage ('Building a Docker image'){
        steps {
            script {
          docker.build registry + ":$BUILD_NUMBER"
        }
        }
        }
        stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            sh 'docker push $registry:$BUILD_NUMBER'
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    stage ('Deploy to Docker') {
   agent {
        label 'DockerServer'
    }
      steps {
	      sh "docker run -p 8080:8080 -d $registry:$BUILD_NUMBER"
        sh "docker ps -a"
       }           
             
     } 
  }
}
