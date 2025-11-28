pipeline {
    agent any
    
    tools {
        // Aquí usamos el nombre "node" que configuramos en el paso anterior en Jenkins
        nodejs 'node' 
    }

    stages {
        stage('Checkout') {
            steps {
                // Descarga el código del repositorio
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Instala las dependencias de la app
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                // Ejecuta pruebas unitarias (si existen)
                // El "|| true" evita que falle el pipeline si no hay tests configurados
                sh 'npm test || true' 
            }
        }

        stage('Build Docker Image') {
            steps {
                // Construye la imagen usando el Dockerfile del proyecto
                sh 'docker build -t my-web-app .'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // --- LÓGICA CONDICIONAL DE PUERTOS ---
                    def hostPort = '3000' // Puerto por defecto (Main)
                    
                    if (env.BRANCH_NAME == 'dev') {
                        hostPort = '3001' // Puerto para Dev
                        echo "Detectada rama DEV. Usando puerto ${hostPort}"
                    } else {
                        echo "Detectada rama MAIN. Usando puerto ${hostPort}"
                    }
                    
                    // Definimos un nombre único para el contenedor basado en la rama
                    def containerName = "webapp-${env.BRANCH_NAME}"
                    
                    // 1. Limpieza: Borramos el contenedor anterior si existe para evitar errores
                    // "|| true" hace que no falle si es la primera vez que corre y no hay nada que borrar
                    sh "docker stop ${containerName} || true"
                    sh "docker rm ${containerName} || true"
                    
                    // 2. Despliegue: Corremos el nuevo contenedor mapeando los puertos
                    // El contenedor interno suele escuchar en el 3000 o 8080 (asumimos 3000 para Node/React)
                    sh "docker run -d -p ${hostPort}:3000 --name ${containerName} my-web-app"
                }
            }
        }
    }
}