pipeline {
    agent any

    stages {
        stage('Clonar repositorio') {
            steps {
                git branch: 'main', url: 'https://github.com/HernanMonserrat/holamundo'
            }
        }

        stage('Levantar servidor PHP') {
            steps {
                sh '''
                    # Detener cualquier servidor previo
                    pkill -f "php -S 0.0.0.0:9000" || true

                    echo "Iniciando servidor PHP..."
                    nohup php -S 0.0.0.0:9000 -t . > server.log 2>&1 &
                    echo "Servidor PHP ejecutándose en http://localhost:9000"

                    # Mantener el pipeline activo
                    echo "Presiona Ctrl+C en Jenkins si deseas detenerlo manualmente."
                    tail -f server.log
                '''
            }
        }
    }
}
