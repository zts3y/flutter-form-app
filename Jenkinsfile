pipeline {
    agent {
        kubernetes {
            inheritFrom 'flutter'  // References the predefined pod template named "flutter"
            defaultContainer 'flutter'  // Sets the default container for steps
        }
    }
    
    environment {
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
                    sh 'flutter --version'
                    // sh 'flutter doctor'
                    sh 'flutter clean'
                    sh 'flutter pub get'
                }
            }
        }

        
        stage('Flutter Tests') {
            steps {
                container('flutter') {
                    sh 'flutter test'
                    sh 'flutter test --machine --coverage > tests.output'
                }
            }
        }
        stage('Sonarqube Tests') {
            steps {
                container('flutter') {
                    script {
                        def scannerHome = tool 'sonarqube'
                        withSonarQubeEnv('sonarqube') {
                            sh 'flutter pub get'
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
                            sh 'flutter build web --release'
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
                            sh 'flutter build apk --release'
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