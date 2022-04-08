def dockerImage

pipeline {
    agent any

    tools {nodejs "nodejs"}

    stages {
        stage('Git checkout project') {
            agent {
                label 'controller'
            }
            steps {
                git(url: 'git@github.com:Nitronius/node-test.git',
                    credentialsId: 'jenkins-ssh-github',
                    branch: 'master')
            }
        }
        stage('test') {
            agent {
                label 'controller'
            }
            steps {
                sh 'npm i'
                sh 'forever start index.js'
                sh 'npm test'
                sh 'forever stopall'
            }
        }
        stage('Docker build') {
            agent {
                label 'controller'
            }
            steps {
                script {
                    dockerImage = docker.build('eu.gcr.io/solid-justice-346509/node-test:staging', '--rm .')
                }
            }
        }
        stage('Docker push') {
            agent {
                label 'controller'
            }
            steps {
                script {
                    docker.withRegistry('https://eu.gcr.io', 'gcr:solid-justice-346509') {
                        dockerImage.push('staging')
                    }
                }
            }
        }
        stage('Docker remove') {
            agent {
                label 'controller'
            }
            steps {
                sh 'docker image remove eu.gcr.io/solid-justice-346509/node-test:staging'
            }
        }
        stage('Workspace clean') {
            agent {
                label 'controller'
            }
            steps {
                deleteDir()
            }
        }
        stage('Deploy') {
            agent {
                label 'controller'
            }
            steps {
                script {
                    def remote = [:]
                    remote.name = 'node-app'
                    remote.host = '34.89.133.229'
                    remote.user = 'jenkins'
                    remote.identityFile = '/root/.ssh/gcp'
                    remote.allowAnyHosts = true
                    sshCommand remote: remote, command: 'cd /home/vkpanic/app/; sudo docker-compose stop node-app; sudo docker-compose pull node-app; sudo docker-compose up -d node-app'
        }
    }
}
    }
}
//    post {
//    success {
//        mail to: 'vkpanic@gmail.com',
//             subject: "Pipeline: ${currentBuild.fullDisplayName}",
//             body: "Build successful ${env.BUILD_URL}\nLink: http://34.89.133.229:7000/"
//    }
//}