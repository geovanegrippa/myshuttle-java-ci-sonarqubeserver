pipeline {
    agent { label 'vm-agent-linux-01' }

    environment {
        SONARQUBE_PROJECT_KEY = credentials('sonarqube_project_key')
        SONARQUBE_PROJECT_NAME = "MyShuttle"
        SONAR_HOST_URL = "https://sonarqube.local"
        NODE_EXTRA_CA_CERTS = "/usr/local/share/ca-certificates/sonar.crt"
        JAVA_TOOL_OPTIONS = "-Djavax.net.ssl.trustStore=/usr/local/share/ca-certificates/sonar.jks -Djavax.net.ssl.trustStorePassword=changeit"
    }

    stages {
        stage('Detectar JAVA_HOME') {
            when { expression { false } }
            steps {
                sh '''
                    set -e
                    JAVA_BIN=$(command -v java || true)
                    if [ -z "$JAVA_BIN" ]; then
                      echo "java não encontrado no PATH do agente"
                      exit 1
                    fi

                    JAVA_PATH=$(readlink -f "$JAVA_BIN")
                    JAVA_HOME_DIR=$(dirname "$(dirname "$JAVA_PATH")")

                    echo "JAVA_HOME detectado: $JAVA_HOME_DIR"

                    export JAVA_HOME="$JAVA_HOME_DIR"
                    export PATH="$JAVA_HOME/bin:$PATH"
                    java -version
                '''
            }
        }

        stage('Preparar certificado') {
            when { expression { false } }
            steps {
                script {
                    if (!fileExists('/usr/local/share/ca-certificates/sonar.crt')) {
                        sh """
                            # Baixar o certificado do servidor
                            echo | openssl s_client -connect sonarqube.local:443 -servername sonarqube.local | openssl x509 -outform PEM > sonarqube.crt

                            # copiar e atualizar Certificates
                            cp sonarqube.crt /usr/local/share/ca-certificates/sonar.crt
                            sudo update-ca-certificates

                            # Importar no truststore do Java
                            sudo keytool -import -alias sonarqube \
                            -keystore /usr/local/share/ca-certificates/sonar.jks \
                            -file sonarqube.crt \
                            -storepass changeit \
                            -noprompt

                        """
                    }
                }
            }
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
                            -Dsonar.qualitygate.wait=true
                    '''
                }
            }
        }

        stage('Quality Gate') {
            when { expression { false } }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
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
