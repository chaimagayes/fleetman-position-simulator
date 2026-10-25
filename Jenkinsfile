pipeline {
  agent any
  parameters {
    booleanParam(name: 'RUN_SONAR', defaultValue: false, description: 'Analyse SonarQube (SonarQube doit etre demarre)')
    booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploiement Kubernetes (Minikube doit etre demarre)')
  }
  environment {
    DOCKERHUB_USER = 'chaimagayes'
    IMAGE = "${DOCKERHUB_USER}/fleetman-position-simulator"
    JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
    MAVEN_OPTS = '-Xmx512m'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Maven') {
      steps { sh 'mvn -B -DskipTests clean package' }
    }
    stage('SonarQube Analysis') {
      when { expression { params.RUN_SONAR } }
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.0.0.4389:sonar -Dsonar.projectKey=fleetman-position-simulator -Dsonar.projectName=Fleetman-Position-Simulator -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
        }
      }
    }
    stage('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $IMAGE:$BUILD_NUMBER
            docker push $IMAGE:latest
            docker logout
          '''
        }
      }
    }
    stage('Deploy to Kubernetes') {
      when { expression { params.DEPLOY } }
      steps {
        sh '''
          kubectl set image deployment/position-simulator position-simulator=$IMAGE:$BUILD_NUMBER
          kubectl rollout status deployment/position-simulator --timeout=180s
        '''
      }
    }
  }
}
