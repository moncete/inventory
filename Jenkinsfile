def ent = env.BRANCH_NAME //Variable para el entorno de ejecucion

pipeline {

    agent { label 'vlish4200' }

    parameters { 
        text(name: 'Nodo', defaultValue: '', description:'Indicar nodo, para mas de uno separar por COMAS' )
        choice(name: 'Tipo', choices: ['server','agent'], description: 'Tipo de nodo por Consul')
        choice(name: 'Accion', choices: ['Crear','Borrar'], description: 'Accion a Ejecutar')
    }

    
    
    stages {

        stage("Checkout Repos") {

            parallel {

                stage("Clone Telegraf") {
                    
                    steps {
                        checkout(
                            [$class: 'GitSCM', 
                            branches: [[name: '*/master']], 
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [
                                [$class: 'RelativeTargetDirectory',relativeTargetDir: 'telegraf'],
                                [$class: 'LocalBranch', localBranch: 'master'], 
                                [$class: 'CleanBeforeCheckout']], 
                                submoduleCfg: [], 
                                userRemoteConfigs: [
                                    [credentialsId: 'bitbu', url: 'https://aqdes-stash.cm.es/stash/scm/arqdvo/ansible-devops.git']
                                ]
                            ]
                        )
                    }
                }
            
                stage("Clone Filebeat") {
                    
                    steps {
                        checkout(
                            [$class: 'GitSCM', 
                            branches: [[name: '*/master']], 
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [
                                [$class: 'RelativeTargetDirectory',relativeTargetDir: 'filebeat'],
                                [$class: 'LocalBranch', localBranch: 'master'],
                                [$class: 'CleanBeforeCheckout']], 
                                submoduleCfg: [], 
                                userRemoteConfigs: [
                                    [credentialsId: 'bitbu', url: 'https://aqdes-stash.cm.es/stash/scm/arqdvo/trazas-aws-devops.git']
                                ]
                            ]
                        )
                    }
                }
            }
        }

        stage("Dar de alta Nodo") {

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

        stage("Intalar Servicios") {
            
            when {
                expression { params.Accion == "Crear" }
            }

            parallel {

                stage("Instalar Filebeat") {
                   
                    steps {
                        ansiColor('xterm') {
                            ansiblePlaybook(
                                playbook: './filebeat/pb-filebeat-aws.yml',
                                inventory: "./filebeat/inventory/filebeat/container/filebeat-${ent}",
                                credentialsId: '343c97f1-a663-4d78-94ca-5ed66a5fb039',
                                limit: params.Nodo,
                                extraVars: [target: "${ent}", action: "start", mode: "container", ENTORNO: "${ent}", IMAGE_VERSION: "6.4.2"],
                                colorized: true
                            )
                        }

                    }

                }
                
                stage("Instalar Telegraf") {
                    
                    steps {
                        ansiColor('xterm') {
                            ansiblePlaybook(
                                playbook: './telegraf/pb-devops-infra.yml',
                                inventory: "./telegraf/inventory/infra_monitor_${ent}",
                                credentialsId: '343c97f1-a663-4d78-94ca-5ed66a5fb039',
                                limit: params.Nodo,
                                extraVars: [action: "start", SERVICE: "telegraf", ENTORNO: "${ent}", IMAGE_VERSION: "1.10"],
                                colorized: true 
                            )
                        }

                    }

                }
            }            
        }

        stage("Dar de baja Nodo") {

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
        
        stage("Actualizar Inventarios") {

            parallel {

                stage("Actualizar General") {
                   
                    steps {
                    
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

                stage("Actualizar Invetario Filebeat") {
                   
                    steps {
                        
                        withCredentials([usernamePassword(credentialsId: 'bitbu', passwordVariable: 'passwd', usernameVariable: 'user')]) {
                            sh """
                                cd ./filebeat/inventory/filebeat/container/
                                git add filebeat-${ent}
                                git commit -m \"Nodos para ${params.Accion} son ${params.Nodo}\"
                                git push --verbose -u https://${user}:'${passwd}'@aqdes-stash.cm.es/stash/scm/arqdvo/trazas-aws-devops.git  master
                            """
                        }
                    }
                }

                stage("Actualizar Invetario Telegraf") {
                    
                    steps {
                        
                        withCredentials([usernamePassword(credentialsId: 'bitbu', passwordVariable: 'passwd', usernameVariable: 'user')]) {
                            sh """
                                cd ./telegraf/inventory
                                git add infra_monitor_${ent}
                                git commit -m \"Nodos para ${params.Accion} son ${params.Nodo}\"
                                git push --verbose -u https://${user}:'${passwd}'@aqdes-stash.cm.es/stash/scm/arqdvo/ansible-devops.git  master
                            """
                        }
                    }
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
