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
Se descargaron e instalaron los binarios de Hyperledger Fabric utilizando el script proporcionado, `install-fabric.sh`, con el siguinete comando.

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

Aqui podemos ver que se muestra que no consigue el fichero en la ruta
```bash
/var/hyperledger/orderer/tls/servre.crt
```
De aquí podemos observar que el nombre del fichero esta incorrectamente escrito. Para solucionarlo se modificó en el fichero compose-test-net.yaml la linea 42 (41 del fichero original) cambiando servre.crt por server.crt

En segundo lugar el Peer de la Organización 1:
```bash 
docker logs peer0.org1.example.com
```
![docker logs 2](img/hlf05.png)

En este caso podmeos determinar que el problema esta en un par de letras cambiadas en el ruta al fichero server.bey. Para solucionarlo, se modificó en el fichero compose-test-net.yaml la linea 91 (89 del fichero original) cambiando /hypreledger/fabric por /hyperledger/fabric.

> NOTA: la numeración corresponde con la actual del fichero, al cual se ha añandido lineas para las atender a las actividades posteriores. Coloco entre parentesis la linea correspondiente al fichero original.

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
![orderer](img/hlf08.png)

-----------------------------------------------------------
apartado 4
export SOFTHSM2_CONF=/etc/softhsm2.conf

sudo softhsm2-util --init-token --slot 0 --label ca_orderer --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 1 --label ca_org1 --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 2 --label ca_org2 --so-pin 1234 --pin 1234

git clone https://github.com/hyperledger/fabric-ca.git

make docker UBUNTU_VER=22.04 GO_TAGS=pkcs11

------------------------------------------------------------
apartado 5

export FABRIC_CA_CLIENT_HOME=/home/ubuntu/merca-chain_hlf_deployment/merca-chain/organizations/peerOrganizations/org1.example.com/

sudo ../bin/fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"

sudo ../bin/fabric-ca-client register -d -u https://localhost:7054 --caname ca-org1 --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"