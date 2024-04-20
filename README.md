# merca-chain_hlf_deployment
This repository hosts scripts, deployment configurations, and troubleshooting guides for the 'Merca-chain' Hyperledger Fabric network 

1. clonar el repositorio de la práctica
1. añadir el fichero .gitignore para evistar que los binarios y ficheros muy grandes no se suba al repo
1. descargar el script install-fabric.sh y dar al mismo capacidad de ser ejecutable utilizando: 
```bash 
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
```
1. ejecutar el script bash para tarernos los binarios 
```bash 
./install-fabric.sh b
```
1. 


Despliega la red. Parece que existen algunos contenedores que no se levantan correctamente. Identifícalos y resuelve los problemas asociados.
```bash 
cd merca-chain
chmod +x network.sh
chmod +x scripts/createChannel.sh 
./network.sh up createChannel -ca
```
----------------------------------------------------------------------------------
Status: Downloaded newer image for hyperledger/fabric-tools:latest
LOCAL_VERSION=v2.5.6
DOCKER_IMAGE_VERSION=v2.5.7
Local fabric binaries and docker images are out of  sync. This may cause problems.

Status: Downloaded newer image for hyperledger/fabric-ca:latest
CA_LOCAL_VERSION=v1.5.9
CA_DOCKER_IMAGE_VERSION=v1.5.10
Local fabric-ca binaries and docker images are out of sync. This may cause problems.

Solución borrar los binarios y volver  a descargar el install-fabric.sh y volver a instalar los binarios
-------------------------------------------------------

WARNING: Found orphan containers (ca_org1, ca_org2, ca_orderer) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
---------------------------------------------------------------------------------------------

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=1
Error: error getting endorser client for channel: endorser client failed to connect to localhost:7051: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp 127.0.0.1:7051: connect: connection refused"
After 5 attempts, peer0.org1 has failed to join channel 'mychannel' 
-----------------------------------------------------------------------
ubuntu@ip-10-3-104-56:~/merca-chain_hlf_deployment/merca-chain$ docker ps -a
CONTAINER ID   IMAGE                               COMMAND                  CREATED              STATUS                          PORTS                                                                                                NAMES
0c25ebde76c9   hyperledger/fabric-tools:latest     "/bin/bash"              About a minute ago   Up About a minute                                                                                                                    cli
c3dda06916eb   hyperledger/fabric-orderer:latest   "orderer"                About a minute ago   Exited (2) About a minute ago                                                                                                        orderer.example.com
a76ea31062cb   hyperledger/fabric-peer:latest      "peer node start"        About a minute ago   Exited (1) About a minute ago                                                                                                        peer0.org1.example.com
3fa87897c291   hyperledger/fabric-peer:latest      "peer node start"        About a minute ago   Up About a minute               0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp       peer0.org2.example.com
c88ee1e0795d   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   About a minute ago   Up About a minute               0.0.0.0:7054->7054/tcp, :::7054->7054/tcp, 0.0.0.0:17054->17054/tcp, :::17054->17054/tcp             ca_org1
61efdc1cf1cf   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   About a minute ago   Up About a minute               0.0.0.0:8054->8054/tcp, :::8054->8054/tcp, 7054/tcp, 0.0.0.0:18054->18054/tcp, :::18054->18054/tcp   ca_org2
a92f5c5057b9   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   About a minute ago   Up About a minute               0.0.0.0:9054->9054/tcp, :::9054->9054/tcp, 7054/tcp, 0.0.0.0:19054->19054/tcp, :::19054->19054/tcp   ca_orderer
-------------------------------------------------------------------------------------

docker logs -f orderer.example.com
2024-04-18 08:43:05.218 UTC 000e PANI [orderer.common.server] Main -> failed to start admin server: open /var/hyperledger/orderer/tls/servre.crt: no such file or directory
panic: failed to start admin server: open /var/hyperledger/orderer/tls/servre.crt: no such file or directory

goroutine 1 [running]:
go.uber.org/zap/zapcore.CheckWriteAction.OnWrite(0x0?, 0xc00044afc8?, {0x0?, 0x0?, 0xc00048eee0?})
        /vendor/go.uber.org/zap/zapcore/entry.go:196 +0x54
go.uber.org/zap/zapcore.(*CheckedEntry).Write(0xc000272c30, {0x0, 0x0, 0x0})
        /vendor/go.uber.org/zap/zapcore/entry.go:262 +0x3ec
go.uber.org/zap.(*SugaredLogger).log(0xc0000676b0, 0x4, {0x11b9038?, 0x27?}, {0xc00044b5a8?, 0x1?, 0x1?}, {0x0, 0x0, 0x0})
        /vendor/go.uber.org/zap/sugar.go:316 +0xec
go.uber.org/zap.(*SugaredLogger).Panicf(...)
        /vendor/go.uber.org/zap/sugar.go:202
github.com/hyperledger/fabric/common/flogging.(*FabricLogger).Panicf(...)
        /common/flogging/zap.go:74
github.com/hyperledger/fabric/orderer/common/server.Main()
        /orderer/common/server/main.go:267 +0x171b
main.main()
        /cmd/orderer/main.go:15 +0xf

cambié en el fichero compose-test-net.yaml la linea 33 cambiando servre.crt por server.crt
------------------------------------------------------------------------------------------
docker logs -f peer0.org1.example.com
2024-04-18 08:43:05.241 UTC 0005 FATA [nodeCmd] serve -> Error loading secure config for peer (error loading TLS key (open /etc/hyperledger/fabric/tls/server.key: no such file or directory))

cambié en el fichero compose-test-net.yaml la linea 33 cambiando servre.crt por server.crt
----------------------------------------------------------
chmod +x organizations/ccp-generate.sh
chmod +x scripts/setAnchorPeer.sh 
chmod +x scripts/deployCC.sh
chmod +x scripts/packageCC.sh


la dos cerrar la llave en la linea 89 del chaincode (assetTransfer.js)
--------------------------------------------------------
# export SOFTHSM2_CONF=/home/ubuntu/merca-chain_hlf_deployment/hsm-config/softhsm2.conf

sudo softhsm2-util --init-token --slot 0 --label ca_orderer --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 1 --label ca_org1 --so-pin 1234 --pin 1234
sudo softhsm2-util --init-token --slot 2 --label ca_org2 --so-pin 1234 --pin 1234

git clone https://github.com/hyperledger/fabric-ca.git

make docker UBUNTU_VER=22.04 GO_TAGS=pkcs11

<!-- 

fabric-ca-client register -d --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1


export FABRIC_CA_CLIENT_HOME=/home/ubuntu/merca-chain_hlf_deployment/merca-chain/hsm-config -->
export SOFTHSM2_CONF=/etc/softhsm2.conf

./network.sh deployCC -ccn merca-chaincode -ccl javascript -ccv 1.0.0 -ccp ../merca-chaincode

sudo ../bin/fabric-ca-client enroll -u https://admin:adminpw@localhost:7054

sudo ../bin/fabric-ca-client register -d --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1


export FABRIC_CA_CLIENT_HOME=/home/ubuntu/merca-chain_hlf_deployment/merca-chain/organizations/peerOrganizations/org1.example.com/

sudo ../bin/fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"

sudo ../bin/fabric-ca-client register -d -u https://localhost:7054 --caname ca-org1 --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"

sudo ../bin/fabric-ca-client register -d -u https://localhost:7054 --caname ca-org1 --id.name merca-admin --id.secret merka-12345 --id.type client --id.attrs '"hf.Registrar.Roles=peer,client"' --id.affiliation org1.department1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/ca-cert.pem"