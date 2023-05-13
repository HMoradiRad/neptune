pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                script {
                    neptunImage = docker.build("192.168.56.30/neptun:latest")
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.image("mysql:8").withRun("--name $JOB_NAME-$BUILD_ID-mysql-test -e MYSQL_ROOT_PASSWORD=testing -e MYSQL_DATABASE=testing") {
                     
                      container ->
                      neptunImage.inside("--name $JOB_NAME-$BUILD_ID-api-test -e NEPTUNE_API_ENV=testing -e NEPTUNE_API_DEBUG=1 -e NEPTUNE_API_TESTING=1 -e NEPTUNE_API_DATABASE_URI=mysql+pymysql://root:testing@$JOB_NAME-$BUILD_ID-mysql-test:3306/testing --link $container.id") {
                          retry(90) {
                              sh "flask app test"
                              sleep(5)
                          }
                          sh "flask db upgrade"
                          sh "coverage run -m pytest --junit-xml=junit.xml"
                          sh "coverage html"
                          sh "coverage xml"
                          
                      }
                    }
                    
                }
            }
        }
        stage('Relese') {
            steps {
                echo "stage relese"
            }
        }
    }
}
