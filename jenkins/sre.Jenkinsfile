pipeline {
    agent runner
    
    parameters {
        string(name: 'HELM_RELEASE_NAME', defaultValue: 'prometheus', description: 'Name for the Helm release')
        string(name: 'HELM_CHART', defaultValue: 'prometheus-community/kube-prometheus-stack', description: 'Helm chart to install')
        string(name: 'K8S_NAMESPACE', defaultValue: 'monitoring', description: 'Kubernetes namespace to deploy to')
        string(name: 'GRAFANA_ADMIN_PASSWORD', defaultValue: 'admin', description: 'Grafana admin password')
        booleanParam(name: 'MONITOR_HOST', defaultValue: true, description: 'Install Node Exporter on the host VM')
        string(name: 'ADDITIONAL_HOSTS', defaultValue: '', description: 'Comma-separated list of additional hosts to monitor')
    }
    
    stages {
        stage('Install Minikube') {
            steps {
                sh '''
                # Install Minikube if not already installed
                if ! command -v minikube &> /dev/null; then
                    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
                    sudo install minikube-linux-amd64 /usr/local/bin/minikube
                    rm minikube-linux-amd64
                fi
                
                # Start Minikube if not already running
                if ! minikube status | grep -q "Running"; then
                    minikube start --driver=docker --cpus=2 --memory=3g
                fi
                
                # Verify installation
                minikube status
                kubectl cluster-info
                '''
            }
        }
        
        stage('Install Helm') {
            steps {
                sh '''
                # Install Helm if not already installed
                if ! command -v helm &> /dev/null; then
                    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
                    chmod 700 get_helm.sh
                    ./get_helm.sh
                    rm get_helm.sh
                fi
                
                # Add Prometheus community Helm repository
                helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                helm repo update
                '''
            }
        }
        
        stage('Deploy Prometheus Stack') {
            steps {
                sh '''
                # Create namespace if it doesn't exist
                kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                
                # Install or upgrade Prometheus Stack using Helm
                helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART} \
                  --namespace ${K8S_NAMESPACE} \
                  --set prometheus.service.type=NodePort \
                  --set grafana.service.type=NodePort \
                  --set grafana.adminPassword=${GRAFANA_ADMIN_PASSWORD} \
                  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
                  --set prometheus.prometheusSpec.serviceMonitorSelector.matchLabels.release=${HELM_RELEASE_NAME}
                
                # Wait for deployment to complete
                echo "Waiting for prometheus operator deployment..."
                kubectl -n ${K8S_NAMESPACE} wait --for=condition=available --timeout=300s deployment/${HELM_RELEASE_NAME}-kube-prometheus-operator || true
                
                echo "Waiting for grafana deployment..."
                kubectl -n ${K8S_NAMESPACE} wait --for=condition=available --timeout=300s deployment/${HELM_RELEASE_NAME}-grafana || true
                '''
            }
        }
        
        stage('Install Node Exporter on Host') {
            when {
                expression { params.MONITOR_HOST == true }
            }
            steps {
                sh '''
                # Download and extract node_exporter
                wget -q https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
                tar xvfz node_exporter-*.tar.gz
                
                # Install and start Node Exporter
                sudo cp node_exporter-*/node_exporter /usr/local/bin/
                sudo groupadd --system node_exporter || true
                sudo useradd --system -g node_exporter node_exporter || true
                
                sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOL
[Unit]
Description=Node Exporter
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOL
                
                sudo systemctl daemon-reload
                sudo systemctl enable node_exporter
                sudo systemctl start node_exporter
                
                # Clean up
                rm -rf node_exporter-*
                '''
                
                // Configure ServiceMonitor for host Node Exporter
                sh '''
                HOST_IP=$(hostname -I | awk '{print $1}')
                
                # Create a ServiceMonitor for the host Node Exporter
                cat > host-node-exporter-monitor.yaml << EOL
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: host-node-exporter
  namespace: ${K8S_NAMESPACE}
  labels:
    release: ${HELM_RELEASE_NAME}
spec:
  endpoints:
  - port: metrics
    interval: 15s
  namespaceSelector:
    matchNames:
    - ${K8S_NAMESPACE}
  selector:
    matchLabels:
      app: host-node-exporter
---
apiVersion: v1
kind: Service
metadata:
  name: host-node-exporter
  namespace: ${K8S_NAMESPACE}
  labels:
    app: host-node-exporter
spec:
  ports:
  - name: metrics
    port: 9100
    targetPort: 9100
  type: ExternalName
  externalName: ${HOST_IP}
EOL

                # Apply the configuration
                kubectl apply -f host-node-exporter-monitor.yaml
                '''
            }
        }
        
        stage('Configure Additional Hosts') {
            when {
                expression { params.ADDITIONAL_HOSTS != '' }
            }
            steps {
                script {
                    def hosts = params.ADDITIONAL_HOSTS.split(',')
                    hosts.each { host ->
                        def trimmedHost = host.trim()
                        sh """
                        # Install Node Exporter on additional host
                        ssh ubuntu@${trimmedHost} '
                            wget -q https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
                            tar xvfz node_exporter-*.tar.gz
                            sudo cp node_exporter-*/node_exporter /usr/local/bin/
                            sudo groupadd --system node_exporter || true
                            sudo useradd --system -g node_exporter node_exporter || true
                            
                            sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOL
[Unit]
Description=Node Exporter
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOL
                            
                            sudo systemctl daemon-reload
                            sudo systemctl enable node_exporter
                            sudo systemctl start node_exporter
                            rm -rf node_exporter-*
                        '
                        
                        # Create ServiceMonitor and Service for this host
                        cat > additional-host-${trimmedHost}.yaml << EOL
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: node-exporter-${trimmedHost.replaceAll('\\.', '-')}
  namespace: ${K8S_NAMESPACE}
  labels:
    release: ${HELM_RELEASE_NAME}
spec:
  endpoints:
  - port: metrics
    interval: 15s
  namespaceSelector:
    matchNames:
    - ${K8S_NAMESPACE}
  selector:
    matchLabels:
      app: node-exporter-${trimmedHost.replaceAll('\\.', '-')}
---
apiVersion: v1
kind: Service
metadata:
  name: node-exporter-${trimmedHost.replaceAll('\\.', '-')}
  namespace: ${K8S_NAMESPACE}
  labels:
    app: node-exporter-${trimmedHost.replaceAll('\\.', '-')}
spec:
  ports:
  - name: metrics
    port: 9100
    targetPort: 9100
  type: ExternalName
  externalName: ${trimmedHost}
EOL

                        kubectl apply -f additional-host-${trimmedHost}.yaml
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            sh '''
            echo "========================================================"
            echo "Monitoring stack deployed successfully with Kubernetes!"
            echo
            echo "Access URLs:"
            echo "Prometheus: $(minikube service ${HELM_RELEASE_NAME}-kube-prometheus-prometheus -n ${K8S_NAMESPACE} --url)"
            echo "Grafana: $(minikube service ${HELM_RELEASE_NAME}-grafana -n ${K8S_NAMESPACE} --url)"
            echo
            echo "Grafana credentials:"
            echo "Username: admin"
            echo "Password: ${GRAFANA_ADMIN_PASSWORD}"
            echo "========================================================"
            '''
        }
        failure {
            echo "Deployment failed. Please check the logs for details."
        }
    }
}
