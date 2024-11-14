pipeline {
    agent any
    environment {
        PROJECT_NAME = 'flask-app'
        OPENSHIFT_SERVER = 'https://api.cacheocpnode.cacheocp.com:6443'
        OPENSHIFT_TOKEN = credentials('openshift-token')  // Add token in Jenkins credentials
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
                    // Login to OpenShift (use 'bat' for Windows)
                    bat "oc login ${OPENSHIFT_SERVER} --token=${OPENSHIFT_TOKEN} --insecure-skip-tls-verify"
                    
                    // Switch to project
                    bat "oc project flask-app-project"
                    
                    // Check if the image stream exists before attempting deletion
                    bat """
                    oc get imagestream flask-app || echo 'Image stream flask-app not found, skipping deletion'
                    oc delete imagestream flask-app || echo 'Failed to delete imagestream (flask-app), it may not exist'
                    """

                    // Create the image stream if it doesn't exist
                    bat "oc create imagestream flask-app || echo 'Image stream flask-app already exists'"

                    // Delete build config if it exists
                    bat "oc delete buildconfig flask-app || echo 'Failed to delete buildconfig (flask-app), it may not exist'"

                    // Create a new build and trigger it
                    bat "oc new-build --binary --name=flask-app --strategy=docker"
                    bat "oc start-build flask-app --from-dir=. --follow"
                }
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Apply DeploymentConfig, Service, and Route
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
