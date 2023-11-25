pipeline {
    agent any
    environment {
        nombreBaseDeDatos = "Employees"
        archivoSQL = "sqlite.sql"
        nombreBackup="backup.db"
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
                  if (fileExists("${nombreBackup}.db")) {
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

   
}

