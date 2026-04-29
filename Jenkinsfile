pipeline {
agent any

environment {
    PROJECT_ID = "kong-gke-493913"
    REGION = "us-central1"
    CLUSTER_NAME = "kong-cluster1"
    NAMESPACE = "kong"
}

stages {

    stage('Checkout Code') {
        steps {
            checkout scm
        }
    }

    stage('Authenticate GCP') {
        steps {
            withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh '''
                gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                gcloud config set project $PROJECT_ID
                gcloud auth list
                '''
            }
        }
    }

    stage('Connect to GKE') {
        steps {
            sh '''
            gcloud container clusters get-credentials $CLUSTER_NAME \
              --region $REGION \
              --project $PROJECT_ID
            '''
        }
    }


    stage('Deploy Kong via Helm') {
        steps {
            sh '''
            helm repo add kong https://charts.konghq.com
            helm repo update

            helm upgrade --install kong kong/kong \
              -n $NAMESPACE \
              --create-namespace \
              -f values.yaml
            '''
        }
    }

    stage('Verify Deployment') {
        steps {
            sh '''
            kubectl get pods -n $NAMESPACE
            kubectl get svc -n $NAMESPACE
            '''
        }
    }
}

}
