pipeline {
    agent any

    environment {
        PHP_PORT = "50000"
    }

    stages {
        stage('Instalar PHP') {
            steps {
                echo 'Instalando PHP...'
                sh '''
                if ! command -v php >/dev/null 2>&1; then
                    sudo apt-get update -y
                    sudo apt-get install -y php
                fi
                '''
            }
        }

        stage('Crear página PHP') {
            steps {
                echo 'Creando archivo index.php...'
                sh '''
                mkdir -p workspace
                cd workspace
                cat << 'EOF' > index.php
                <?php
                echo "<h1>Hola Mundo desde PHP en Jenkins 🚀</h1>";
                ?>
                EOF
                '''
            }
        }

        stage('Levantar servidor PHP') {
            steps {
                echo "Iniciando servidor PHP en puerto ${PHP_PORT}..."
                sh '''
                cd workspace
                nohup php -S 0.0.0.0:${PHP_PORT} > servidor.log 2>&1 &
                echo $! > php_server.pid
                '''
                echo "Servidor iniciado. Accede en: http://localhost:${PHP_PORT}"
            }
        }

        stage('Verificar página') {
            steps {
                sh '''
                sleep 5
                curl -I http://localhost:${PHP_PORT} || true
                '''
            }
        }
    }

    post {
        always {
            echo "El servidor PHP sigue activo. Puedes acceder en:"
            echo "👉 http://localhost:${PHP_PORT}"
            echo "Para detenerlo manualmente: kill $(cat workspace/php_server.pid)"
        }
    }
}
