pipeline {
    agent {
        kubernetes {
            inheritFrom 'flutter'  // References the predefined pod template named "flutter"
            defaultContainer 'flutter'  // Sets the default container for steps
        }
    }
    
    environment {
        FLUTTER_HOME = '/sdks/flutter/bin/'
        ANDROID_SDK_ROOT = '/android-sdk'
        PATH = "$FLUTTER_HOME/bin:$ANDROID_SDK_ROOT/tools/bin:$PATH"
        APP_NAME = 'flutter-form-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                container('flutter') {
                    git branch: 'main', url: 'https://github.com/zts3y/flutter-form-app.git'
                }
            }
        }
        stage('Flutter Clean') {
            steps {
                container('flutter') {
                    sh '/sdks/flutter/bin/flutter --version'
                    // sh '/sdks/flutter/bin/flutter doctor'
                    sh '/sdks/flutter/bin/flutter clean'
                    sh '/sdks/flutter/bin/flutter pub get'
                }
            }
        }

        stage('Tests'){
            parallel {
                stage('Flutter Tests') {
                    steps {
                        container('flutter') {
                            sh '/sdks/flutter/bin/flutter test'
                        }
                    }
                }
                stage('Sonarqube Tests') {
                    steps {
                        container('flutter') {
                            script {
                                def scannerHome = tool 'sonarqube'
                                withSonarQubeEnv('sonarqube') {
                                    sh """
                                        ${scannerHome}/bin/sonar-scanner \
                                        -Dsonar.projectKey=${env.APP_NAME} \
                                        -Dsonar.sources=./lib \
                                        -Dsonar.host.url=${SONAR_HOST_URL} \
                                    """
                                }
                            }                    
                        }
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build') {
            parallel {
                stage('Build Web') {
                    steps {
                        container('flutter') {
                            sh '/sdks/flutter/bin/flutter build web --release'
                        }
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'build/web/**/*', fingerprint: true
                        }
                    }
                }
                stage('Build Android') {
                    steps {
                        container('flutter') {
                            sh '/sdks/flutter/bin/flutter build apk --release'
                        }
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'build/app/outputs/flutter-apk/app-release.apk', fingerprint: true
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}