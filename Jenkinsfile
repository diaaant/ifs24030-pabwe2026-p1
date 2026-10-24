// Pipeline CI/CD BengkelKu: lint, test, build, lalu deploy dengan Docker Compose.
// Prasyarat Jenkins: plugin Docker Pipeline, serta docker dan docker compose pada agent.
pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  parameters {
    booleanParam(name: 'DEPLOY', defaultValue: true, description: 'Jalankan tahap deploy setelah pengujian lulus')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Backend: lint dan test') {
      agent { docker { image 'node:22-bookworm'; reuseNode true } }
      environment { HOME = "${WORKSPACE}" }
      steps {
        dir('backend') {
          sh 'npm ci'
          sh 'npm run lint'
          sh 'npm run test:coverage'
        }
      }
    }

    stage('Frontend: test dan build') {
      agent { docker { image 'node:22-bookworm'; reuseNode true } }
      environment { HOME = "${WORKSPACE}" }
      steps {
        dir('frontend') {
          sh 'npm ci'
          sh 'npm run test:coverage'
          sh 'npx nuxt generate'
        }
      }
    }

    stage('Deploy') {
      when { expression { params.DEPLOY } }
      steps {
        sh 'docker compose up -d --build'
        sh 'sleep 8 && curl -fsS http://localhost:8000/api/health'
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'backend/coverage/**, frontend/coverage/**', allowEmptyArchive: true
    }
    success { echo 'Pipeline berhasil: pengujian lulus dan aplikasi ter-deploy.' }
    failure { echo 'Pipeline gagal: periksa log tahap yang gagal.' }
  }
}
