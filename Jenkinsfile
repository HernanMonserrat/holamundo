stage('Diagnóstico de entorno') {
  options { timeout(time: 60, unit: 'SECONDS') }   // evita cuelgues largos
  steps {
    sh '''
      set -euxo pipefail
      echo "=== DIAGNOSTICO ==="
      echo "Usuario: $(whoami || true)"
      echo "Shell  : $SHELL"
      echo "OS     : $(uname -a || true)"
      echo "PATH   : $PATH"

      # Dónde está sh y php
      which sh || true
      command -v php || true

      # Si estás en macOS con Homebrew, revisa si existe el binario esperado:
      ls -l /opt/homebrew/bin/php 2>/dev/null || true
      ls -l /usr/local/bin/php 2>/dev/null || true

      # No uses la forma { ... } que en algunos /bin/sh viejos se porta distinto
      if ! command -v php >/dev/null 2>&1; then
        echo "ERROR: PHP no está en PATH. Ajusta Manage Jenkins → System → PATH o instala PHP."
        exit 1
      fi

      php -v | head -n1
      echo "=== FIN DIAGNOSTICO ==="
    '''
  }
}
