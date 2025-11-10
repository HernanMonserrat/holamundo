pipeline {
  agent {
    docker {
      image 'php:8.2-cli'
      // Publica el puerto 9000 del contenedor al host
      args '-p 9000:9000'
    }
  }

  stages {
    stage('Clonar repo') {
      steps {
        git branch: 'main', url: 'https://github.com/HernanMonserrat/holamundo'
      }
    }

    stage('Levantar PHP y mantener vivo') {
      steps {
        sh '''
          echo "Iniciando servidor PHP..."
          nohup php -S 0.0.0.0:9000 -t . > server.log 2>&1 &
          echo "Servidor en: http://localhost:9000"
          # Mantener el job activo para que puedas probar desde el navegador
          tail -f server.log
        '''
      }
    }
  }
}
