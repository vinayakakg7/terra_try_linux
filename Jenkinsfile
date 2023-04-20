pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'Java'
    }

    environment{
        GIT_REPO = 'https://github.com/vinayakakg7/Terra_Try.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Clone Git repository') {
            steps {
                git branch: GIT_BRANCH, url: GIT_REPO
            script {
					// Get the list of culprits
					def culprits = currentBuild.changeSets.collect { it.authorEmail }
                    def subject = 'Git checkout ' + (currentBuild.currentResult == 'SUCCESS' ? 'successful' : 'failed')
                    def body = 'The branch main was checked out ' + (currentBuild.currentResult == 'SUCCESS' ? 'successfully' : 'unsuccessfully') + '.\n\nChanges were made by: ' + culprits.join(', ')
                    emailext subject: subject, body: body, to: 'vinayakakg7@gmail.com', attachLog: true
                }
            }
        }
        stage('Terraform Init') {
            steps {
                script {
                  withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_CREDENTIALS']]) {
                        sh 'terraform init'
                }
            }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_CREDENTIALS']]) {
                      sh 'terraform plan'
                }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_CREDENTIALS']]) {
            // Get the value of the "terra" parameter
					def terra = params.terra

            // Check if the "terra" parameter is set to "destroy"
					if (terra == 'destroy') {
                      echo 'Destroying infrastructure...'
                      sh "terraform destroy --auto-approve"
                      error "Aborting the pipeline after destroying infrastructure" // Stop the pipeline after the destroy command
                    } else {
                          echo 'Applying infrastructure...'
                          sh "terraform apply --auto-approve"
                        }
                }
                }
            }
        }

      stage('Build and test using Maven') {
            steps {
                sh 'mvn clean install -DskipTests=true'
				
				script {
                    def subject = 'Build ' + (currentBuild.currentResult == 'SUCCESS' ? 'successful' : 'failed')
                    def body = 'Maven Build was done ' + (currentBuild.currentResult == 'SUCCESS' ? 'successfully' : 'unsuccessfully')
                    emailext subject: subject, body: body, to: 'vinayakakg7@gmail.com', attachLog: true
                }
            }
          }
}
}