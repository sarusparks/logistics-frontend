pipeline {
    agent any

    tools {
        nodejs 'node18'
        jdk 'jdk21'
    }

    environment {
        ANDROID_HOME = "/home/ubuntu/Android"
        PATH = "${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:$PATH"
    }

    stages {
        stage('Set AWS Credentials') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh 'aws sts get-caller-identity'
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git credentialsId: 'git-access', url: 'https://github.com/sarusparks/logistics-frontend.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'yarn install'
            }
        }

        stage('Build APK') {
            steps {
                dir('android') {
                    sh '''
                        echo "sdk.dir=/home/ubuntu/Android" > local.properties
                        ls -ld /home/ubuntu/Android
                        ls -l local.properties
                        chmod +x ./gradlew
                        ./gradlew assembleRelease --stacktrace
                    '''
                }
            }
        }

        stage('Upload APK to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    script {
                        def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                        def apkPath = "android/app/build/outputs/apk/release/app-release.apk"
                        def s3Path = "s3://testgithub-saru/apk/dev/${commitId}.apk"
                        sh "aws s3 cp ${apkPath} ${s3Path}"
                    }
                }
            }
        }
    }
}
