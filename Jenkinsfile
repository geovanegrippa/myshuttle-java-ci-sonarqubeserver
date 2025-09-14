pipeline {
    agent { label 'vm-agent-linux-01' }

    environment {
        //NODE_EXTRA_CA_CERTS = "/usr/local/share/ca-certificates/sonar.crt"
        SONARQUBE_PROJECT_KEY = credentials('sonarqube_project_key')
        SONARQUBE_PROJECT_NAME = "MyShuttle"
        SONAR_HOST_URL = "https://sonarqube.local"
        SONAR_TOKEN = credentials('sonar_token') 
        SONAR_CERT_PATH = "/usr/local/share/ca-certificates/sonar.crt"
        SONAR_KEYSTORE = "/usr/local/share/ca-certificates/sonar.jks"
    }

    stages {
        stage('Detectar JAVA_HOME') {
            when { expression { true } }
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

        stage('Preparar certificado') {
            steps {
                script {
                    // Verifica se o keystore já existe, se não cria
                    if (!fileExists(env.SONAR_KEYSTORE)) {
                        sh """
                        keytool -importcert \
                            -file ${env.SONAR_CERT_PATH} \
                            -alias sonar \
                            -keystore ${env.SONAR_KEYSTORE} \
                            -storepass changeit \
                            -noprompt
                        """
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube_server') {
                    sh '''
                        export JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=${SONAR_KEYSTORE} -Djavax.net.ssl.trustStorePassword=changeit"
                        
                        /opt/sonar-scanner/bin/sonar-scanner \
                            -Dsonar.projectKey=$SONARQUBE_PROJECT_KEY \
                            -Dsonar.projectName=$SONARQUBE_PROJECT_NAME \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.libraries=target/**/*.jar
                    '''
                }
            }
        }

    }
}
