def ent = env.BRANCH_NAME //Variable para el entorno de ejecucion

pipeline {

    agent { label 'vlish4200' }

    parameters { 
        text(name: 'Nodo', defaultValue: '', description:'Indicar nodo, para mas de uno separar por coma' )
        choice(name: 'Tipo', choices: ['server','agent'], description: 'Tipo de nodo por Consul')
        choice(name: 'Accion', choices: ['Crear','Borrar'], description: 'Accion a Ejecutar')
    }

    
    
    stages {

        stage ("Dar de alta Nodo") {

            when {
                expression { params.Accion == 'Crear' }
            }

            steps {
                 ansiColor('xterm') {
                    ansiblePlaybook (
                        playbook: 'provision.yml',
                        inventory: 'inventario',
                        credentialsId: '343c97f1-a663-4d78-94ca-5ed66a5fb039',
                        extraVars: [ ENT: "${ent}", TIPO: params.Tipo, host: params.Nodo ],
                        tags: 'Crear',
                        colorized: true
                    )                     
                }
            }
        }
        stage ("Dar de baja Nodo") {

            when {
                expression { params.Accion == "Borrar" }
            }

            steps {
                 ansiColor('xterm') {
                    ansiblePlaybook (
                        playbook: 'provision.yml',
                        inventory: 'inventario',
                        credentialsId: '343c97f1-a663-4d78-94ca-5ed66a5fb039',
                        extraVars: [ ENT: "${ent}", TIPO: params.Tipo, host: params.Nodo ],
                        tags: 'Borrar',
                        colorized: true
                    )                     
                }
            }
       }
        
        stage ("Actualizar Inventarios"){

            steps{

                withCredentials([usernamePassword(credentialsId: 'bitbu', passwordVariable: 'passwd', usernameVariable: 'user')]) {
                    sh """
                        git checkout
                        git add inventario
                        git commit -m \"Nodos para ${params.Accion} son ${params.Nodo}\"
                        git push --verbose -u https://${user}:'${passwd}'@aqdes-stash.cm.es/stash/scm/arqdvo/ansible-devops-maintenance.git  ${ent}
                    """
                }
            }
        }

    }

    post {
        always {
            sh "git clean -ffxd"
        }
    }

}