pipeline {
    agent any
    
    stages {
        
        stage("Clean Up"){
            steps {
                deleteDir()
            }
        }
        
        stage("Clone Repo"){
            steps {
                sh "git clone https://github.com/subbasri17/Brain-Tasks-App.git"
            }
        }
        
        stage("Build"){
            steps {
                dir("Brain-Tasks-App") {
                    sh "docker build -t webserver ."
                }
            }
        }
        
        stage("Test"){
            steps {
                dir("Brain-Tasks-App") {
                    echo "Add test steps here"
                }
            }
        }  
    }
}
