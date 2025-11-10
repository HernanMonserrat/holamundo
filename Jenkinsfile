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
      options { timeout(time: 60, unit: 'SECONDS') }
      steps {
        sh '''
          set -euxo pipefail
          echo "=== DIAGNOSTICO ==="
          echo "Usuario: $(whoami || true)"
          echo "Shell  : $SHELL"
          echo "OS     : $(uname -a || true)"
          echo "PATH   : $PATH"
          which sh || true
          command -v php || true
          ls -l /opt/homebrew/bin/php 2>/dev/null || true
          ls -l /usr/local/bin/php 2>/dev/null || true
          if ! command -v php >/dev/null 2>&1; then
            echo "ERROR: PHP no está en PATH. Agrega /opt/homebrew/bin:/usr/local/bin:${PATH} en Manage Jenkins → System o instala PHP."
            exit 1
          fi
          php -v | head -n1
          echo "=== FIN DIAGNOSTICO ==="
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
          set -euxo pipefail

          # Evitar conflicto de puerto
          if command -v lsof >/dev/null 2>&1 && lsof -iTCP:${PORT} -sTCP:LISTEN -Pn >/dev/null 2>&1; then
            echo "ERROR: El puerto ${PORT} ya está en uso."
            lsof -iTCP:${PORT} -sTCP:LISTEN -Pn || true
            exit 2
          fi

          # Apagar instancia previa (si la hubiera)
          pkill -f "php -S 0.0.0.0:${PORT}" || true

          test -f index.php || (echo "No se encontró index.php en $(pwd)"; ls -la; exit 1)

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

          echo "Servidor activo. Dejando el job vivo mostrando los logs…"
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
