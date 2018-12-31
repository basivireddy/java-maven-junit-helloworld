pipeline {
    agent any
    stages {
        stage('Checkout') {
	    steps {
            git url: 'https://github.com/basivireddy/java-maven-junit-helloworld.git', credentialsId: 'github-basivireddy', branch: 'master'
	    }
	}

        stage('Clean and compile') {
	    steps {
	      compile("maven", "compile");
	    }
	      }
        
       stage('SonarQube analysis') {
	       steps {
		    checkCodeQuality("sonarqube 6.7.6");
	       }
       }

        stage('Jacoco test') {
		steps {
	      jacocotest("maven", "jacoco:report");
		}
	      }
        
        stage('Package') {
		steps {
	      buildpackage("maven", "package");
		}
	}

        stage('Push artifacts to Nexus2') {
		steps {
	      nexus2("maven");
		}
	    }

        /* stage('Docker Build and Publish') {
	 buildAndPush('basivireddy', "${env.JOB_NAME}", 'v1');
        }
    
         stage ('Final') {
            build job: 'gateway-service-pipeline', wait: false
        }      
        */
    }
}
def compile(mvnVersion, task){
	def mvnHome = tool name: "${mvnVersion}"
	env.PATH = "${mvnHome}/bin:${env.PATH}"
  sh "mvn -B -f pom.xml clean ${task}"
}
def checkCodeQuality(sonarVersion){
	withSonarQubeEnv("${sonarVersion}") { // SonarQube taskId is automatically attached to the pipeline context
        sh "mvn -B -f pom.xml sonar:sonar" 
    }
}
def jacocotest(mvnVersion, task){
	def mvnHome = tool name: "${mvnVersion}"
	env.PATH = "${mvnHome}/bin:${env.PATH}"
	sh "mvn -B -f pom.xml ${task}"
     
    publishHTML (target: [
      allowMissing: false,
      alwaysLinkToLastBuild: false,
      keepAll: true,
      reportDir: 'target/site/jacoco',
      reportFiles: 'index.html',
      reportName: "RCov Report"
    ])
}

def buildpackage(mvnVersion, task){
	def mvnHome = tool name: "${mvnVersion}"
	env.PATH = "${mvnHome}/bin:${env.PATH}"
        sh "mvn -B -f pom.xml ${task}"
}

def nexus2(mvnVersion){
   def mvnHome = tool name: "${mvnVersion}"
   env.PATH = "${mvnHome}/bin:${env.PATH}"
   sh "mvn -B -f pom.xml deploy:deploy-file -DgroupId=com.vimo -DartifactId=java-maven-junit-helloworld -Dversion=2.0-SNAPSHOT -DgeneratePom=true -Dpackaging=jar -DrepositoryId=nexus -Durl=http://localhost:8081/nexus/content/repositories/snapshots/ -Dfile=target/java-maven-junit-helloworld.jar"
}

def buildAndPush(registryID, imageName, version){
    def app
    try {
	    app = docker.build("${registryID}/${imageName}")
		docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
		    app.push("${version}.${env.BUILD_NUMBER}")
		}
	} catch (error){
		sh 'echo "Not able to build and push docker image ${error}"'
	}
}
