pipeline {
    agent { label 'vm-agent-linux-01' }

    environment {
        SONARQUBE_PROJECT_KEY = credentials('sonarqube_project_key')
        SONARQUBE_PROJECT_NAME = "MyShuttle"
        NODE_EXTRA_CA_CERTS = "/usr/local/share/ca-certificates/sonar.crt"
        JAVA_TOOL_OPTIONS = "-Djavax.net.ssl.trustStore=/usr/local/share/ca-certificates/sonar.jks -Djavax.net.ssl.trustStorePassword=changeit"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }

         stage('Build/Test Maven') {
            steps {
                sh '''
                    mvn clean verify \
                      -Dmaven.test.failure.ignore=false \
                      -Dsonar.skip=true
                '''
            }
            post {
                always {
                    junit '**/TEST-*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube_server') {
                    sh '''                      
                        /opt/sonar-scanner/bin/sonar-scanner \
                            -Dsonar.projectKey=$SONARQUBE_PROJECT_KEY \
                            -Dsonar.projectName=$SONARQUBE_PROJECT_NAME \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.libraries=target/**/*.jar \
                            -Dsonar.qualitygate.wait=false
                    '''
                }
            }
        }

        stage('Quality Gate') {
            when { expression { true } }
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Publicar Artefato') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

    }
}
