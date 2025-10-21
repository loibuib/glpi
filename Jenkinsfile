node {
    stage('SCM') {
        checkout scm
    }

    stage('TruffleHog Secret Scan') {
        // Run TruffleHog from the Python virtual environment
        sh '''
            /opt/venv/bin/trufflehog filesystem ./ --json > trufflehog-report.json
        '''

        // Archive the TruffleHog JSON report as a build artifact
        archiveArtifacts artifacts: 'trufflehog-report.json', allowEmptyArchive: true

        // Optional: Fail build if secrets are found by checking if report is non-empty
        script {
            def report = readJSON file: 'trufflehog-report.json'
            if (report.size() > 0) {
                error("TruffleHog found secrets in the repository, failing the build!")
            }
        }
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

    stage('Dependency-Check Scan') {
        sh 'mkdir -p dependency-check-reports'

        dependencyCheck(
            odcInstallation: 'owasp-dc',
            additionalArguments: '-s ./ -f HTML -f XML -o dependency-check-reports --disableYarnAudit --disableNodeAudit'
        )

        archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*.html', fingerprint: true
        archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*.xml', fingerprint: true

        dependencyCheckPublisher(
            pattern: 'dependency-check-reports/dependency-check-report.xml'
        )
    }
}
