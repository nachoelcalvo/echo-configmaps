pipeline {
    //Pipelines runs every minute
    triggers {
        pollSCM('* * * * *')
    }
    environment {
        PATH = "$OC_TOOL:$PATH"
        GIT_PROJECT_URL='github.com/nachoelcalvo/echo-configmaps'
        ocProject = 'echo-kubernetes'
        project_env = 'qa'
        qaChanges = ''
    }
    agent any
    stages {
        stage('Finding Config changes') {
            steps {
                script{
                    echo "${env.GIT_COMMIT}"
                    echo "${env.GIT_PREVIOUS_COMMIT}"
                    qaChanges = sh returnStatus: true, script: "git diff --name-only ${env.GIT_PREVIOUS_COMMIT} ${env.GIT_COMMIT} | grep ${project_env}"
                }
           }
        }
        stage('Fetching new configuration'){
            when {
                expression { qaChanges == 0 }
            }
            steps {
              withCredentials([usernamePassword(credentialsId: 'joseignacio-casado-externo', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                  sh "git init"
                  sh "git remote rm origin"
                  sh "git remote add -f origin https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_PROJECT_URL}"
                  sh "git config core.sparseCheckout true"
                  sh "echo ${project_env} >> .git/info/sparse-checkout"
                  sh "git pull origin master"
              }
            }
        }
        stage('Creating Configmap') {
            when {
                expression { qaChanges == 0 }
            }
            steps {
                sh "${oc_cmd} login"
                sh "oc project ${ocProject}"
                sh "oc delete configmap/poc-config || true"
                sh "oc create configmap poc-config --from-file=${project_env}"
            }
        }
        stage('Deploying') {
            when {
                expression { qaChanges == 0 }
            }
            steps {
                echo 'Deploying ...'
            }
        }
    }
    post {
            always {
                deleteDir()
            }
    }
}