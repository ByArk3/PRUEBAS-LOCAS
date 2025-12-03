pipeline {

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    } 
    environment {
        // La clave de acceso de AWS y la clave secreta se inyectan aquí.
        // Asumiendo que has creado credenciales tipo 'Username with password' en Jenkins.
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

   agent  any
    stages {
        
        stage('Plan') {
            steps {
                script {
                    // *** CORRECCIÓN CRÍTICA DE LA RUTA ***
                    // Cambiamos dir("terraform") a dir("IAC/infra/terraform")
                    dir("IAC/infra/terraform") {
                        sh 'terraform init'
                        sh "terraform plan -out tfplan"
                        // Guardamos el plan en un archivo para el stage 'Approval'
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
                   // *** CORRECCIÓN CRÍTICA DE LA RUTA ***
                   // Leemos el archivo desde la ruta completa, ya que Jenkins lo busca desde la raíz del workspace.
                   def plan = readFile 'IAC/infra/terraform/tfplan.txt' 
                   
                   input message: "Do you want to apply the plan?",
                   parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            steps {
                script {
                    // *** CORRECCIÓN CRÍTICA DE LA RUTA ***
                    dir("IAC/infra/terraform") {
                        sh "terraform apply -input=false tfplan"
                    }
                }
            }
        }
    }

  }