pipeline {
    agent any
    tools {
        maven 'Maven-3.6.0' 
        jdk 'JDK-1.8' 
    }

    parameters {
        booleanParam(name: "RELEASE",
                description: "Build a release from current commit.",
                defaultValue: false)
    }
    triggers {
        cron('H 10 * * 1-5')
    }
    stages {
        stage("Build & Deploy SNAPSHOT") {
            steps {
                configFileProvider([configFile(fileId: 'nexus-maven-settings-xml', variable: 'MAVEN_SETTINGS')]) {
                    bat 'mvn -B -e -s %MAVEN_SETTINGS% clean deploy'
                }
            }
        }

        stage("Release") {
            when {
                branch 'master'
                expression { params.RELEASE }
            }
            steps {
                configFileProvider([configFile(fileId: 'nexus-maven-settings-xml', variable: 'MAVEN_SETTINGS')]) {
                    withCredentials([usernamePassword(credentialsId: 'test-github-token',  usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {

                        bat 'mvn -s %MAVEN_SETTINGS% -B -e release:prepare'
                        bat 'mvn -s %MAVEN_SETTINGS% -B -e release:perform'

                    }
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
