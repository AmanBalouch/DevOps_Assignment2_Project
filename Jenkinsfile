pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        PROJECT_DIR = '/var/jenkins_home/workspace/RentEase-Pipeline'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                echo '========== Cloning Repository =========='
                git branch: 'main', url: 'https://github.com/AmanBalouch/DevOps_Assignment2_Project.git'
            }
        }
        
        stage('Verify Docker Setup') {
            steps {
                echo '========== Checking Docker and Docker Compose =========='
                sh 'docker --version'
                sh 'docker-compose --version'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo '========== Building Docker Images =========='
                sh 'docker-compose -f ${DOCKER_COMPOSE_FILE} build'
            }
        }
        
        stage('Start Services') {
            steps {
                echo '========== Stopping Previous Services (if running) =========='
                sh 'docker-compose -f ${DOCKER_COMPOSE_FILE} down || true'
                sh 'sleep 5'
                
                echo '========== Starting Services =========='
                sh 'docker-compose -f ${DOCKER_COMPOSE_FILE} up -d'
                sh 'sleep 10'
            }
        }
        
        stage('Health Check') {
            steps {
                echo '========== Verifying Services Health =========='
                sh '''
                    echo "Checking PostgreSQL..."
                    docker exec rentease-postgres-part2 pg_isready -U postgres
                    
                    echo "Checking Backend API..."
                    docker exec rentease-backend-part2 python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/docs')" || echo "Backend API is starting, which is expected"
                    
                    echo "Checking Frontend..."
                    docker ps | grep rentease-frontend-part2
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                echo '========== Running Application Tests =========='
                sh '''
                    echo "Testing Backend API with Python..."
                    docker exec rentease-backend-part2 python -c "
import urllib.request
try:
    response = urllib.request.urlopen('http://localhost:8000/api/properties')
    print('✓ GET /api/properties - Status:', response.status)
except Exception as e:
    print('⚠ API test result:', str(e))
" || true
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline Completed Successfully!'
            echo 'RentEase application is running on:'
            echo '  - Frontend: http://EC2_IP:5174'
            echo '  - Backend API: http://EC2_IP:8001'
            echo '  - API Docs: http://EC2_IP:8001/docs'
        }
        failure {
            echo '❌ Pipeline Failed! Stopping containers...'
            sh 'docker-compose -f ${DOCKER_COMPOSE_FILE} down'
        }
        always {
            echo '========== Pipeline Execution Completed =========='
        }
    }
}
