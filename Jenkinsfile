pipeline {
    agent any
    parameters {
        string(name: 'git_repo', defaultValue: 'https://github.com/jleetutorial/maven-project.git', description: 'Git Repo')
    }
    stages {
        stage('Preparation') {
            steps {
                echo "Git Repo => ${params.git_repo}"
                git "${params.git_repo}"
            }
        }
        stage('Build') {
            steps {
                sh "'mvn' -Dmaven.test.failure.ignore clean package -U"
            }
        }
        stage('Test') {
            steps {
                junit '**/target/surefire-reports/TEST-*.xml'
            }
            post {
                success {
                    echo "Archiving pacakge..."
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage ('Deploy to Staging') {
            steps {
                build job: "deploy-to-staging"
            }
        }
    }
}
