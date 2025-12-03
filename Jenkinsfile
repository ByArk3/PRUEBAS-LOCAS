pipeline {

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    } 
    environment {
        // Asegúrate de que el ID de credencial 'AWS_ACCESS_KEY_ID' contenga AMBOS (Access Key ID y Secret Access Key).
        // Si usaste el tipo 'Username with password', el ID de la credencial debe contener el Access Key ID como Username.
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

   agent  any
    stages {
        
        // --- ELIMINAR EL STAGE DE CHECKOUT ---
        // stage('checkout') { ... } 
        // -------------------------------------

        stage('Plan') {
            steps {
                script {
                    // Usamos el bloque 'dir' para asegurarnos de que todos los comandos
                    // de Terraform se ejecuten DENTRO de la subcarpeta 'terraform'.
                    dir("terraform") {
                        // El 'pwd' ya no es necesario, el contexto es 'terraform/'
                        sh 'terraform init'
                        sh "terraform plan -out tfplan"
                        sh 'terraform show -no-color tfplan > tfplan.txt'
                    }
                }
            }
        }
        
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
           }

           steps {
               script {
                   // La ruta aquí es 'terraform/tfplan.txt' porque Jenkins siempre
                   // busca el archivo desde la raíz del workspace.
                   def plan = readFile 'terraform/tfplan.txt' 
                   input message: "Do you want to apply the plan?",
                   parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            steps {
                script {
                    dir("terraform") {
                        sh "terraform apply -input=false tfplan"
                    }
                }
            }
        }
    }

  }