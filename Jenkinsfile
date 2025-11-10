pipeline {
  agent any
  environment {
    // Ajusta esta ruta si tu "which php" devolvió otra distinta
    PHP_BIN = '/opt/homebrew/bin/php'  // o '/usr/local/bin/php'
  }
  stages {
    stage('Diagnóstico') {
      steps {
        sh '''
          echo "Usuario    : $(whoami)"
          echo "Shell      : $SHELL"
          echo "PATH       : $PATH"
          command -v php || true
          [ -x "$PHP_BIN" ] && echo "PHP_BIN ok en: $PHP_BIN" || echo "PHP_BIN no existe: $PHP_BIN"
        '''
      }
    }

    stage('Clonar repositorio') {
      steps {
        git branch: 'main', url: 'https://github.com/HernanMonserrat/holamundo'
      }
    }

    stage('Levantar servidor PHP (persistente)') {
      steps {
        sh '''
          # Verificaciones
          if [ ! -x "$PHP_BIN" ] && ! command -v php >/dev/null 2>&1; then
            echo "ERROR: PHP no está instalado o no está en PATH."
            echo "Soluciones:"
            echo "  1) brew install php"
            echo "  2) Agrega /opt/homebrew/bin o /usr/local/bin al PATH en Manage Jenkins > System."
            exit 1
          fi

          # Selecciona binario: preferimos PHP_BIN si existe
          if [ -x "$PHP_BIN" ]; then
            PHP_CMD="$PHP_BIN"
          else
            PHP_CMD="$(command -v php)"
          fi

          # Limpieza de servidor previo
          pkill -f "$PHP_CMD -S 0.0.0.0:9000" || true

          echo "Iniciando servidor con: $PHP_CMD"
          nohup "$PHP_CMD" -S 0.0.0.0:9000 -t . > server.log 2>&1 &

          echo "Servidor corriendo en http://localhost:9000 (o IP_del_agente:9000)"
          echo "Manteniendo el job activo para que el server siga disponible..."
          tail -f server.log
        '''
      }
    }
  }
}
