pipeline {
agent any

```
environment {
    PROJECT_ID = "kong-gke-493913"
    REGION = "asia-south1"
    CLUSTER_NAME = "kong-cluster"
    RELEASE_NAME = "kong-dp"
    NAMESPACE = "default"
}

stages {

    stage('Checkout Repo') {
        steps {
            checkout scm
        }
    }

    stage('Authenticate to GCP') {
        steps {
            withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh '''
                gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                gcloud config set project $PROJECT_ID
                '''
            }
        }
    }

    stage('Connect to GKE Cluster') {
        steps {
            sh '''
            gcloud container clusters get-credentials $CLUSTER_NAME --region $REGION
            '''
        }
    }

    stage('Install Helm (if not present)') {
        steps {
            sh '''
            if ! command -v helm &> /dev/null
            then
              curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
            fi
            '''
        }
    }

    stage('Add Kong Helm Repo') {
        steps {
            sh '''
            helm repo add kong https://charts.konghq.com
            helm repo update
            '''
        }
    }

    stage('Deploy Kong DP via Helm') {
        steps {
            sh '''
            helm upgrade --install $RELEASE_NAME kong/kong \
              --namespace $NAMESPACE \
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
```

}
