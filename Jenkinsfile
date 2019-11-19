
node {
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def server = Artifactory.server "artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    
    tools {
    maven 'maven'
  }

    stage('Clone sources') {
        git url: 'https://github.com/duorg/webapp.git'
    }

    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "maven"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
    }

    stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
    }

    stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage('build & SonarQube Scan') {
      withSonarQubeEnv('sonar') {
      sh 'mvn clean package sonar:sonar'
    } // SonarQube taskId is automatically attached to the pipeline context
  }
  
   stage("Quality Gate") {
   timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
     def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
     if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
  }
    }
	 
