pipeline {
  agent any

  stages {
    stage('Instalar PHP con Homebrew') {
      steps {
        sh '''
          # Instala PHP si no existe
          if ! command -v php >/dev/null 2>&1; then
            echo "Instalando PHP con Homebrew..."
            brew update
            brew install php
          else
            echo "PHP ya está instalado."
          fi

          php -v
        '''
      }
    }

    stage('Clonar repositorio') {
      steps {
        git branch: 'main', url: 'https://github.com/HernanMonserrat/holamundo'
      }
    }

    stage('Levantar servidor PHP') {
      steps {
        sh '''
          nohup php -S 0.0.0.0:9000 -t . > server.log 2>&1 &
          echo "Servidor PHP corriendo en http://localhost:9000/"
          tail -f server.log
        '''
      }
    }
  }
}
