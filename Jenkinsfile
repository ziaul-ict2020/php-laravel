pipeline {
    agent {label "app" } 
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://192.168.1.127:8081/repository/glue-app/"
        NEXUS_REPOSITORY = "glue-app"
        NEXUS_CREDENTIAL_ID = "nexus"
        HOME_DIR = "/opt/lampp/htdocs/workspace/"
      }
    stages {
        stage ('Clean Artifacts') {
            steps {
                sh 'ls'
                sh 'rm -rf $HOME_DIR/artifact'
                sh 'rm -rf $HOME_DIR/Glue/artifact.tar.gz'
            }
        }
        stage ('Unit Test') {
            steps {
                sh 'php artisan make:test UserTest --unit'
            }
        }  
        stage ('GitHub Integration') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gluebucket', url: 'https://bitbucket.org/nahid210/glue/src/master/']]])
            }
        }   
        stage ('Larastan  Test') {
            steps {
                sh 'composer require --dev phpstan/phpstan'
                //sh 'vendor/bin/phpstan analyse tests'
                sh 'vendor/bin/phpstan analyse bootstrap'
                sh 'vendor/bin/phpstan analyse public'
            }
        }
        stage ('Codeception Test') {
            steps {
               sh 'composer require --dev codeception/codeception'
               sh 'composer require codeception/module-phpbrowser --dev'
               sh 'composer require codeception/module-asserts --dev'
			   //sh 'php vendor/bin/codecept bootstrap'
               sh 'php vendor/bin/codecept run --steps'
           }
        }
        
        stage ('PHPCS  Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                //sh 'phpcs app/Http/Controllers/TestController.php'
                sh 'phpcs app/Http/Controllers/Admin/FeatureBoxController.php'
                sh "exit 1"
            }
        }
     }
        stage ('Build') {
            steps {
                sh 'cp .env.example .env'
                sh 'composer update'
                sh 'cp -r $HOME_DIR/Glue  $HOME_DIR/artifact'
                sh 'tar -czvf $HOME_DIR/artifact.tar.gz  $HOME_DIR/artifact'
                sh 'mv $HOME_DIR/artifact.tar.gz $HOME_DIR/Glue'
                sh 'ls -la'
            }
        }
        stage ('SonarQube') {
            steps {
                 sh '  sonar-scanner -X '
                 sh '  -Dsonar.projectKey=glue '
                 sh '  -Dsonar.sources=./lampp/htdocs/glue'
                 sh '  -Dsonar.host.url=http://192.168.122.151:9000 '
                 sh '  -Dsonar.login=dc216b1aecd8b647fd7c4d269bd64729ad39dadc '
            }
        }
    
        stage ('Upload Artifact') {
            steps {
                sh 'curl -v -u ${nexus_admin}:${nexus_pass} --upload-file artifact.tar.gz ${NEXUS_URL}'
            }
        }     
        stage ('Deploy') {
            steps {
                sh 'nohup php artisan serve --host=0.0.0.0 --port=8000 &'
            }
        }
}
}