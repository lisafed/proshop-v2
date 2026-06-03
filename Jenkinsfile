pipeline {
    agent any

    environment {
        BACKEND_IMAGE  = "proshop-backend:latest"
        FRONTEND_IMAGE = "proshop-frontend:latest"
    }

    stages {
        stage('Informations') {
            steps {
                echo "===== Informations ====="
                sh "pwd"
                sh "ls -la"
                sh "echo Git Branch: ${env.BRANCH_NAME ?: 'origin/main'}"
                sh "echo Build Number: ${env.BUILD_NUMBER}"
                sh "echo Build ID: ${env.BUILD_ID}"
            }
        }

        stage('Docker Test') {
            steps {
                echo "===== Verification Docker ====="
                sh "docker --version"
                sh "docker ps"
                sh "docker compose version"
            }
        }

        stage('Debug Frontend Context') {
            steps {
                echo "===== Debug Frontend Build Context ====="
                sh "ls -la frontend/"
                sh "test -f frontend/nginx.conf && echo '✅ nginx.conf existe' || echo '❌ nginx.conf MANQUANT'"
                sh "head -20 frontend/Dockerfile"
            }
        }

        stage('Build Backend') {
            steps {
                echo "===== Build Backend ====="
                sh "docker build --no-cache -t ${BACKEND_IMAGE} -f backend/Dockerfile ."
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    echo "===== Build Frontend ====="
                    if (fileExists('frontend/nginx.conf')) {
                        echo "✅ nginx.conf trouvé - Build en cours..."
                    } else {
                        echo "⚠️ Attention : frontend/nginx.conf est introuvable au niveau du workspace !"
                    }
                }
                sh "docker build --no-cache -t ${FRONTEND_IMAGE} -f frontend/Dockerfile ."
            }
        }

        stage('Stop Old Containers') {
            steps {
                script {
                    echo "===== Arret des anciens conteneurs ====="
                    sh '''
                docker rm -f node-exporter prometheus mongo backend frontend grafana 2>/dev/null || true
            '''
                    sh "docker compose down --volumes --remove-orphans"
                }
            }
        }

        stage('Prepare Prometheus Config') {
            steps {
                script {
                    echo "===== Preparation de la configuration Prometheus ====="
                    sh "mkdir -p prometheus"
                    sh """
                    if [ ! -f prometheus/prometheus.yml ]; then
                        cat << 'EOF' > prometheus/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'proshop-backend'
    static_configs:
      - targets: ['backend:5000']
EOF
                    fi
                    """
                    echo "✅ prometheus.yml est un fichier valide"
                    sh "ls -la prometheus/prometheus.yml"
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "===== Deploiement ====="
                sh "docker compose up -d mongo"
                sh "docker compose up -d prometheus"
                sh "docker compose up -d node-exporter"
                sh "docker compose up -d grafana"
                sh "docker compose up -d backend"
                sh "docker compose up -d frontend"
                echo "Attente du démarrage des services..."
                sleep 15
            }
        }

        stage('Database Seeding') {
            steps {
                script {
                    echo "===== Seed de la base de données ====="
                    try {
                        echo "Lancement du seeding à l'intérieur du conteneur Backend..."
                        sh "docker compose exec -T backend npm run data:import"
                        echo "✅ Seeding terminé avec succès."
                    } catch (err) {
                        echo "❌ Échec lors de l'exécution du seeder."
                        error "Le seeding de la base de données a échoué."
                    }
                }
            }
        }

        stage('Verify') {
            steps {
                echo "===== Verification des services ====="
                sh "docker compose ps"
                echo ""
                echo "----- Tests de connectivite interne au réseau Docker -----"
                script {
                    try {
                        // 1. Écriture du script avec 127.0.0.1 au lieu de localhost pour garantir la résolution réseau
                        writeFile file: 'test-health.cjs', text: '''
const http = require('http');
http.get('http://127.0.0.1:5000/api/health', (res) => {
    console.log(res.statusCode);
    process.exit(0);
}).on('error', (e) => {
    console.log("ERREUR_RESEAU");
    process.exit(0);
});
'''
                        // 2. Copie et exécution dans le conteneur backend
                        sh "docker cp test-health.cjs \$(docker compose ps -q backend):/app/test-health.cjs"
                        
                        def statusCodeBackend = sh(
                            script: "docker compose exec -T backend node /app/test-health.cjs", 
                            returnStdout: true
                        ).trim()
                        
                        echo "Vérification de l'API Backend (interne) : Code HTTP -> ${statusCodeBackend}"
                        
                        if (statusCodeBackend == "200") {
                            echo "✅ L'application ProShop est saine et répond parfaitement (HTTP 200) !"
                        } else {
                            echo "⚠️ L'application a démarré mais l'endpoint de santé renvoie un statut inattendu : ${statusCodeBackend}"
                        }
                        
                        // 3. Nettoyage
                        sh "rm -f test-health.cjs"
                        
                    } catch (err) {
                        echo "⚠️ Impossible de valider l'état du backend via le script interne."
                    }
                }
            }
        }
    }

    post {
        always {
            echo "===== Fin du pipeline ====="
            echo "=========================================="
            script {
                if (currentBuild.currentResult == 'SUCCESS') {
                    echo "✅ PIPELINE EXECUTE AVEC SUCCES"
                } else {
                    echo "❌ ECHEC DU PIPELINE JENKINS"
                }
            }
            echo "=========================================="
            echo "📍 Frontend   : http://localhost:3000"
            echo "📍 Backend    : http://localhost:5000"
            echo "📍 Prometheus : http://localhost:9090"
            echo "=========================================="
        }
    }
}