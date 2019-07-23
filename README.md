# Alta de Maquinas a traves de Jenkins   
 [![Build Status](http://vlibkd223:8081/buildStatus/icon?job=Alta_maquinas%2Fepd)](http://vlibkd223:8081/job/Alta_maquinas/job/epd/)


1. [ Descripción. ](#desc)

2. [ Ficheros y uso. ](#files)

3. [ Uso de inventario. ](#usage)

    3.1 [ Jenkinsfile. ](#Jenkins)

    3.2 [ Invetario.](#inv)

    3.3 [ Ficheros YML ](#yml)


<a name="desc"></a>
## 1. Descripción

  Servcio creado para dar de alta y baja los nodos en los entornos __PREVIOS__

  Pensado para llevar un inventario completo de todas las maquinas que estan disponibles 

  por entorno.

<a name="usage"></a>
## 2. Uso del inventario

    - El fichero de inventario se actualiza despues de cada ejecucion de la tarea en Jenkins
    - Es utilizado tanto para dar de Alta maquinas como para darlas de Baja
    - IMPORTANTE: para dar mas de un noda a la vez, separar cada nodo por comas
            Ejemplo:
             vliamd222,vliamd232


<a name="desc"></a>
## 3. Ficheros y uso


   <a name="Jenkins"></a>
   ### 3.1 Jenkinsfile 

   -  Aquí está configurado todo la tarea de plataformado de maquinas, se hace por entornos

   se genera una tarea por cada entorno en el jenkins. __EPD__, __EPI1__, __EPI2__

   - Tambien se genera una rama por entorno para tener los inventarios separados

   <a name="inv"></a>
   ### 3.2 Invetario
   - Esta estructurado por tags __[entorno_server]__ o __[entorno_agent]__
       
       - __[entorno_server]__ : Orientado para los nodos que esten instalados con __consul_server__
       - __[entorno_agent]__: Orientado para todos los nodos que vienen como __consul_agent__
   <a name="yml"></a>
   
   ### 3.3 Ficheros YML
   - Aquí esta configurada la tarea de dar de Alta y Baja maquinas.
   - Toda esta tarea solo se realiza de forma local en la vlish4200, aqui se hace 
   un clone del o los repositorios y se actualiza los inventarios, antes de lanzar las
   tareas de los otros playbooks(__Telegraf__,__Filebeat__).
    
   

