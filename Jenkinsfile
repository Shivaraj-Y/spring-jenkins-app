pipeline {
  agent any

  environment {
    APP_PORT = "8081"
    JAR_NAME = "app.jar" // change to actual jar name after build if needed
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        // if repository uses maven wrapper
        sh '''
          if [ -f ./mvnw ]; then
            ./mvnw -B clean package -DskipTests
          else
            mvn -B clean package -DskipTests
          fi
        '''
      }
      post {
        success {
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
      }
    }

    stage('Deploy') {
      steps {
        // deploy on the same machine where Jenkins runs (app server)
        sh '''
          # find jar produced in target/
          JAR=$(ls */target/*.jar | head -n 1 || true)
          if [ -z "$JAR" ]; then
            echo "Jar not found!"
            exit 1
          fi

          # copy jar to /opt/myapp/
          sudo mkdir -p /opt/myapp
          sudo cp "$JAR" /opt/myapp/${JAR_NAME}
          sudo chown -R $(whoami):$(whoami) /opt/myapp

          # kill anything on APP_PORT
          PID=$(lsof -ti tcp:${APP_PORT} || true)
          if [ -n "$PID" ]; then
            echo "Killing old process on port ${APP_PORT}: $PID"
            sudo kill -9 $PID || true
          fi

          # start jar in background with nohup
          nohup java -jar /opt/myapp/${JAR_NAME} --server.port=${APP_PORT} > /opt/myapp/app.log 2>&1 &
          echo $! > /opt/myapp/app.pid
          sleep 5
          echo "Started app (PID=$(cat /opt/myapp/app.pid))"
          tail -n 50 /opt/myapp/app.log || true
        '''
      }
    }
  }

  post {
    failure {
      echo "Build or deploy failed"
    }
    success {
      echo "Pipeline completed successfully"
    }
  }
}
