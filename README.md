# Memoria del Proyecto de Despliegue de 'Merca-chain' en Hyperledger Fabric

### Introducción
Este documento detalla las operaciones realizadas y los problemas solucionados durante el despliegue de la red Hyperledger Fabric para la empresa Merca-link. Se describe cómo se configuró la red, se desplegaron los chaincodes, y se ajustaron las configuraciones de seguridad y monitoreo.

### Configuración Inicial y Clonación del Repositorio
El proyecto comenzó con la clonación del repositorio que contiene los scripts y configuraciones necesarios para el despliegue de la red.

Pasos realizados:

Clonación del repositorio desde GitHub.

```bash 
https://github.com/KeepCodingBlockchain-I/hf-certification-practica
```

Creación de un archivo .gitignore para excluir archivos innecesarios del control de versiones.

### Instalación de los binarios de Hyperledger Fabric
Se descargaron e instalaron los binarios de Hyperledger Fabric utilizando el script proporcionado, `install-fabric.sh`, con el siguiente comando:

```bash 
./install-fabric.sh b
```

Al intentar desplegar la red detectamos que se mostraba una alerta indicando que las verisones de los binarios locales y las imagenes de docker no eran las mismas y que esto podria traer problemas.

![OutofSync](img/hlf01.png)

Para solucionar dicho situación procedimos a borrar los binarios actuales, eliminar todas las imagenes docker, volvimos a descargar nuevamente los binarios. Para ello primero descargamos el script `install-fabric.sh` y le dimos al mismo capacidad de ser ejecutable utilizando los siguientes comandos:

```bash 
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
```

Para luego volver a ejecutar dicho script bash para traernos los binarios actualizados.

```bash 
./install-fabric.sh b
```

### Despliegue de la Red
El despliegue inicial de la red enfrentó varios problemas, primero la diferencia en la versiones entre los binarios locales e imagenes de docker que ya mencionamos, y luego varios de los script para poder levantar tanto la red como el chaincode no estaba habilitados para ejecutarse. Con lo cual se tuvo que dar a los script dicha capacidad con el siguiente comando:

```bash 
cd merca-chain
chmod +x network.sh
```

A lo largo de la actividad encontramos la misma situación con otros scripts los cuales listamos a continuación:

```bash 
chmod +x scripts/createChannel.sh 
chmod +x organizations/ccp-generate.sh
chmod +x scripts/setAnchorPeer.sh 
chmod +x scripts/deployCC.sh
chmod +x scripts/packageCC.sh
```

Finalmente para desplegar la red utilizamos el comando:

```bash 
./network.sh up createChannel -ca
```

De aqui la primera parte de la actividad era detectar y corregir la razón por la quie algunos de los contenedores no se levantan correctamente.

![peer0.org1](img/hlf02.png)

Lo primero que observamos fue este mensaje de error el cual indica la imposibilidad de unir el peer de la organización 1 al canal `mychannel`. con lo cual procedimos a revisar si los contenedores estaban funcionando adecuadamente. Para ello utilizamos el comando:

```bash 
docker ps -a
```

![docker ps](img/hlf03.png)

En esta imagen podemos ver que tanto el contendor del orderer como el del peer de la organización 1 no estan funcionando.

Para determinar que estaba ocurriendo procedimos a revisar los logs de ambos contenedores.

Primero el orderer:

```bash 
docker logs orderer.example.com
```

![docker logs 1](img/hlf04.png)

Aqui podemos ver que se muestra que no consigue el fichero en la ruta:

```bash
/var/hyperledger/orderer/tls/servre.crt
```

De aquí podemos observar que el nombre del fichero esta incorrectamente escrito. Para solucionarlo se modificó en el fichero compose-test-net.yaml la linea 42 (41 del fichero original) cambiando servre.crt por server.crt

En segundo lugar el Peer de la Organización 1:

```bash 
docker logs peer0.org1.example.com
```

![docker logs 2](img/hlf05.png)

En este caso podmeos determinar que el problema esta en un par de letras cambiadas en el ruta al fichero server.key. Para solucionarlo, se modificó en el fichero compose-test-net.yaml la linea 91 (89 del fichero original) cambiando _/hypreledger/fabric_ por _/hyperledger/fabric_.

>NOTA: la numeración corresponde con la actual del fichero, al cual se ha añandido lineas para las atender a las actividades posteriores. Coloco entre parentesis la linea correspondiente al fichero original.

### Despliegue del Chaincode "merca-chaincode"
Durante el proceso de despliegue del chaincode "merca-chaincode" en la red 'Merca-chain', se encontró que, aunque el ciclo de vida del chaincode completaba satisfactoriamente, el chaincode no operaba como se esperaba. Tras ejecutar el comando para levantar el chaincode:

```bash 
./network.sh deployCC -ccn merca-chaincode -ccl javascript -ccv 1.0.0 -ccp ../merca-chaincode
```

Se observó que los contenedores de los chaincodes se encontraban caídos. 

![chaincodes caidos](img/hlf06.png)

Una inspección inmediata de los registros del contenedor reveló un error crítico en el archivo assetTransfer.js, específicamente en la línea 92, donde se encontró un SyntaxError: Unexpected identifier. Esta es una indicación clara de un error de programación en el script del chaincode.

![error chaincode](img/hlf07.png)

Al revisar el archivo assetTransfer.js, se descubrió que el error se debía a una llave de cierre ausente en el método `CreateAsset`.

Finalmente detuvimos y volvimos a levantar tanto la red como el chaincode con los comandos antes utilizados. Con estos pasos, el chaincode "merca-chaincode" se desplegó correctamente, y todos los contenedores operaron de manera estable.

### Ajustes de Logging en la Red 'Merca-chain' de Hyperledger Fabric
La configuración eficiente del registro (logging) es vital para el monitoreo, la depuración y la auditoría en cualquier red de blockchain empresarial. Merca-link ha expresado la necesidad de mejorar la calidad y la estructura de los registros de la red 'Merca-chain', especificando requisitos precisos para los registros de los orderers y los peers. A continuación, se detallan los ajustes implementados en el archivo `compose-test-net.yaml`, tal como se refleja en las imágenes adjuntas, y cómo estos cumplen con las necesidades establecidas por Merca-link.

#### Cambios en el Logging de los Orderers:
De acuerdo a la actividad estos registros deben estar en formato JSON y lo mismo se cumplió modificando la configuración de entorno del servicio de orderer en el archivo `compose-test-net.yaml`.

![orderer](img/hlf08.png)

#### Cambios en el Logging de los Peers:
Para los peers, la configuración de registros requiere una salida detallada que incluya la fecha y la hora al final de cada entrada de registro, además de un nivel de registro ajustado. Los cambios realizados se describen a continuación:

- Se ha configurado el formato de registro para incluir la fecha y hora pero al final de cada registro, es decir, posterior al mensaje que refleja el mismo.
- Para la fase de pruebas, se ha establecido un nivel de registro por defecto de DEBUG para ambos peers, asi como para los loggers específicos de gossip y chaincode, se han establecido niveles de WARNING e INFO, respectivamente.

![peer0,org1](img/hlf09.png)
![peer0,org2](img/hlf10.png)

### Integración de Hardware Security Module (HSM) con Fabric CA en Merca-chain
Como parte de los esfuerzos para mejorar la seguridad en la red 'Merca-chain', se implementó un Hardware Security Module (HSM) simulado utilizando __SoftHSM2__, el cual posee soporte para el protocolo __PKCS#11__. Este paso es fundamental para aumentar la protección de las claves criptográficas usadas dentro de la red. En el proceso de integrar HSM con las CAs de Hyperledger Fabric, se realizaron ajustes específicos en los archivos de configuración y se añadieron archivos especializados para cada organización y los orderers.

#### Procedimiento de Integración
##### Reconstrucción de la Imagen de Fabric CA:
Dado que las imágenes predeterminadas de Fabric CA no incluyen soporte para __PKCS#11__, fue necesario reconstruir las imágenes para agregar esta funcionalidad. Los pasos seguidos para realizar esta tarea fueron:

Clonar el repositorio de Fabric CA:

```bash 
git clone https://github.com/hyperledger/fabric-ca.git
cd fabric-ca
```

Reconstruir las imágenes con soporte __PKCS#11__ ejecutando desde la raíz de fabric-ca, especificando la versión de Ubuntu adecuada para poder tener compatibilidad con los tokens que generaremos en el host, asi como el soporte al protocolo __PKCS#11__.

```bash 
make docker UBUNTU_VER=22.04 GO_TAGS=pkcs11
```

##### Configuración de SoftHSM2
Se ejecutaron los siguientes comandos para inicializar tokens para cada CA:

```bash 
export SOFTHSM2_CONF=/etc/softhsm2.conf

sudo softhsm2-util --init-token --slot 0 --label ca_orderer --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 1 --label ca_org1 --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 2 --label ca_org2 --so-pin 1234 --pin 1234
```

Estos comandos establecen un token para cada CA que Hyperledger Fabric utilizará, con etiquetas y pines específicos para cada una, mejorando así la seguridad de las claves criptográficas.

##### Configuración de la Red
Se realizaron los ajustes necesarios en el archivo `compose-ca.yaml` para integrar el HSM fueron:

- Establecer __FABRIC_CA_HOME__ para definir el directorio donde Fabric CA buscará su configuración y almacenará materiales criptográficos.
- Utilizar __SOFTHSM2_CONF__ para indicar la ubicación del archivo de configuración del HSM en el contenedor.
- En la sección __volumes__, montar el directorio HOME de la CA a la carpeta local correspondiente a cada organización o orderer, permitiendo acceso externo a los archivos generados por la CA.

![directorio](img/hlf11.png)

- Además, como parte de la configuración segura de HSM, se añadieron archivos específicos `fabric-ca-server-config.yaml` y `softhsm2.conf` en la estructura de directorios de cada organización y orderer dentro de la red. Estos archivos se personalizaron para cada entidad para reflejar las necesidades y políticas de seguridad individuales de cada uno.
- Montar la carpeta conteniendo los tokens del HSM, asegurándose de que las rutas coincidan con las especificadas en `softhsm2.conf`.
- Montar la librería de __SoftHSM2__ dentro del contenedor.

### Configuración y Uso del Fabric-CA-Client con SoftHSM
Para utilizar esta configuración segura, fue necesario reconstruir el binario local de `fabric-ca-client` para incluir soporte PKCS#11. Una vez reconstruido, se procedió a registrar un usuario administrativo, denominado merca-admin, utilizando el cliente CA actualizado.

#### Procedimiento de Configuración
##### Reconstrucción del Binario:
Clonación y acceso al repositorio de `fabric-ca`:

```bash 
git clone https://github.com/hyperledger/fabric-ca.git
cd fabric-ca
```

Reconstrucción del binario del `fabric-ca-client` con soporte para __PKCS#11__:

```bash 
make fabric-ca-client GO_TAGS=pkcs11
```

Tras reconstruir el binario, se localizó el nuevo `fabric-ca-client` en la carpeta bin del directorio `fabric-ca` y se movio a la carpeta de binarios de la raíz de el repositorio de la actividad.

#### Registro de Usuario con Fabric-CA-Client y SoftHSM
##### Inscripción y Registro de Usuario
Utilizando el binario reconstruido, se inscribió el usuario `admin` y luego se registró el usuario `merca-admin` con la capacidad de registrar nuevos clientes y peers. Los comandos utilizados fueron:

Se configuró el entorno para el `fabric-ca-client` especificando la ubicación del directorio de trabajo para la organización:

```bash 
export FABRIC_CA_CLIENT_HOME=/home/ubuntu/merca-chain_hlf_deployment/merca-chain/organizations/peerOrganizations/org1.example.com/
```

Ingresar al servidor de la CA y enrolar el administrador con el siguiente comando:

```bash 
sudo ../bin/fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"
```

![enroll](img/hlf12.png)

Registrar al usuario `merca-admin` con las credenciales y atributos correspondientes:

```bash 
sudo ../bin/fabric-ca-client register -d -u https://localhost:7054 --caname ca-org1 --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"
```

Este procedimiento completó con éxito el registro del usuario administrativo `merca-admin` en la red 'Merca-chain' como se puede apreciar en la imagen a continuación:

![merca-admin](img/hlf13.png)

En los comandos mostrados anteriormente se incluyeron elementos como la __url__ y la __caname__ para evitar el uso del fichero de configuración `fabric-ca-client-config.yaml`.

### Implementación de Herramientas de Monitorización en Merca-chain
#### Despliegue de Prometheus, Grafana, Node Exporter y cAdvisor
Se procedió a configurar y desplegar instancias de Prometheus, Grafana, Node Exporter y cAdvisor utilizando el fichero `docker-compose.yml` provisto por el repostorio, utilizando el siguiente comando:

```bash 
docker compose up -d
```

Este despliegue facilitaría la monitorización de la red y permitiría al equipo técnico anticipar y resolver problemas con mayor eficiencia, así como optimizar el rendimiento del sistema. 

##### Problema Inicial con cAdvisor
Como parte de la solución de monitorización, se decidió emplear cAdvisor para obtener métricas de los contenedores Docker. Sin embargo, al utilizar la imagen original `google/cadvisor:latest` especificada en el docker-compose.yml, se encontró un problema significativo: el contenedor de cAdvisor no se levantaba:

![nocAdvisor](img/hlf14.png)

presentando el siguiente error en el inicio:

![cAdvisorerror](img/hlf15.png)

Este error impedía que cAdvisor se ejecutara correctamente, lo que resultaba en la falta de métricas de los contenedores y, por lo tanto, un vacío en la monitorización integral de la red.

Para resolver este problema, se realizó una evaluación del error y se identificó que la solución radicaba en la actualización de la imagen de cAdvisor. Se procedió de la siguiente manera:

1. Se detuvieron todos los contenedores del ecosistema de Prometheus y Grafana para asegurarse de que no hubiera procesos de cAdvisor ejecutándose que pudieran afectar la actualización, utilizando el comando:  

```bash 
docker compose down
```

2. Se modificó el archivo `docker-compose.yml`, reemplazando la imagen de cAdvisor original
![google/cadvisor:latest](img/hlf16.png)
a 
![gcr.io/cadvisor/cadvisor:latest](img/hlf17.png)

3. Se volvió a levantar el ecosistema completo nuevamente con el comando

```bash 
docker compose up -d
```

Este cambio fue exitoso, y los contenedores se iniciaron sin errores.

![noproblem](img/hlf18.png)

##### Verificación y Monitorización
Con cAdvisor ahora funcionando correctamente, se continua con la implementación exitosa de las herramientas de monitorización Prometheus, Grafana y Node Exporter.

![noproblem](img/hlf19.png)

![noproblem](img/hlf20.png)

Los servicios pertinentes se levantaron y están ahora accesibles en la siguiente configuración de puertos dentro de la máquina virtual de Merca-link:

- Grafana: Disponible en el puerto 3000, lo que permite a los usuarios acceder a los dashboards interactivos y configurar alertas personalizadas para el seguimiento en tiempo real de la red.


```bash 
http://<IP_Maquina_Virtual>:3000
```

- Prometheus: Accesible en el puerto 9090, donde los usuarios pueden consultar y visualizar métricas recopiladas de toda la infraestructura de la red blockchain.

```bash 
http://<IP_Maquina_Virtual>:9090
```

- Node Exporter: Corriendo en el puerto 9100, proporciona métricas detalladas del sistema de la máquina virtual, lo que es esencial para la supervisión y análisis de rendimiento del hardware subyacente.

```bash 
http://<IP_Maquina_Virtual>:9100
```

- cAdvisor: Disponible en el puerto 8080, ofreciendo una visión detallada y métricas específicas sobre el uso de recursos y rendimiento de los contenedores Docker.

```bash 
http://<IP_Maquina_Virtual>:8080
```

Utilizando los dashboards proporcionados por la actividad, los merca-ingenieros de Merca-link comenzaron a recibir métricas en tiempo real, lo que les permitió tener una visión completa del estado y el rendimiento de la red 'Merca-chain'.

![dashboard1](img/hlf21.png)

![dashboard2](img/hlf22.png)
