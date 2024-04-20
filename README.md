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
De aquí podemos observar 
se modificó en el fichero compose-test-net.yaml la linea 42 (41 del fichero original) cambiando servre.crt por server.crt


```bash 
docker logs peer0.org1.example.com
```
![docker logs 2](img/hlf05.png)


se modificó en el fichero compose-test-net.yaml la linea 91 (89 del fichero  original) cambiando /hypreledger/fabric por /hypelredger/fabric

NOTA: la numeración corresponde con la actual del fichero, al cual se ha añandido lineas para las atender a las actividades posteriores. Coloco entre parentesis la linea correpsondiente al fichero original.

----------------------------------------------------------

./network.sh deployCC -ccn merca-chaincode -ccl javascript -ccv 1.0.0 -ccp ../merca-chaincode

la dos cerrar la llave en la linea 89 del chaincode (assetTransfer.js)

-----------------------------------------------------------

export SOFTHSM2_CONF=/etc/softhsm2.conf

sudo softhsm2-util --init-token --slot 0 --label ca_orderer --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 1 --label ca_org1 --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 2 --label ca_org2 --so-pin 1234 --pin 1234

git clone https://github.com/hyperledger/fabric-ca.git

make docker UBUNTU_VER=22.04 GO_TAGS=pkcs11

------------------------------------------------------------

export FABRIC_CA_CLIENT_HOME=/home/ubuntu/merca-chain_hlf_deployment/merca-chain/organizations/peerOrganizations/org1.example.com/

sudo ../bin/fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"

sudo ../bin/fabric-ca-client register -d -u https://localhost:7054 --caname ca-org1 --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"