import groovy.json.*

node () {
   def policyEvaluation

   stage('Preparation') {
      checkout scm
      mvnHome = tool 'Maven'
      jdk = tool name: 'JDK-8'
      env.JAVA_HOME = "${jdk}"
   }
    stage('Build') {
         sh "${mvnHome}/bin/mvn --version"
         sh "${mvnHome}/bin/mvn -N io.takari:maven:wrapper"
         sh "./mvnw -Dmaven.test.failure.ignore clean install"
   }

   stage('IQ Policy Check') {
        // env.GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
        policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: false, iqScanPatterns: [[scanPattern: 'target/struts2-rest-showcase.war']], iqApplication: "struts2-rce-github-lw", iqStage: 'build', jobCredentialsId: ''
        sh "echo ${policyEvaluation.applicationCompositionReportUrl}"
   }

    stage('Create TAG') {
      sh "echo IQ-Policy-Evaluation-struts2-rce-github-lw_${currentBuild.number}"
        createTag nexusInstanceId: 'nxrm3', tagAttributesJson: '''{
            "IQScan": "\${policyEvaluation.applicationCompositionReportUrl}",
            "JenkinsBuild": "\${BUILD_URL}"
          }''', tagName: "IQ-Policy-Evaluation-struts2-rce-github-lw_${currentBuild.number}"
    }

    stage('Publish') {
        nexusPublisher nexusInstanceId: 'nxrm3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: ["./target/struts2-rest-showcase.war"], mavenCoordinate: [artifactId: 'struts2-rest-showcase', groupId: 'org.apache.struts', packaging: 'war', version: '2.5.10']]], tagName: "IQ-Policy-Evaluation-struts2-rce-github-lw_${currentBuild.number}"
    }
}  

