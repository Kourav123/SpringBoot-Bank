#!/usr/bin/env groovy

pipeline {
    agent {
        docker {
            label 'docker'
            image 'rb-dtr.de.bosch.com/bidos/gradle:4.5.1-jdk8'
            args '-u root:root'
        }
    }

    stages {
        stage('Prepare'){
            steps{
                script {
                    withCredentials([file(credentialsId: 'GradleArtifactory', variable: 'CREDFILE')]){
                        sh 'cp $CREDFILE ~/.gradle/gradle.properties'
                        sh 'chmod 744 ~/.gradle/gradle.properties'
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh 'gradle clean compileJava'
            }
        }
        stage('Verify') {
            steps {
                sh 'gradle check'
            }
            post {
                always {
                    //JUNIT task report
                    junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'
                }
            }
        }
        //Starting the SonarQube analysis using our sonarCube
        stage('SonarQube analysis') {
            steps {
                echo "Performing static code analysis with sonar"
                script {
                    // Pull the SonarQube Environment variables from jenkins
                    withSonarQubeEnv('CI SonarQube') {
                        sh "gradle :sonarqube -Dsonar.branch.name=$BRANCH_NAME"
                    }
                }
            }
        }
        stage('Documentation') {
            steps {
                sh 'gradle javadoc'
            }
            post {
                success {
                    zip dir: "${workspace}/build/docs/javadoc", zipFile: "javaDoc.zip"
                    archiveArtifacts artifacts: 'javaDoc.zip', fingerprint: false, allowEmptyArchive: true
                }
            }
        }
        stage('Package and Archive') {
            steps {
                sh 'gradle  build'
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true, allowEmptyArchive: true

            }
        }
    }
    post {
        always {
            script {
                cleanWs()
            }
        }
    }
}
