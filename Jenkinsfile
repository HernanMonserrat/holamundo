pipeline {
  agent any
  options { timestamps() }
  stages {
    stage('Instalar PHP') {
      steps {
        sh '''
          set -e
          if ! command -v php >/dev/null 2>&1; then
            echo "Instalando PHP..."
            DEBIAN_FRONTEND=noninteractive apt-get install -y php
          fi
          php -v
        '''
      }
    }

    stage('Levantar servidor PHP') {
      steps {
        sh '''
          pkill -f "php -S 0.0.0.0:8085" || true
          nohup php -S 0.0.0.0:8085 -t . > server.log 2>&1 &
          sleep 2
          pgrep -af "php -S 0.0.0.0:8085"
        '''
      }
    }

    stage('Prueba de Hola Mundo') {
      steps {
        sh '''
          echo "Probando endpoint PHP..."
          curl -sS http://localhost:8085/index.php || true
          echo
          curl -I http://localhost:8085/index.php || true
        '''
      }
    }
  }
  post {
    always {
      echo 'Logs del servidor PHP:'
      sh 'tail -n 20 server.log || true'
    }
  }
}
