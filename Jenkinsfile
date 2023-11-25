pipeline {
    agent any
    environment {
        nombreBaseDeDatos = "Employees"
        archivoSQL = "sqlite.sql"
        nombreBackup="backup.db"
        GITHUB_REPO_OWNER = 'CarlosD21'
        GITHUB_REPO_NAME = 'PROF-2023-Ejercicio4'
        GITHUB_CONTEXT = 'Jenkins'
        TARGET_URL="http://ec2-54-224-87-86.compute-1.amazonaws.com:8080/"
        STATE_SUCCESS = 'success'
        STATE_FAILURE = 'failure'
        CONTEXT = 'Jenkins'
        DESCRIPTION_SUCCESS = 'Build successful'
        DESCRIPTION_FAILURE = 'Build failed'
   
       
    }
  

    stages {
        
        stage('Backup Base de Datos Actual') {
            steps {
                script {
                    // Verificar si la base de datos existe
                    def resultadoConsulta = sh(script: "sqlite3 ${nombreBaseDeDatos}.db 'SELECT * FROM employees LIMIT 1;' 2>&1", returnStatus: true)
                    if (resultadoConsulta == 0) {
                        // Realizar el backup con la fecha en el nombre
                        sh "sqlite3 ${nombreBaseDeDatos}.db \".backup ${nombreBackup}\""
                    }else {
                        echo "La base de datos no existe o no se puede acceder."
                    }
                }
            }
        }
		stage('Eliminar Base de Datos Actual') {
            steps {
                script {
                    // Verificar si la base de datos existe
                    def resultadoConsulta = sh(script: "sqlite3 ${nombreBaseDeDatos}.db 'SELECT * FROM employees LIMIT 1;' 2>&1", returnStatus: true)
                    if (resultadoConsulta == 0) {
                        // Elimina base de datos
                        sh "rm ${nombreBaseDeDatos}.db"  
                    }else {
                        echo "La base de datos no existe o no se puede acceder."
                    }
                }
            }
        }
		stage('Cargar Nueva Base de Datos') {
            steps {
                script {
                    // Ejecuta las declaraciones SQL desde el archivo para crear la nueva base de datos
                    sh "sqlite3 ${nombreBaseDeDatos}.db < ${archivoSQL}"
                }
            }
        }

		stage('Restaurar Base de Datos Anterior') {
            steps {
                script {
                  if (fileExists("${nombreBackup}")) {
                    // Restaurar la base de datos desde el archivo de backup
                    sh "sqlite3 ${nombreBaseDeDatos}.db \".restore ${nombreBackup}\""
                  }else{
                    echo "El archivo ${nombreBackup} no existe."
                  }
                }
            }
        }

       
        // Otras etapas del pipeline...
    }
  post {
        success {
            // Acciones a realizar en caso de éxito
            script {
                updateGitHubStatus(STATE_SUCCESS, DESCRIPTION_SUCCESS)
            }
        }
        failure {
            // Acciones a realizar en caso de fallo
            script {
                updateGitHubStatus(STATE_FAILURE, DESCRIPTION_FAILURE)
            }
        }
    }
    def updateGitHubStatus(state, description) {
         def COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()

         sh """
              curl -X POST \
              -H 'Accept: application/vnd.github.v3+json' \
              'https://api.github.com/repos/$REPO_OWNER/$REPO_NAME/statuses/$COMMIT_SHA' \
               -d '{"state": "$state", "context": "$CONTEXT", "description": "$description", "target_url": "$TARGET_URL"}'
             """
    }
    
   
}

