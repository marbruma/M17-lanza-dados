// partir de aquí crear los siguientes Stages:
//• Limpieza del workspace.
//• Checkout del código de nuestro repositorio.
//• Contruimos la imagen del contenedor.
//• Probamos que se ejecuta correctamente. Posteriormente borramos el contenedor creado.
//• Subimos la imagen a Docker Hub.


pipeline{
    agent any // Le decimos que se ejecute en cualquier nodo, aunque solo tenemos uno 

    environment {
        // Definir las variables de entorno
        registry = 'mmartinbru/m17-cicd-lanzadados' 
        //las credenciales que va a usar jenkins de ese registry que serán necesarias para poder subir la imagen a DockerHub
        registriCredentials = 'dockerhub'   // credenciales de DockerHub en Jenkins
        project ="M17-CICD-jenkins" // Nombre del proyecto en Jenkins
        projectVersion= '1.0' // Versión del proyecto
        repository = "https://github.com/marbruma/M17-lanza-dados.git"
        repositoryCredentials="github" // Credenciales de GitHub en Jenkins
      
    }

    stages {
        // STAGE 1: Limpieza del workspace
        stage('Clean Workspace') {
            steps {
                cleanWs() // Plugin de Jenkins que limpia automáticamente el workspace. 
            }
        }

        // STAGE 2: Checkout del código
        stage('Checkout code') {
            steps {
                script {
                    git  branch  'main' ,
                            credentialsId: repositoryCredentials, 
                            url: repository
                }
            }
        }

        // STAGE 3: Construimos la imagen del contenedor
        stage('Build') {
            steps {
                script {
                    // plugin de docker para crear la imgen de nuestro conteneder a partir del Dockerfile que tenemos en el repositorio.
                    dockerImage = docker.build registry 
                }
            }
        }

        // STAGE 4: Probamos la ejecución y borramos el contenedor
        stage('Test') {
            steps {
                script {
                    try {
                        // Arrancamos el contenedor para verificar que no da fallos de sintaxis o importación
                        sh 'docker run --name $project -e $registry' 
                    } finally {
                        // El bloque 'finally' asegura que el contenedor se borre SIEMPRE, pase o falle el test
                        sh 'docker rm $project'
                    }
                }
            }
        }

        // STAGE 5: Subimos la imagen a Docker Hub
        stage('Deploy') {
            steps {
                script {
                    // Se conecta a DockerHub usando el ID de tus credenciales guardadas en Jenkins
                    docker.withRegistry('', registriCredentials) {
                        dockerImage.push()          // Sube la versión con el número de build actual
                        
                    }
                }
            }
        }
        stage ('Cleaning up'){
            steps {
                script {
                    // Limpiar la imagen local del servidor Jenkins 
                    sh 'docker rmi $registry'
                }
        }
   
    }

    

    // POST: Acciones que se ejecutan independientemente de si el pipeline tiene éxito o falla
    // Deberá mostrarse un mensaje: "El pipeline ha fallado." solo si el pipeline ha finalizado con fallo.
    post {
        failure {
            // Este mensaje solo se imprimirá en la consola si alguna fase del pipeline falla
            echo "El pipeline ha fallado."
        }
        always {
            echo  'Registrar Build'
            // Buena práctica: Limpiar la imagen local del servidor Jenkins para no agotar el disco
            //sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
        }
    }
}
}