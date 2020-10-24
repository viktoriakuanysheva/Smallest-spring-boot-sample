def withDockerNetwork(Closure inner) {
  try {
    networkId = UUID.randomUUID().toString()
    sh "docker network create ${networkId}"
    inner.call(networkId)
  } finally {
    sh "docker network rm ${networkId}"
  }
}

pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Building..'
                sh 'mvn -f ./SmallestSpringApp/pom.xml clean package'
                stash includes: 'SmallestSpringApp/target/*', name: 'app'
            }
        }
        stage('Build image') {
            agent any
            steps {
                unstash 'app'
                sh 'ls'
                sh 'docker build -t victoriakuanysheva/smallest-spring-app:${BUILD_NUMBER} .'
                sh 'docker tag victoriakuanysheva/smallest-spring-app:${BUILD_NUMBER} victoriakuanysheva/smallest-spring-app:latest'
            }
        }
        stage("test") {
            agent any
            steps {
               script{
                    withDockerNetwork{ n ->
                        docker.image('victoriakuanysheva/smallest-spring-app:latest').withRun("--network ${n} --name app") { c ->
                            docker.image('curlimages/curl').inside("--network ${n}") {
                                sh """
                                sleep 10;
                                curl app:8080;
                                """
                            }
                        }
                    }
                }
            }
        }
        stage('CD') {
            agent any
            steps {
                withDockerRegistry([ credentialsId: "docker-hub", url: "https://index.docker.io/v1/" ]) {
                    sh 'docker push victoriakuanysheva/smallest-spring-app:latest'
                }
            }
        }
    }
}
