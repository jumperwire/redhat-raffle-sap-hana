#!groovy
// Run this pipeline on the custom Maven Slave ('maven-appdev')
// Maven Slaves have JDK and Maven already installed
// 'maven-appdev' has skopeo installed as well.
node('maven-appdev') {
  // Define Maven Command. Make sure it points to the correct
  // settings for our Nexus installation (use the service to
  // bypass the router). The file nexus_openshift_settings.xml
  // needs to be in the Source Code repository.
  def mvnCmd = "mvn -s ../../nexus_openshift_settings.xml"
  // Checkout Source Code
  stage('Checkout Source') {
    git credentialsId: '4b54c031-43eb-49e9-bc05-c201e8af69f2', url: 'http://gogs-gogs.cloudapps.rhdemo.io/SKODemo/redhat-raffle-sap-hana.git'
  }
  
  dir('raffle-microservice/raffle-service') {
    // The following variables need to be defined at the top level
    // and not inside the scope of a stage - otherwise they would not
    // be accessible from other stages.
    // Extract version and other properties from the pom.xml
    def groupId    = getGroupIdFromPom("pom.xml")
    def artifactId = getArtifactIdFromPom("pom.xml")
    def version    = getVersionFromPom("pom.xml")
    // Set the tag for the development image: version + build number
    def devTag  = "${version}-${BUILD_NUMBER}"
    // Set the tag for the production image: version
    def prodTag = "${version}"
    // Using Maven build the war file
    // Do not run tests in this step
    stage('Dev') {
      echo "Dev stage"
      sh "wget https://www.dropbox.com/s/nvkw8gh3rmf932j/ngdbc-2.3.58.jar?dl=1"
      sh "mv ngdbc-2.3.58.jar?dl=1 ngdbc-2.3.58.jar"
      sh "${mvnCmd} install:install-file -Dfile=ngdbc-2.3.58.jar -DgroupId=com.sap.db.jdbc -DartifactId=ngdbc -Dversion=2.3.58 -Dpackaging=jar"
    }
    
    stage('Test') {
      echo "Test stage"
      sh "${mvnCmd} clean package -DskipTests"
    }
    
    stage('QA') {
      echo "QA stage"
      sh "oc project redhat-raffle"
      sh "${mvnCmd} fabric8:deploy -Popenshift"
    }
    
    stage('Prod') {
      echo "Prod stage"
      sleep 5
    }
  }
}
// Convenience Functions to read variables from the pom.xml
// Do not change anything below this line.
// --------------------------------------------------------
def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
def getGroupIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
  matcher ? matcher[0][1] : null
}
def getArtifactIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
  matcher ? matcher[0][1] : null
}
