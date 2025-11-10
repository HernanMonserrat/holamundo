pipeline {
  agent {
    docker {
      image 'php:8.2-cli'
      args '-u root -p 9000:9000'   // <-- ejecuta como root y publica el puerto
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
    stage('Instalar PHP y utilitarios (con permisos admin)') {
      steps {
        sh '''
          set -e
          echo "Actualizando e instalando PHP si es necesario..."
          if command -v apt-get >/dev/null 2>&1; then
            apt-get update -y
            apt-get install -y php-cli php-common php-curl php-mbstring git curl
          elif command -v apk >/dev/null 2>&1; then
            apk add --no-cache php php-cli php-curl php-mbstring git curl
          else
            echo "Sistema no reconocido, saltando instalación automática de PHP"
          fi
          php -v
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
          pkill -f "php -S 0.0.0.0:${PORT}" || true

          echo "Iniciando servidor PHP en 0.0.0.0:${PORT}…"
          nohup php -S 0.0.0.0:${PORT} -t . > server.log 2>&1 &
          sleep 1

          # Verificar que responde
          for i in $(seq 1 20); do
            if curl -sSf "http://localhost:${PORT}/" >/dev/null 2>&1; then
              echo "OK -> http://localhost:${PORT}/"
              break
            fi
            sleep 0.5
          done

          echo "Servidor activo. Abre en tu navegador: http://localhost:${PORT}/"
          exec tail -f server.log
        '''
      }
    }
  }
}
