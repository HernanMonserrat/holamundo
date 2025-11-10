pipeline {
  agent any
  options { timestamps() }

  parameters {
    choice(name: 'ACTION', choices: ['start','stop'], description: 'Iniciar o detener el servidor PHP')
    string(name: 'BRANCH', defaultValue: 'main', description: 'Rama a clonar')
    string(name: 'PORT', defaultValue: '9000', description: 'Puerto HTTP para servir la app')
  }

  environment {
    REPO_URL = 'https://github.com/HernanMonserrat/holamundo'
    PID_FILE = 'php_server.pid'
    DOCROOT  = "${env.WORKSPACE}"
  }

  stages {
    stage('Checkout') {
      when { expression { params.ACTION == 'start' } }
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: "*/${params.BRANCH}"]],
          userRemoteConfigs: [[url: "${REPO_URL}"]]
        ])
      }
    }

    stage('Verificar PHP & index.php') {
      when { expression { params.ACTION == 'start' } }
      steps {
        sh '''
          if ! command -v php >/dev/null 2>&1; then
            echo "ERROR: php no está instalado en este agente Jenkins."
            exit 1
          fi
          if [ ! -f index.php ]; then
            echo "ERROR: No se encontró index.php en ${PWD}"
            ls -la
            exit 1
          fi
        '''
      }
    }

    stage('Iniciar servidor PHP') {
      when { expression { params.ACTION == 'start' } }
      steps {
        sh '''
          if [ -f "${PID_FILE}" ] && ps -p $(cat ${PID_FILE}) >/dev/null 2>&1; then
            echo "El servidor ya está corriendo con PID $(cat ${PID_FILE})."
            exit 0
          fi

          # Comprobar puerto libre
          if command -v lsof >/dev/null 2>&1 && lsof -iTCP:${PORT} -sTCP:LISTEN -Pn >/dev/null 2>&1; then
            echo "ERROR: El puerto ${PORT} ya está en uso."
            lsof -iTCP:${PORT} -sTCP:LISTEN -Pn || true
            exit 2
          fi

          nohup php -S 0.0.0.0:${PORT} -t "${DOCROOT}" > "${WORKSPACE}/php-${PORT}.log" 2>&1 &
          echo $! > "${PID_FILE}"
          sleep 1

          if ! ps -p $(cat ${PID_FILE}) >/dev/null 2>&1; then
            echo "ERROR: No se pudo iniciar el servidor PHP."
            tail -n +200 "${WORKSPACE}/php-${PORT}.log" || true
            exit 3
          fi
          echo "Servidor PHP iniciado. PID: $(cat ${PID_FILE})"
        '''
      }
    }

    stage('Smoke test') {
      when { expression { params.ACTION == 'start' } }
      steps {
        sh '''
          # Intento simple para verificar respuesta
          for i in $(seq 1 20); do
            if curl -sS "http://localhost:${PORT}/" >/dev/null 2>&1; then
              echo "Servidor respondiendo en localhost:${PORT}"
              break
            fi
            sleep 0.5
          done

          # Mejor hint de URL accesible en red local
          HOST="localhost"
          if command -v hostname >/dev/null 2>&1; then
            if hostname -I >/dev/null 2>&1; then
              HOST=$(hostname -I 2>/dev/null | awk '{print $1}')
            elif command -v ipconfig >/dev/null 2>&1; then
              HOST=$(ipconfig getifaddr en0 2>/dev/null || echo "localhost")
            fi
          fi

          echo "Prueba en el navegador:"
          echo "  • Desde este host:   http://localhost:${PORT}/"
          echo "  • Desde tu red LAN:  http://${HOST}:${PORT}/"
        '''
      }
    }

    stage('Detener servidor PHP') {
      when { expression { params.ACTION == 'stop' } }
      steps {
        sh '''
          if [ -f "${PID_FILE}" ]; then
            PID=$(cat ${PID_FILE})
            if ps -p $PID >/dev/null 2>&1; then
              kill $PID || true
              sleep 1
            fi
            rm -f "${PID_FILE}"
            echo "Servidor PHP detenido."
          else
            echo "No hay PID file; nada que detener."
          fi
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: "php-${PORT}.log", allowEmptyArchive: true
      echo "Para detener el servidor: vuelve a ejecutar el build con ACTION=stop"
    }
  }
}
