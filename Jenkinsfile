node {
    stage('SCM') {
        checkout scm
    }

    stage('TruffleHog Secret Scan') {
        sh '''
            /opt/venv/bin/trufflehog filesystem --repo_path ./ --json > trufflehog-report.json
        '''

        archiveArtifacts artifacts: 'trufflehog-report.json', allowEmptyArchive: true

        script {
            def report = readJSON file: 'trufflehog-report.json'
            if (report.size() > 0) {
                error("Secrets found by TruffleHog, failing the build!")
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
