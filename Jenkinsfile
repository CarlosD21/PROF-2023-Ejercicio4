pipeline {
    agent any
    environment {
        nombreBaseDeDatos = "Employees"
        archivoSQL = "sqlite.sql"
        nombreBackup="backup.db"
        GITHUB_TOKEN = credentials('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')
        GITHUB_REPO_OWNER = 'CarlosD21'
        GITHUB_REPO_NAME = 'PROF-2023-Ejercicio4'
        GITHUB_CONTEXT = 'Jenkins'
        GITHUB_TARGET_URL = 'http://ec2-54-224-87-86.compute-1.amazonaws.com:8080/'
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
        stage('Create GitHub Status') {
            steps {
                script {
                    def commitHash = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()


                    // Enviar el estado de verificación a GitHub en caso de éxito
                    if (currentBuild.resultIsBetterOrEqualTo('SUCCESS')) {
                        githubCommitStatus(
                            credentialsId: GITHUB_TOKEN,
                            repoOwner: GITHUB_REPO_OWNER,
                            repository: GITHUB_REPO_NAME,
                            commitSha: commitHash,
                            status: githubStatus
                        )
                    } else {
                        // Cambiar el estado de verificación a 'failure' en caso de fallo
                        githubStatus.state = 'failure'
                        githubStatus.description = 'Build failed'
                        githubCommitStatus(
                            credentialsId: GITHUB_TOKEN,
                            repoOwner: GITHUB_REPO_OWNER,
                            repository: GITHUB_REPO_NAME,
                            commitSha: commitHash,
                            status: githubStatus
                        )
                    }
                }
            }
        }
        // Otras etapas del pipeline...
    }

   
}

