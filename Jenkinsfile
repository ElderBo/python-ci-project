pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
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
          set -eux
          . "$VENV/bin/activate"
          flake8 app tests
        '''
      }
    }

    stage('Test (pytest)') {
        steps {
            sh '''
            set -eux
            . "$VENV/bin/activate"
            mkdir -p reports
            pytest -q --junitxml=reports/junit.xml
            '''
        }
    }


    stage('Package (source artifact)') {
        steps {
            sh '''
            set -eux
            mkdir -p dist
            # Create a build artifact from the exact source Jenkins tested
            tar -czf "dist/source-${BUILD_NUMBER}.tar.gz" .
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
