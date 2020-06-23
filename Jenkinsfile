node {

  try {
    def mvnHome
    def jdk
    def pom
    def artifactVersion
    def tagVersion
    def retrieveArtifact

    stage('Prepare') {
      mvnHome = tool 'maven'
      jdk = tool 'jdk11'
      env.JAVA_HOME = "${jdk}"
      
    }

    stage('Checkout') {
       checkout scm
    }

    stage('Build') {
                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
    }

    stage('Unit Test') {
       junit '**/target/surefire-reports/TEST-*.xml'
       archive 'target/*.jar'
    }

    stage('Integration Test') {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean verify"
    }

   
    if(env.BRANCH_NAME == 'master'){
      stage('Validate Build Post Prod Release') {
        if (isUnix()) {
           sh "'${mvnHome}/bin/mvn' clean package"
        } else {
           bat(/"${mvnHome}\bin\mvn" clean package/)
        }
      }

    }

      if(env.BRANCH_NAME == 'develop'){
        stage('Snapshot Build And Upload Artifacts') {
             sh "'${mvnHome}/bin/mvn' clean deploy"
        }

      }

      if(env.BRANCH_NAME ==~ /release.*/){
        pom = readMavenPom file: 'pom.xml'
        artifactVersion = pom.version.replace("-SNAPSHOT", "")
        tagVersion = 'v'+artifactVersion

        stage('Release Build And Upload Artifacts') {
             sh "'${mvnHome}/bin/mvn' clean release:clean release:prepare release:perform"
        }
         


        /* stage("Deploy from Artifactory to QA"){
           retrieveArtifact = 'http://localhost:8081/artifactory/libs-release-local/com/example/devops/' + artifactVersion + '/devops-' + artifactVersion + '.war'
           echo "${tagVersion} with artifact version ${artifactVersion}"
           echo "Deploying war from http://localhost:8081/artifactory/libs-release-local/com/example/devops/${artifactVersion}/devops-${artifactVersion}.war"
           sh 'curl -O ' + retrieveArtifact
           sh 'curl -u jenkins:jenkins -T *.war "http://localhost:7080/manager/text/deploy?path=/devops&update=true"'
         }*/

      }
  } finally {
     step([$class: 'Wscleanup'])
  }

}
