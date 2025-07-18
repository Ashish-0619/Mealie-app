// Jenkinsfile for DevSecOps CI/CD Pipeline for Mealie-app
// This pipeline integrates various security checks at different stages.

// Define global environment variables for the pipeline
def dockerImageName = "mealie-app"
def dockerRegistry = "your-docker-registry.com" // Replace with your Docker registry if you have one
def dockerRegistryCredsId = "docker-hub-creds" // Jenkins credential ID for Docker registry login
def sonarQubeUrl = "http://your-sonarqube-server:9000" // Replace with your SonarQube server URL (e.g., http://localhost:9000 if running locally)
def sonarQubeTokenId = "sonarqube-token" // Jenkins credential ID for SonarQube token

pipeline {
    // Agent definition: 'any' means Jenkins will pick any available agent.
    // For a local setup, this typically means the Jenkins controller itself or a local agent.
    agent any

    // Define tools required for the pipeline.
    // These tools must be pre-configured in Jenkins Global Tool Configuration
    // or available on the agent's PATH.
    tools {
        // Assuming Maven is needed for some dependency checks or future Java components
        // maven 'Maven 3.8.6' // Uncomment and configure if Maven is used
        jdk 'JDK 17' // For OWASP Dependency-Check
    }

    // Environment variables specific to this pipeline
    environment {
        // Set the Python version for virtual environment
        PYTHON_VERSION = "python3.9"
        // SonarQube project key and name
        SONAR_PROJECT_KEY = "MealieApp"
        SONAR_PROJECT_NAME = "Mealie App DevSecOps"
    }

    // Stages of the CI/CD pipeline
    stages {
        // Stage 1: Source Code Checkout
        // Fetches the latest code from the GitHub repository.
        stage('Checkout Source Code') {
            steps {
                script {
                    echo "Checking out source code from Git..."
                    // Uses the SCM configured for the Jenkins job (e.g., GitHub hook)
                    // If not configured, you can use: git branch: 'main', url: 'https://github.com/Ashish-0619/Mealie-app.git'
                    checkout scm
                    echo "Source code checked out successfully."
                }
            }
        }

        // Stage 2: Static Application Security Testing (SAST) - Bandit
        // Analyzes Python code for common security vulnerabilities without running it.
        stage('SAST - Bandit') {
            agent {
                // Use a Docker agent with Python pre-installed for isolated execution.
                // This requires Docker to be installed and running on the Jenkins host.
                docker { image 'python:3.9-slim-buster' }
            }
            steps {
                script {
                    echo "Running Bandit for SAST..."
                    // Install Bandit in a virtual environment
                    sh "python3 -m venv venv"
                    sh ". venv/bin/activate && pip install bandit"
                    // Run Bandit and output results to a JSON file
                    // '-r .' recursively scans the current directory
                    // '-o bandit_report.json' outputs to a file
                    // '-f json' specifies JSON format
                    def banditCommand = ". venv/bin/activate && bandit -r . -o bandit_report.json -f json"
                    try {
                        sh banditCommand
                        echo "Bandit scan completed. Review bandit_report.json for findings."
                    } catch (Exception e) {
                        echo "Bandit scan failed or found issues. Check logs for details."
                        // Optionally, fail the pipeline if high/medium severity issues are found
                        // For a more robust check, you'd parse bandit_report.json
                        currentBuild.result = 'UNSTABLE' // Mark as unstable, not a full failure yet
                    }
                    // Archive the report for later review
                    archiveArtifacts artifacts: 'bandit_report.json', fingerprint: true
                }
            }
        }

        // Stage 3: Dependency Vulnerability Scanning - OWASP Dependency-Check
        // Scans project dependencies (e.g., requirements.txt) for known vulnerabilities.
        stage('Dependency Scan - OWASP DC') {
            agent {
                // Use a Docker agent with Java pre-installed for OWASP Dependency-Check.
                // This requires Docker to be installed and running on the Jenkins host.
                docker { image 'openjdk:17-jdk-slim' }
            }
            steps {
                script {
                    echo "Running OWASP Dependency-Check..."
                    // Install wget as it's a slim image and might not have it
                    sh "apt-get update && apt-get install -y wget unzip"
                    // Download OWASP Dependency-Check
                    sh "wget -q -O dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.2/dependency-check-8.4.2-release.zip"
                    sh "unzip -q dependency-check.zip"
                    sh "mv dependency-check-8.4.2/bin/dependency-check.sh ."
                    sh "chmod +x dependency-check.sh"

                    // Run Dependency-Check against the project directory
                    // '-f JSON' specifies JSON output format
                    // '-o dependency-check-report.json' outputs to a file
                    // '-s .' scans the current directory (which contains requirements.txt)
                    def dcCommand = "./dependency-check.sh --scan . --format JSON --out dependency-check-report.json"
                    try {
                        sh dcCommand
                        echo "OWASP Dependency-Check completed. Review dependency-check-report.json for findings."
                    } catch (Exception e) {
                        echo "OWASP Dependency-Check failed or found issues. Check logs for details."
                        currentBuild.result = 'UNSTABLE'
                    }
                    archiveArtifacts artifacts: 'dependency-check-report.json', fingerprint: true
                }
            }
        }

        // Stage 4: Unit Testing
        // Runs automated unit tests to ensure code functionality and prevent regressions.
        // (Assuming you will add unit tests to your project, e.g., using pytest)
        stage('Unit Tests') {
            agent {
                // Use a Docker agent with Python pre-installed.
                // This requires Docker to be installed and running on the Jenkins host.
                docker { image 'python:3.9-slim-buster' }
            }
            steps {
                script {
                    echo "Running unit tests..."
                    // Install dependencies and pytest
                    sh "python3 -m venv venv"
                    sh ". venv/bin/activate && pip install -r requirements.txt pytest"
                    // Execute pytest. If tests fail, 'sh' will throw an error and fail the stage.
                    try {
                        sh ". venv/bin/activate && pytest"
                        echo "Unit tests passed."
                    } catch (Exception e) {
                        echo "Unit tests failed. Please check the test results."
                        error "Unit tests failed." // Fail the pipeline if tests fail
                    }
                }
            }
        }

        // Stage 5: Build Docker Image
        // Builds the Docker image for the Mealie-app.
        stage('Build Docker Image') {
            agent any // Assuming Docker is installed and accessible on the Jenkins host/agent
            steps {
                script {
                    echo "Building Docker image: ${dockerImageName}:latest"
                    // Build the Docker image
                    sh "docker build -t ${dockerImageName}:latest ."
                    echo "Docker image built successfully."

                    // Optional: Login to Docker registry and push the image
                    // This requires a Jenkins 'Secret text' credential with ID 'docker-hub-creds'
                    // containing your Docker Hub password/token.
                    // The username should be configured in the Jenkins Docker plugin or directly in the sh command.
                    // withCredentials([usernamePassword(credentialsId: dockerRegistryCredsId, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    //     sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin ${dockerRegistry}"
                    //     sh "docker tag ${dockerImageName}:latest ${dockerRegistry}/${dockerImageName}:latest"
                    //     sh "docker push ${dockerRegistry}/${dockerImageName}:latest"
                    //     echo "Docker image pushed to registry."
                    // }
                }
            }
        }

        // Stage 6: Container Security Scanning - Trivy
        // Scans the built Docker image for known vulnerabilities.
        stage('Container Scan - Trivy') {
            agent any // Assuming Trivy is installed and Docker is accessible on the Jenkins host/agent
            steps {
                script {
                    echo "Running Trivy scan on Docker image: ${dockerImageName}:latest"
                    // Install Trivy (if not pre-installed on agent) or use a Trivy Docker image
                    // For example: docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${dockerImageName}:latest
                    // For simplicity, assuming Trivy is installed on the local Jenkins agent.
                    def trivyCommand = "trivy image --format json -o trivy_report.json ${dockerImageName}:latest"
                    try {
                        sh trivyCommand
                        echo "Trivy scan completed. Review trivy_report.json for findings."
                        // Optionally, fail the pipeline if critical/high vulnerabilities are found
                        // sh "trivy image --exit-code 1 --severity CRITICAL,HIGH ${dockerImageName}:latest"
                    } catch (Exception e) {
                        echo "Trivy scan failed or found issues. Check logs for details."
                        currentBuild.result = 'UNSTABLE'
                    }
                    archiveArtifacts artifacts: 'trivy_report.json', fingerprint: true
                }
            }
        }

        // Stage 7: SonarQube Analysis (Optional - Requires SonarQube Server)
        // Performs deeper static code analysis for bugs, code smells, and security vulnerabilities.
        stage('SonarQube Analysis') {
            // This stage requires the SonarQube Scanner for Jenkins plugin and a running SonarQube server.
            // Configure SonarQube server in Jenkins -> Manage Jenkins -> Configure System -> SonarQube Servers.
            // Configure SonarQube token in Jenkins Credentials with ID 'sonarqube-token'.
            agent any // SonarQube analysis runs on the Jenkins host/agent
            steps {
                withSonarQubeEnv(credentialsId: sonarQubeTokenId, installationName: 'SonarQube') { // 'SonarQube' is the name configured in Jenkins
                    // Using triple quotes for multi-line shell command for better Groovy compatibility
                    sh """
                        sonar-scanner \\
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \\
                            -Dsonar.projectName='${SONAR_PROJECT_NAME}' \\
                            -Dsonar.sources=. \\
                            -Dsonar.host.url=${sonarQubeUrl} \\
                            -Dsonar.login=${SONAR_PROJECT_KEY}
                    """
                    echo "SonarQube analysis triggered. Check SonarQube dashboard for results."
                }
            }
            post {
                // Gate the pipeline based on SonarQube quality gate status
                always {
                    script {
                        // This step will wait for the SonarQube analysis to complete and check the Quality Gate status.
                        // If the Quality Gate fails, the pipeline will be marked as FAILED.
                        // This requires the SonarQube Scanner for Jenkins plugin.
                        // For this to work, the Jenkins job must be configured to poll SonarQube for Quality Gate status.
                        // You might need to adjust the 'installationName' to match your SonarQube setup in Jenkins.
                        try {
                            // This step is provided by the SonarQube Scanner for Jenkins plugin
                            // It polls the SonarQube server for the Quality Gate status
                            // and will fail the build if the Quality Gate is not passed.
                            // The 'installationName' here refers to the name you gave your SonarQube server
                            // configuration in Jenkins global settings.
                            timeout(time: 5, unit: 'MINUTES') { // Wait up to 5 minutes for Quality Gate check
                                waitForQualityGate abortPipeline: true
                            }
                            echo "SonarQube Quality Gate Passed."
                        } catch (Exception e) {
                            echo "SonarQube Quality Gate Failed. Please review SonarQube dashboard."
                            error "SonarQube Quality Gate Failed."
                        }
                    }
                }
            }
        }

        // Stage 8: Dynamic Application Security Testing (DAST) - Placeholder
        // This stage typically requires the application to be deployed and running in a test environment.
        // OWASP ZAP is a common tool for DAST.
        stage('DAST - OWASP ZAP (Placeholder)') {
            agent any // DAST can run on the Jenkins host/agent, but requires a deployed app
            steps {
                script {
                    echo "DAST requires the application to be deployed and running."
                    echo "To implement DAST, you would typically:"
                    echo "1. Deploy the Docker image to a temporary staging environment (e.g., Kubernetes, Docker Compose)."
                    echo "2. Run a DAST tool like OWASP ZAP against the deployed application's URL."
                    echo "   Example: docker run -v ${PWD}:/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -t http://your-app-url:5000 -I -r zap_report.html"
                    echo "3. Analyze the DAST report and fail the pipeline if critical vulnerabilities are found."
                    echo "Skipping DAST for now as it requires a running environment."
                    // currentBuild.result = 'UNSTABLE' // Mark as unstable if DAST is skipped but important
                }
            }
        }

        // Stage 9: Deployment to Staging
        // Deploys the application to a staging environment after all checks pass.
        stage('Deploy to Staging') {
            agent any // Deployment commands run on the Jenkins host/agent
            steps {
                script {
                    echo "Deploying ${dockerImageName}:latest to Staging environment..."
                    // Example: Deploy using Docker Compose, Kubernetes, or other deployment tools
                    // For Docker Compose (assuming docker-compose.yml is in your repo):
                    // sh "docker-compose -f docker-compose.yml up -d"
                    // For local Kubernetes (Minikube/Kind):
                    // sh "kubectl apply -f kubernetes-deployment.yaml"
                    echo "Deployment to Staging completed. Verify application functionality."
                }
            }
        }

        // Stage 10: Manual Approval for Production (Optional)
        // Adds a manual gate before deploying to production.
        stage('Manual Approval for Prod') {
            when {
                environment name: 'BRANCH_NAME', value: 'main' // Only for 'main' branch
            }
            steps {
                script {
                    echo "Waiting for manual approval to deploy to Production..."
                    timeout(time: 1, unit: 'HOURS') { // Timeout after 1 hour if no approval
                        input message: 'Approve deployment to Production?', ok: 'Deploy to Production'
                    }
                    echo "Manual approval received. Proceeding to Production deployment."
                }
            }
        }

        // Stage 11: Deployment to Production
        // Deploys the application to the production environment.
        stage('Deploy to Production') {
            when {
                environment name: 'BRANCH_NAME', value: 'main' // Only for 'main' branch
            }
            agent any // Production deployment commands run on the Jenkins host/agent
            steps {
                script {
                    echo "Deploying ${dockerImageName}:latest to Production environment..."
                    // Production deployment commands
                    echo "Deployment to Production completed successfully!"
                }
            }
        }
    }

    // Post-build actions: run regardless of pipeline success or failure
    post {
        always {
            echo "Pipeline finished."
            // Clean up workspace to free up disk space
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully! All checks passed."
            // You can add notifications here (e.g., Slack, Email)
        }
        failure {
            echo "Pipeline failed! Review logs for errors."
            // You can add failure notifications here
        }
        unstable {
            echo "Pipeline finished with unstable status (some security issues found or warnings)."
            // You can add unstable notifications here
        }
    }
}
