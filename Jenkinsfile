pipeline {
  agent {
    docker {
      image 'php:8.2-cli'          // PHP listo
      args '-p 9000:9000'          // Publica el puerto al host
      reuseNode true
    }
  }

  options { timestamps() }

  environment {
    REPO = 'https://github.com/HernanMonserrat/holamundo'
    BRANCH = 'main'
    PORT = '9000'
  }

  stages {
    stage('Preparar herramientas') {
      steps {
        sh '''
          set -e
          # Si la imagen no trae git/curl, los instalamos rápido (Debian o Alpine)
          if ! command -v git >/dev/null 2>&1; then
            if command -v apt-get >/dev/null 2>&1; then
              apt-get update && apt-get install -y git curl
            elif command -v apk >/dev/null 2>&1; then
              apk add --no-cache git curl
            fi
          fi
        '''
      }
    }

    stage('Clonar repo') {
      steps {
        sh '''
          set -e
          rm -rf .git || true
          git init
          git remote add origin "$REPO"
          git fetch --depth 1 origin "$BRANCH"
          git checkout -f FETCH_HEAD
          ls -la
          test -f index.php || (echo "No se encontró index.php" && exit 1)
        '''
      }
    }

    stage('Levantar PHP y mantener vivo') {
      steps {
        sh '''
          set -e
          # Parar servidor previo si existiera
          pkill -f "php -S 0.0.0.0:${PORT}" || true

          echo "Iniciando servidor PHP en 0.0.0.0:${PORT}…"
          nohup php -S 0.0.0.0:${PORT} -t . > server.log 2>&1 &
          sleep 1

          # Comprobación rápida
          for i in $(seq 1 20); do
            if curl -sSf "http://localhost:${PORT}/" >/dev/null 2>&1; then
              echo "OK -> http://localhost:${PORT}/"
              break
            fi
            sleep 0.5
          done

          echo "Servidor activo. Abre: http://localhost:${PORT}/"
          # Deja el job vivo para que puedas navegar
          exec tail -f server.log
        '''
      }
    }
  }
}
