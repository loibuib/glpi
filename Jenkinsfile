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

stage('OWASP FS SCAN') {
        // Run Dependency-Check scan with arguments and specified tool
        dependencyCheck(
            odcInstallation: 'owasp-dc',
            additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit'
        )

        // Publish the XML report in Jenkins
        dependencyCheckPublisher(
            pattern: '**/dependency-check-report.xml'
        )
    }

}
