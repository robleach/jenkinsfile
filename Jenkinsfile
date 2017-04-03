// see https://dzone.com/refcardz/continuous-delivery-with-jenkins-workflow for tutorial
// see https://documentation.cloudbees.com/docs/cookbook/_pipeline_dsl_keywords.html for dsl reference
// This Jenkinsfile should simulate a minimal Jenkins pipeline and can serve as a starting point.
// NOTE: sleep commands are solelely inserted for the purpose of simulating long running tasks when you run the pipeline
node('master') {
   // Mark the code checkout 'stage'....
   stage 'checkout'

   // Get some code from a GitHub repository
   git url: 'https://github.com/robleach/jenkinsfile.git'
   sh 'git clean -fdx; sleep 4;'

   // Get the maven tool.
   // ** NOTE: This 'mvn' maven tool must be configured
   // **       in the global configuration.
   def mvnHome = tool 'maven339'
   def server = Artifactory.server('DCSR Artifactory')
   def rtMaven = Artifactory.newMavenBuild()
   rtMaven.resolver server: server, releaseRepo: 'remote-repos', snapshotRepo: 'libs-snapshot'
   rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
   rtMaven.deployer.deployArtifacts = false
   rtMaven.tool = 'maven339'

   stage 'build'
   // set the version of the build artifact to the Jenkins BUILD_NUMBER so you can
   // map artifacts to Jenkins builds
   withEnv(["MAVEN_HOME=${mvnHome}"]) {
      //sh "${mvnHome}/bin/mvn versions:set -DnewVersion=${env.BUILD_NUMBER}"
      //sh "${mvnHome}/bin/mvn package"
      rtMaven.run versions:set -DnewVersion=${env.BUILD_NUMBER}
      def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'package'
   }

   stage 'test'
   parallel 'test': {
      buildInfo = rtMaven.run pom: 'pom.xml', goals: 'test';
      sleep 2;
      //sh "${mvnHome}/bin/mvn test; sleep 2;"
   }, 'verify': {
      buildInfo = rtMaven.run pom: 'pom.xml', goals: 'verify';
      sleep 3;
      //sh "${mvnHome}/bin/mvn verify; sleep 3"
   }

   stage 'archive'
   archive 'target/*.jar'
}


node('master') {
   stage 'deploy Canary'
   sh 'echo "write your deploy code here"; sleep 5;'

   stage 'deploy Production'
   input 'Proceed?'
   sh 'echo "write your deploy code here"; sleep 6;'
   archive 'target/*.jar'
}
