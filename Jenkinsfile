pipeline {
    agent any
    environment {
        PROJECT_NAME = 'flask-app'
        OPENSHIFT_SERVER = 'https://api.cacheocpnode.cacheocp.com:6443'
        OPENSHIFT_TOKEN = credentials('openshift-token')  // Make sure you have a Jenkins credential for OpenShift Token
        GIT_REPO = 'https://github.com/shashank-1-1/flask-app.git'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }
        stage('Build Image') {
            steps {
                script {
                    // Login to OpenShift
                    bat "oc login ${OPENSHIFT_SERVER} --token=${OPENSHIFT_TOKEN} --insecure-skip-tls-verify"
                    
                    // Switch to the OpenShift project
                    bat "oc project flask-app-project"
                    
                    // Handle ImageStream: Delete if it exists, otherwise create a new one
                    bat """
                    oc get imagestream flask-app || echo 'Image stream flask-app not found, skipping deletion'
                    oc delete imagestream flask-app || echo 'Failed to delete imagestream (flask-app), it may not exist'
                    """
                    
                    // Create the ImageStream if it doesn't exist
                    bat "oc create imagestream flask-app || echo 'Image stream flask-app already exists'"

                    // Handle BuildConfig: Check if it exists, create it if it doesn't
                    bat """
                    oc get buildconfig flask-app || (
                        echo 'BuildConfig flask-app not found, creating a new one'
                        oc new-build --binary --name=flask-app --strategy=docker
                    )
                    """

                    // Start the build
                    bat "oc start-build flask-app --from-dir=. --follow"
                }
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Apply DeploymentConfig, Service, and Route if required
                    bat "oc apply -f openshift/deploymentconfig.yaml"
                    bat "oc apply -f openshift/service.yaml"
                    bat "oc apply -f openshift/route.yaml"
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    // Wait for rollout to complete
                    bat "oc rollout status dc/flask-app"
                }
            }
        }
    }
}
