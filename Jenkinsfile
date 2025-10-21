node {
  stage('SCM') {
    checkout scm
  }

  stage('SonarQube Analysis') {
    def scannerHome = tool 'SonarScanner'
    withSonarQubeEnv() {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }

  stage('Generate SBOM') {
    sh 'syft dir:. --output cyclonedx-json=sbom.json'
    archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom.json', fingerprint: true
  }

  stage('Scan Vulnerabilities with Grype') {
    sh 'grype sbom:sbom.json --output json > grype-report.json'
    archiveArtifacts allowEmptyArchive: true, artifacts: 'grype-report.json', fingerprint: true
  }

  stage('OWASP Dependency-Check Vulnerabilities') {
      steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
}


