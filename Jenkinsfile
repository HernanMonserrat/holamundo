pipeline {
  agent any
  options { timestamps() }
  environment {
    REPO   = 'https://github.com/HernanMonserrat/holamundo'
    BRANCH = 'main'
    PORT   = '9000'
  }

  stages {
    stage('Diagnóstico de entorno') {
      steps {
        sh '''
          set -e
          echo "Usuario Jenkins: $(whoami)"
          echo "PATH: $PATH"
          command -v php || { echo "ERROR: PHP no está en PATH"; exit 1; }
          php -v | head -n1
        '''
      }
    }

    stage('Checkout') {
      steps {
        git branch: "${BRANCH}", url: "${REPO}"
      }
    }

    stage('Levantar PHP (persistente)') {
      steps {
        sh '''
          set -e
          # Evitar conflicto de puerto
          if command -v lsof >/dev/null 2>&1 && lsof -iTCP:${PORT} -sTCP:LISTEN -Pn >/dev/null 2>&1; then
            echo "ERROR: El puerto ${PORT} está en uso."
            lsof -iTCP:${PORT} -sTCP:LISTEN -Pn || true
            exit 2
          fi

          # Apagar servidor previo
          pkill -f "php -S 0.0.0.0:${PORT}" || true

          echo "Iniciando servidor en 0.0.0.0:${PORT}…"
          nohup php -S 0.0.0.0:${PORT} -t . > server.log 2>&1 &
          sleep 1

          # Smoke test
          for i in $(seq 1 20); do
            if curl -sSf "http://localhost:${PORT}/" >/dev/null 2>&1; then
              echo "OK -> http://localhost:${PORT}/"
              break
            fi
            sleep 0.5
          done

          echo "Servidor activo. Dejo el job vivo para que navegues."
          exec tail -f server.log
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'server.log', allowEmptyArchive: true
    }
  }
}
