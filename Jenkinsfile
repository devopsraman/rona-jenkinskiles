pipeline {
    
    agent any 

    options {
        buildDiscarder(logRotator(numToKeepStr:'10'))
    }

    tools { 
        maven 'Maven 3.5.3' 
        jdk 'Sun JDK 1.8' 
    }
    
    stages {

        stage('Init') {
            steps{
                withMaven(
                    mavenSettingsConfig: 'bps-mvn-settings') {
                        sh "cp $MVN_SETTINGS settings.xml"
                    }
            }
        }

        stage('Build') {
            steps{
                sh 'mvn -B -s settings.xml clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn -B -s settings.xml test'
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Integration Tests') {
            steps {
                sh 'mvn -B -s settings.xml verify'
            }
        }

        stage('SonarQube analysis') {
            steps {
                sh 'mvn -B -s settings.xml clean org.jacoco:jacoco-maven-plugin:prepare-agent test'
                jacoco(execPattern: '**/target/jacoco.exec')
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B -s settings.xml -Dsonar.branch="' + env.BRANCH_NAME + '" package sonar:sonar'
                }

            }
        }

/*
        stage('NexusIQ Analysis') {
            steps {
                nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'feature-devops-poc', iqStage: 'build', jobCredentialsId: 'nexusiq'
            }
        }  
*/

        stage('Package') {
            steps {
                script {
                    if (GIT_BRANCH == "develop") {
                            sh 'mvn -B -s settings.xml package deploy:deploy -DaltDeploymentRepository=bps-snapshot::default::http://URL/nexus/repository/bkp-snapshot -DskipTests -DskipITs'
                    }else {
                           sh 'mvn -B -s settings.xml package -DskipTests -DskipITs '
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar'
                }
            }
        }

    }

   post {
        failure {
            emailext (
                subject: "[JENKINS SEED2] FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                         <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                to: '#bpsdevops@vocalink.com',
                recipientProviders: [[$class: 'DevelopersRecipientProvider'],[$class: 'RequesterRecipientProvider'],[$class: 'CulpritsRecipientProvider']]
        )
        }
    }
}
