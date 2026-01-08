pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    VENV = ".venv"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Info') {
      steps {
        sh '''
          set -eux
          python3 --version
          pip3 --version || true
          uname -a
        '''
      }
    }

    stage('Create venv & install deps') {
      steps {
        sh '''
          set -eux
          python3 -m venv "$VENV"
          . "$VENV/bin/activate"
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Lint (flake8)') {
      steps {
        sh '''
            set -eu
            . "$VENV/bin/activate"
            flake8 app tests
        '''
      }
    }

    stage('Test (pytest)') {
        steps {
            sh '''
                set -eu
                . "$VENV/bin/activate"
                mkdir -p reports
                export PYTHONPATH="$PWD"
                pytest -q --junitxml=reports/junit.xml
            '''
        }
    }


    stage('Package (source artifact)') {
        steps {
            sh '''
                set -eux
                mkdir -p dist
                tar -czf "dist/source-${BUILD_NUMBER}.tar.gz" \
                app requirements.txt
                ls -lah dist
            '''
        }
    }

  }

  post {
    always {
      // If the reports folder doesn't exist, Jenkins will just skip it
      junit allowEmptyResults: true, testResults: 'reports/junit.xml'
      archiveArtifacts allowEmptyArchive: true, artifacts: 'dist/*.tar.gz, reports/**'
      cleanWs()
    }
  }
}
