# Hyperledger-Fabric-Prod-Network

A simple HyperLedger Fabric Production sample network.

CA Deployment setup instruction steps.

1. `mkdir Hyperledger-Fabric-Prod-Network`

2. Download the latest CA binaries from https://github.com/hyperledger/fabric-ca/releases and extract it into the folder created in step 1.

3. Setup Fabric CA client:

   `cd Hyperledger-Fabric-Prod-Network`

   `mkdir fabric-ca-client`

   `cd fabric-ca-client`
   
   `mkdir tls-ca org1-ca tls-root-cert`

   Copy the fabric-ca-client binary into the fabric-ca-client folder.

4. Deploy TLS CA:

   `cd ..`

   `mkdir fabric-ca-server-tls`

   copy the fabric-ca-server binary into fabric-ca-server-tls folder.

   `cd fabric-ca-server-tls`

   `./fabric-ca-server init -b tls-admin:tls-adminpw`

5. Open _fabric-ca-server-tls/fabric-ca-server-config.yaml_ and modify it:

   1. change tls.enabled: false to true
   2. assign ca.name: tls-ca
   3. edit csr.hosts if needed
   4. edit csr.ca.expiry: 131400h to 876000h
   5. remove signing.profiles.ca block and leave only tls block under signing.profiles.
   6. edit signing.profiles.tls.expiry: 8760h to 876000h
   7. edit signing.default.expiry: 8760h to 876000h
   8. save

6. delete _fabric-ca-server-tls/ca-cert.pem_ file & entire _fabric-ca-server-tls/msp_ folder, then run `./fabric-ca-server start`

7. copy _fabric-ca-server-tls/ca-cert.pem_ to _fabric-ca-client/tls-root-cert/tls-ca-cert.pem_ (file name changed for clarity)

8. open new terminal tab:

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

   `./fabric-ca-client enroll -d -u https://tls-admin:tls-adminpw@localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --enrollment.profile tls --csr.hosts 'kky-dell,*localhost' --mspdir tls-ca/tlsadmin/msp`

   `./fabric-ca-client register -d --id.name rcaadmin --id.secret rcaadminpw -u https://localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --mspdir tls-ca/tlsadmin/msp`

   `./fabric-ca-client enroll -d -u https://rcaadmin:rcaadminpw@localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --enrollment.profile tls --csr.hosts 'kky-dell,*localhost' --mspdir tls-ca/rcaadmin/msp`

   rename the file in _fabric-ca-client/tls-ca/rcaadmin/msp/keystore_ folder to _key.pem_

9. Deploy orgnization CA:

   `cd ..`

   `mkdir fabric-ca-server-org1`

   copy the fabric-ca-server binary in _fabric-ca-server-org1_ folder

   `cd fabric-ca-server-org1`

   `mkdir tls`

   `cp ../fabric-ca-client/tls-ca/rcaadmin/msp/signcerts/cert.pem tls && cp ../fabric-ca-client/tls-ca/rcaadmin/msp/keystore/key.pem tls`

   `./fabric-ca-server init -b rcaadmin:rcaadminpw`

10. Open _fabric-ca-server-org1/fabric-ca-server-config.yaml_ and modify it:

    1. change port from 7054 to 7055
    2. change tls.enabled: false to true
    3. assign tls.certfile: tls/cert.pem
    4. assign tls.keyfile: tls/key.pem
    5. assign ca.name: org1-ca
    6. edit csr.hosts if needed
    7. operations.listenAddress: 127.0.0.1:9444
    8. edit signing.default.expiry: 8760h to 876000h
    9. edit signing.profiles.ca.expiry: 43800h to 876000h
    10. edit signing.profiles.tls.expiry: 8760h to 876000h
    11. edit csr.ca.expiry: 131400h to 876000h
    12. save

11. delete _fabric-ca-server-org1/ca-cert.pem_ file & entire _fabric-ca-server-org1/msp_ folder, then run `./fabric-ca-server start`

12. open new terminal tab:

    `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

    `export FABRIC_CA_CLIENT_HOME=$PWD`

    `./fabric-ca-client enroll -d -u https://rcaadmin:rcaadminpw@kky-dell:7055 --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost' --mspdir org1-ca/rcaadmin/msp`

    rename the file in _fabric-ca-client/org1-ca/rcaadmin/msp/keystore_ folder to _org1-key.pem_

#############################################################################

Registering and enrolling org1 admin identity instruction steps.

1. Start the fabric-ca-server-org1 as stated in step 11 above, then open a new terminal tab:

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register an identity

   `./fabric-ca-client register -d --id.name org1admin --id.secret org1adminpw -u https://kky-dell:7055 --mspdir ./org1-ca/rcaadmin/msp --id.type admin --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll an identity

   `./fabric-ca-client enroll -u https://org1admin:org1adminpw@kky-dell:7055 --mspdir ./org1.example.com/org1admin/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _org1.example.com/org1admin/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _org1.example.com/org1admin/msp/signcerts/cert.pem_ to _org1-admin-cert.pem_

   ii) rename file inside _org1.example.com/org1admin/msp/keystore_ to _org1-admin-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir org1.example.com/org1admin/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem org1.example.com/org1admin/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p org1.example.com/org1admin/orgMsp/msp org1.example.com/org1admin/localMsp/msp`

8. Prepare the orgMsp files

   `cp org1.example.com/org1admin/msp/config.yaml org1.example.com/org1admin/orgMsp/msp && cp -R org1.example.com/org1admin/msp/cacerts org1.example.com/org1admin/orgMsp/msp && cp -R org1.example.com/org1admin/msp/tlscacerts org1.example.com/org1admin/orgMsp/msp`

9. Prepare the localMsp files

   `cp org1.example.com/org1admin/msp/config.yaml org1.example.com/org1admin/localMsp/msp && cp -R org1.example.com/org1admin/msp/cacerts org1.example.com/org1admin/localMsp/msp && cp -R org1.example.com/org1admin/msp/tlscacerts org1.example.com/org1admin/localMsp/msp && cp -R org1.example.com/org1admin/msp/keystore org1.example.com/org1admin/localMsp/msp && cp -R org1.example.com/org1admin/msp/signcerts org1.example.com/org1admin/localMsp/msp`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Registering and enrolling peer identity.

1. Start the org CA

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-server-org1`

   `./fabric-ca-server start`

   Open new terminal tab

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register peer identity

   `./fabric-ca-client register -d --id.name org1peer0 --id.secret org1peer0pw -u https://kky-dell:7055 --mspdir ./org1-ca/rcaadmin/msp --id.type peer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll peer identity

   `./fabric-ca-client enroll -u https://org1peer0:org1peer0pw@kky-dell:7055 --mspdir ./org1.example.com/org1peer0/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _org1.example.com/org1admin/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _org1.example.com/org1peer0/msp/signcerts/cert.pem_ to _org1-peer0-cert.pem_

   ii) rename file inside _org1.example.com/org1peer0/msp/keystore_ to _org1-peer0-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir org1.example.com/org1peer0/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem org1.example.com/org1peer0/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p org1.example.com/org1peer0/orgMsp/msp org1.example.com/org1peer0/localMsp/msp`

8. Prepare the orgMsp files

   `cp org1.example.com/org1peer0/msp/config.yaml org1.example.com/org1peer0/orgMsp/msp && cp -R org1.example.com/org1peer0/msp/cacerts org1.example.com/org1peer0/orgMsp/msp && cp -R org1.example.com/org1peer0/msp/tlscacerts org1.example.com/org1peer0/orgMsp/msp`

9. Prepare the localMsp files

   `cp org1.example.com/org1peer0/msp/config.yaml org1.example.com/org1peer0/localMsp/msp && cp -R org1.example.com/org1peer0/msp/cacerts org1.example.com/org1peer0/localMsp/msp && cp -R org1.example.com/org1peer0/msp/tlscacerts org1.example.com/org1peer0/localMsp/msp && cp -R org1.example.com/org1peer0/msp/keystore org1.example.com/org1peer0/localMsp/msp && cp -R org1.example.com/org1peer0/msp/signcerts org1.example.com/org1peer0/localMsp/msp`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Deploy the peer

1. Get binary and config files.

   Download the peer binaries and configuration files from https://github.com/hyperledger/fabric/releases and extract into _Hyperledger-Fabric-Prod-Network_ and rename the file as _fabric_.

2. Create the following folder structure

   ```
   ├── organizations
        └── peerOrganizations
            └── org1.example.com
                ├── msp
                └── peers
                    └── peer0.org1.example.com
                        ├── msp
                        └── tls
                        └── storage
                        	└── snapshots
   ```
   `cd Hyperledger-Fabric-Prod-Network`

   `mkdir -p organizations/peerOrganizations/org1.example.com/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/storage/snapshots`

3. Copy the peer binary and core.yaml file from _fabric_ into _organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com_  folder. Then add the location of the peer binary to PATH environment variable

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric/bin/peer organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com && cp -R fabric/config/core.yaml organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com`

   `cd organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com`

   `export PATH=$PWD:$PATH`

4. Prepare i) TLS certificates, ii) peer local MSP files, and iii) peer org MSP files.

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric-ca-client/org1.example.com/org1peer0/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tls-cert.pem`

   `cp -R fabric-ca-client/org1.example.com/org1peer0/localMsp/msp/signcerts/org1-peer0-cert.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls`

   `cp -R fabric-ca-client/org1.example.com/org1peer0/localMsp/msp/keystore/org1-peer0-key.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls`

   `cp -R fabric-ca-client/org1.example.com/org1peer0/localMsp/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com`

   `cp -R fabric-ca-client/org1.example.com/org1peer0/orgMsp/msp organizations/peerOrganizations/org1.example.com`

5. Edit _core.yaml_ in _organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com_ with the respective values:

   ```
   peer.id: peer0.org1.example.com

   peer.networkId: prod

   peer.listenAddress: 192.168.0.146:7051

   peer.chaincodeListenAddress: 192.168.0.146:7052

   peer.address: 192.168.0.146:7051

   gossip.bootstrap: 192.168.0.146:7051

   gossip.endpoint: 192.168.0.146:7051

   gossip.externalEndpoint: 118.100.204.232:7051 (Adjust according to your public IP address with portforwarding)

   peer.tls.enabled: true

   peer.tls.rootcert.file: tls/tls-cert.pem

   peer.tls.cert.file: tls/org1-peer0-cert.pem

   peer.tls.key.file: tls/org1-peer0-key.pem

   peer.mspConfigPath: msp

   peer.fileSystemPath: storage

   peer.localMspId: Org1MSP

   ledger.snapshots.rootDir: /storage/snapshots

   operations.listenAddress: 127.0.0.1:9545
   ```

6. Start the peer

   `cd Hyperledger-Fabric-Prod-Network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com`

   `export FABRIC_CFG_PATH=$PWD`

   `sudo ./peer node start`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

2. Output from peer states there are No active channels passed. (which is understandable because we have not joined our peer to any channel yet, but hope it is not an error for the next steps ahead.)

#############################################################################

Setup Org2 CA Server, then register and enroll org2 admin identity instruction steps.

1.  Deploy the organization 2 CA

    `cd Hyperledger-Fabric-Prod-Network`

    `mkdir -p fabric-ca-server-org2/tls`

    `cp hyperledger-fabric-ca-linux-amd64-1.5.3/bin/fabric-ca-server fabric-ca-server-org2`

    `cp fabric-ca-client/tls-ca/rcaadmin/msp/signcerts/cert.pem fabric-ca-server-org2/tls && cp fabric-ca-client/tls-ca/rcaadmin/msp/keystore/key.pem fabric-ca-server-org2/tls`

    `cd fabric-ca-server-org2`

    `./fabric-ca-server init -b Org2rcaadmin:Org2rcaadminpw`

2.  Edit _fabric-ca-server-config.yaml_ under _fabric-ca-server-org2_ folder with the following values:

    ```
    port: 7056
    tls.enabled: true
    tls.certfile: tls/cert.pem
    tls.keyfile: tls/key.pem
    ca.name: org2-ca
    operations.listenAddress: 127.0.0.1:9445

    csr:
       cn: fabric-ca-server
       keyrequest:
          algo: ecdsa
          size: 256
       names:
        - C: Malaysia
          ST: Kuala Lumpur
          L:
          O: Hyperledger
          OU: Fabric
       hosts:
          - kky-dell
          - localhost
       ca:
          expiry: 876000h
          pathlength: 1

    signing:
        default:
            usage:
                - digital signature
            expiry: 876000h
        profiles:
            ca:
                usage:
                    - cert sign
                    - crl sign
                expiry: 876000h
                caconstraint:
                    isca: true
                    maxpathlen: 0
            tls:
                usage:
                    - signing
                    - key encipherment
                    - server auth
                    - client auth
                    - key agreement
                expiry: 876000h
    ```

3.  Delete _fabric-ca-server-org2/ca-cert.pem_ and entire _fabric-ca-server-org2/msp_ folder, and then start the CA.

    `rm ca-cert.pem`

    `rm -r msp`

    `./fabric-ca-server start`

4.  Open new terminal tab:

    `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

    `export FABRIC_CA_CLIENT_HOME=$PWD`

    `./fabric-ca-client enroll -d -u https://Org2rcaadmin:Org2rcaadminpw@kky-dell:7056 --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost' --mspdir org2-ca/rcaadmin/msp`

    rename the file in _fabric-ca-client/org2-ca/rcaadmin/msp/keystore_ folder to _org2-key.pem_

5.  Register and enroll org2 admin identity

    `./fabric-ca-client register -d --id.name org2admin --id.secret org2adminpw -u https://kky-dell:7056 --mspdir ./org2-ca/rcaadmin/msp --id.type admin --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

    `./fabric-ca-client enroll -u https://org2admin:org2adminpw@kky-dell:7056 --mspdir ./org2.example.com/org2admin/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

6.  create config.yaml file inside _org2.example.com/org2admin/msp_ folder with values of:

    ```
    NodeOUs:
        Enable: true
        ClientOUIdentifier:
            Certificate: cacerts/kky-dell-7056.pem
            OrganizationalUnitIdentifier: client
        PeerOUIdentifier:
            Certificate: cacerts/kky-dell-7056.pem
            OrganizationalUnitIdentifier: peer
        AdminOUIdentifier:
            Certificate: cacerts/kky-dell-7056.pem
            OrganizationalUnitIdentifier: admin
        OrdererOUIdentifier:
            Certificate: cacerts/kky-dell-7056.pem
            OrganizationalUnitIdentifier: orderer
    ```

7.  i) rename _org2.example.com/org2admin/msp/signcerts/cert.pem_ to _org2-admin-cert.pem_

    ii) rename file inside _org2.example.com/org2admin/msp/keystore_ to _org2-admin-key.pem_

8.  Create a tlscacerts folder and copy the tls-root-cert into this folder.

    `mkdir org2.example.com/org2admin/msp/tlscacerts`

    `cp tls-root-cert/tls-ca-cert.pem org2.example.com/org2admin/msp/tlscacerts`

9.  Create orgMsp and localMsp folder.

    `mkdir -p org2.example.com//org2admin/orgMsp/msp org2.example.com/org2admin/localMsp/msp`

10. Prepare the orgMsp files

    `cp org2.example.com/org2admin/msp/config.yaml org2.example.com/org2admin/orgMsp/msp && cp -R org2.example.com/org2admin/msp/cacerts org2.example.com/org2admin/orgMsp/msp && cp -R org2.example.com/org2admin/msp/tlscacerts org2.example.com/org2admin/orgMsp/msp`

11. Prepare the localMsp files

    `cp org2.example.com/org2admin/msp/config.yaml org2.example.com/org2admin/localMsp/msp && cp -R org2.example.com/org2admin/msp/cacerts org2.example.com/org2admin/localMsp/msp && cp -R org2.example.com/org2admin/msp/tlscacerts org2.example.com/org2admin/localMsp/msp && cp -R org2.example.com/org2admin/msp/keystore org2.example.com/org2admin/localMsp/msp && cp -R org2.example.com/org2admin/msp/signcerts org2.example.com/org2admin/localMsp/msp`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

 Register and enroll org2peer0, then start org2peer0.

1. Start the org2 CA

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-server-org2`

   `./fabric-ca-server start`

   Open new terminal tab

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register peer identity

   `./fabric-ca-client register -d --id.name org2peer0 --id.secret org2peer0pw -u https://kky-dell:7056 --mspdir ./org2-ca/rcaadmin/msp --id.type peer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll peer identity

   `./fabric-ca-client enroll -u https://org2peer0:org2peer0pw@kky-dell:7056 --mspdir ./org2.example.com/org2peer0/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _org2.example.com/org2peer0/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7056.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7056.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7056.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7056.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _org2.example.com/org2peer0/msp/signcerts/cert.pem_ to _org2-peer0-cert.pem_

   ii) rename file inside _org2.example.com/org2peer0/msp/keystore_ to _org2-peer0-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir org2.example.com/org2peer0/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem org2.example.com/org2peer0/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p org2.example.com/org2peer0/orgMsp/msp org2.example.com/org2peer0/localMsp/msp`

8. Prepare the orgMsp files

   `cp org2.example.com/org2peer0/msp/config.yaml org2.example.com/org2peer0/orgMsp/msp && cp -R org2.example.com/org2peer0/msp/cacerts org2.example.com/org2peer0/orgMsp/msp && cp -R org2.example.com/org2peer0/msp/tlscacerts org2.example.com/org2peer0/orgMsp/msp`

9. Prepare the localMsp files

   `cp org2.example.com/org2peer0/msp/config.yaml org2.example.com/org2peer0/localMsp/msp && cp -R org2.example.com/org2peer0/msp/cacerts org2.example.com/org2peer0/localMsp/msp && cp -R org2.example.com/org2peer0/msp/tlscacerts org2.example.com/org2peer0/localMsp/msp && cp -R org2.example.com/org2peer0/msp/keystore org2.example.com/org2peer0/localMsp/msp && cp -R org2.example.com/org2peer0/msp/signcerts org2.example.com/org2peer0/localMsp/msp`

10. Create the following folder structure

    ```
    ├── organizations
        └── peerOrganizations
            └── org2.example.com
                ├── msp
                └── peers
                    └── peer0.org2.example.com
                        ├── msp
                        └── tls
                        └── storage
                           └── snapshots
    ```

    `cd Hyperledger-Fabric-Prod-Network`

   `mkdir -p organizations/peerOrganizations/org2.example.com/msp organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/storage/snapshots`

11. Copy the peer binary and core.yaml file from _fabric_ into _organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com_  folder. Then add the location of the peer binary to PATH environment variable

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric/bin/peer organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com && cp -R fabric/config/core.yaml organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com`

   `cd organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com`

   `export PATH=$PWD:$PATH`

12. Prepare i) TLS certificates, ii) peer local MSP files, and iii) peer org MSP files.

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric-ca-client/org2.example.com/org2peer0/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tls-cert.pem`

   `cp -R fabric-ca-client/org2.example.com/org2peer0/localMsp/msp/signcerts/org2-peer0-cert.pem organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls`

   `cp -R fabric-ca-client/org2.example.com/org2peer0/localMsp/msp/keystore/org2-peer0-key.pem organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls`

   `cp -R fabric-ca-client/org2.example.com/org2peer0/localMsp/msp organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com`

   `cp -R fabric-ca-client/org2.example.com/org2peer0/orgMsp/msp organizations/peerOrganizations/org2.example.com`

13. Edit _core.yaml_ in _organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com_ with the respective values:

   ```
   peer.id: peer0.org2.example.com

   peer.networkId: prod

   peer.listenAddress: 192.168.0.146:9051

   peer.chaincodeListenAddress: 192.168.0.146:9052

   peer.address: 192.168.0.146:9051

   gossip.bootstrap: 192.168.0.146:9051

   gossip.endpoint: 192.168.0.146:9051

   gossip.externalEndpoint: 118.100.204.232:9051 (Adjust according to your public IP address with portforwarding)

   peer.tls.enabled: true

   peer.tls.rootcert.file: tls/tls-cert.pem

   peer.tls.cert.file: tls/org2-peer0-cert.pem

   peer.tls.key.file: tls/org2-peer0-key.pem

   peer.mspConfigPath: msp

   peer.fileSystemPath: storage

   peer.localMspId: Org2MSP

   ledger.snapshots.rootDir: /storage/snapshots

   operations.listenAddress: 127.0.0.1:11545
   ```

14. Start the peer

   `cd Hyperledger-Fabric-Prod-Network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com`

   `export FABRIC_CFG_PATH=$PWD`

   `sudo ./peer node start`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

2. Output from peer states there are No active channels passed. (which is understandable because we have not joined our peer to any channel yet, but hope it is not an error for the next steps ahead.)

#############################################################################

Setup ordererOrg1 CA Server, then register and enroll ordererOrg1 admin identity instruction steps.

1.  Deploy the ordererOrg1 CA

    `cd Hyperledger-Fabric-Prod-Network`

    `mkdir -p fabric-ca-server-ordererOrg1/tls`

    `cp hyperledger-fabric-ca-linux-amd64-1.5.3/bin/fabric-ca-server fabric-ca-server-ordererOrg1`

    `cp fabric-ca-client/tls-ca/rcaadmin/msp/signcerts/cert.pem fabric-ca-server-ordererOrg1/tls && cp fabric-ca-client/tls-ca/rcaadmin/msp/keystore/key.pem fabric-ca-server-ordererOrg1/tls`

    `cd fabric-ca-server-ordererOrg1`

    `./fabric-ca-server init -b ordererOrg1rcaadmin:ordererOrg1rcaadminpw`

2.  Edit _fabric-ca-server-config.yaml_ under _fabric-ca-server-ordererOrg1_ folder with the following values:

    ```
    port: 7057
    tls.enabled: true
    tls.certfile: tls/cert.pem
    tls.keyfile: tls/key.pem
    ca.name: ordererOrg1-ca
    operations.listenAddress: 127.0.0.1:9447

    csr:
       cn: fabric-ca-server
       keyrequest:
          algo: ecdsa
          size: 256
       names:
        - C: Malaysia
          ST: Kuala Lumpur
          L:
          O: Hyperledger
          OU: Fabric
       hosts:
          - kky-dell
          - localhost
       ca:
          expiry: 876000h
          pathlength: 1

    signing:
        default:
            usage:
                - digital signature
            expiry: 876000h
        profiles:
            ca:
                usage:
                    - cert sign
                    - crl sign
                expiry: 876000h
                caconstraint:
                    isca: true
                    maxpathlen: 0
            tls:
                usage:
                    - signing
                    - key encipherment
                    - server auth
                    - client auth
                    - key agreement
                expiry: 876000h
    ```

3.  Delete _fabric-ca-server-ordererOrg1/ca-cert.pem_ and entire _fabric-ca-server-ordererOrg1/msp_ folder, and then start the CA.

    `rm ca-cert.pem`

    `rm -r msp`

    `./fabric-ca-server start`

4.  Open new terminal tab:

    `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

    `export FABRIC_CA_CLIENT_HOME=$PWD`

    `./fabric-ca-client enroll -d -u https://ordererOrg1rcaadmin:ordererOrg1rcaadminpw@kky-dell:7057 --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost' --mspdir ordererOrg1-ca/rcaadmin/msp`

    rename the file in _fabric-ca-client/ordererOrg1-ca/rcaadmin/msp/keystore_ folder to _ordererOrg1-key.pem_

5.  Register and enroll ordererOrg1 admin identity

    `./fabric-ca-client register -d --id.name ordererOrg1admin --id.secret ordererOrg1adminpw -u https://kky-dell:7057 --mspdir ./ordererOrg1-ca/rcaadmin/msp --id.type admin --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

    `./fabric-ca-client enroll -u https://ordererOrg1admin:ordererOrg1adminpw@kky-dell:7057 --mspdir ./ordererOrg1.example.com/ordererOrg1admin/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

6.  create config.yaml file inside _ordererOrg1.example.com/ordererOrg1admin/msp_ folder with values of:

    ```
    NodeOUs:
        Enable: true
        ClientOUIdentifier:
            Certificate: cacerts/kky-dell-7057.pem
            OrganizationalUnitIdentifier: client
        PeerOUIdentifier:
            Certificate: cacerts/kky-dell-7057.pem
            OrganizationalUnitIdentifier: peer
        AdminOUIdentifier:
            Certificate: cacerts/kky-dell-7057.pem
            OrganizationalUnitIdentifier: admin
        OrdererOUIdentifier:
            Certificate: cacerts/kky-dell-7057.pem
            OrganizationalUnitIdentifier: orderer
    ```

7.  i) rename _ordererOrg1.example.com/ordererOrg1admin/msp/signcerts/cert.pem_ to _ordererOrg1-admin-cert.pem_

    ii) rename file inside _ordererOrg1.example.com/ordererOrg1admin/msp/keystore_ to _ordererOrg1-admin-key.pem_

8.  Create a tlscacerts folder and copy the tls-root-cert into this folder.

    `mkdir ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts`

    `cp tls-root-cert/tls-ca-cert.pem ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts`

9.  Create orgMsp and localMsp folder.

    `mkdir -p ordererOrg1.example.com//ordererOrg1admin/orgMsp/msp ordererOrg1.example.com/ordererOrg1admin/localMsp/msp`

10. Prepare the orgMsp files

    `cp ordererOrg1.example.com/ordererOrg1admin/msp/config.yaml ordererOrg1.example.com/ordererOrg1admin/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/cacerts ordererOrg1.example.com/ordererOrg1admin/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts ordererOrg1.example.com/ordererOrg1admin/orgMsp/msp`

11. Prepare the localMsp files

    `cp ordererOrg1.example.com/ordererOrg1admin/msp/config.yaml ordererOrg1.example.com/ordererOrg1admin/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/cacerts ordererOrg1.example.com/ordererOrg1admin/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts ordererOrg1.example.com/ordererOrg1admin/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/keystore ordererOrg1.example.com/ordererOrg1admin/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1admin/msp/signcerts ordererOrg1.example.com/ordererOrg1admin/localMsp/msp`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Register and enroll ordererOrg1osn0.

1. Start the ordererOrg1 CA

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-server-ordererOrg1`

   `./fabric-ca-server start`

   Open new terminal tab

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register ordererOrg1osn0 identity

   `./fabric-ca-client register -d --id.name ordererOrg1osn0 --id.secret ordererOrg1osn0pw -u https://kky-dell:7057 --mspdir ./ordererOrg1-ca/rcaadmin/msp --id.type orderer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll ordererOrg1osn0 identity

   `./fabric-ca-client enroll -u https://ordererOrg1osn0:ordererOrg1osn0pw@kky-dell:7057 --mspdir ./ordererOrg1.example.com/ordererOrg1osn0/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _ordererOrg1.example.com/ordererOrg1osn0/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _ordererOrg1.example.com/ordererOrg1osn0/msp/signcerts/cert.pem_ to _ordererOrg1-osn0-cert.pem_

   ii) rename file inside _ordererOrg1.example.com/ordererOrg1osn0/msp/keystore_ to _ordererOrg1-osn0-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir ordererOrg1.example.com/ordererOrg1osn0/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem ordererOrg1.example.com/ordererOrg1osn0/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp`

8. Prepare the orgMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn0/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/cacerts ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp`

9. Prepare the localMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn0/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/cacerts ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/keystore ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn0/msp/signcerts ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp`

10. Create the following folder structure

    ```
    ├── organizations
      └── ordererOrganizations
            └── ordererOrg1.example.com
               ├── msp
                  ├── cacerts
                  └── tlscacerts
               ├── orderers
                  └── osn0.ordererOrg1.example.com
                        ├── msp
                        └── tls
                        └── storage
                        └── admin-client
                           └── etcdraft
                              └── write-ahead-logs
                              └── snapshot
    ```

   `cd Hyperledger-Fabric-Prod-Network`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/tls`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/storage/etcdraft/write-ahead-logs`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/storage/etcdraft/snapshot`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/admin-client`

11. Prepare i) TLS certificates, ii) orderer local MSP files, iii) orderer org MSP files, and iv) admin-client certs.

   `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/tls/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp/signcerts/ordererOrg1-osn0-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp/keystore/ordererOrg1-osn0-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/localMsp/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp/cacerts/kky-dell-7057.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn0/orgMsp/msp/config.yaml organizations/ordererOrganizations/ordererOrg1.example.com/msp`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/keystore/ordererOrg1-admin-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/signcerts/ordererOrg1-admin-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com/admin-client`



NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Register and enroll ordererOrg1osn1.

1. Start the ordererOrg1 CA

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-server-ordererOrg1`

   `./fabric-ca-server start`

   Open new terminal tab

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register ordererOrg1osn1 identity

   `./fabric-ca-client register -d --id.name ordererOrg1osn1 --id.secret ordererOrg1osn1pw -u https://kky-dell:7057 --mspdir ./ordererOrg1-ca/rcaadmin/msp --id.type orderer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll ordererOrg1osn1 identity

   `./fabric-ca-client enroll -u https://ordererOrg1osn1:ordererOrg1osn1pw@kky-dell:7057 --mspdir ./ordererOrg1.example.com/ordererOrg1osn1/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _ordererOrg1.example.com/ordererOrg1osn1/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _ordererOrg1.example.com/ordererOrg1osn1/msp/signcerts/cert.pem_ to _ordererOrg1-osn1-cert.pem_

   ii) rename file inside _ordererOrg1.example.com/ordererOrg1osn1/msp/keystore_ to _ordererOrg1-osn1-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir ordererOrg1.example.com/ordererOrg1osn1/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem ordererOrg1.example.com/ordererOrg1osn1/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp`

8. Prepare the orgMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn1/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/cacerts ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp`

9. Prepare the localMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn1/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/cacerts ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/keystore ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn1/msp/signcerts ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp`

10. Create the following folder structure

    ```
    ├── organizations
      └── ordererOrganizations
            └── ordererOrg1.example.com
               ├── msp
                  ├── cacerts
                  └── tlscacerts
               ├── orderers
                  └── osn1.ordererOrg1.example.com
                        ├── msp
                        └── tls
                        └── storage
                        └── admin-client
                           └── etcdraft
                              └── write-ahead-logs
                              └── snapshot
    ```

    `cd Hyperledger-Fabric-Prod-Network`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/tls`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/storage/etcdraft/write-ahead-logs`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/storage/etcdraft/snapshot`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/admin-client`

11. Prepare i) TLS certificates, ii) orderer local MSP files, and iii) orderer org MSP files.

   `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/tls/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp/signcerts/ordererOrg1-osn1-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp/keystore/ordererOrg1-osn1-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/localMsp/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp/cacerts/kky-dell-7057.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn1/orgMsp/msp/config.yaml organizations/ordererOrganizations/ordererOrg1.example.com/msp`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/keystore/ordererOrg1-admin-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/signcerts/ordererOrg1-admin-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com/admin-client`


NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Register and enroll ordererOrg1osn2.

1. Start the ordererOrg1 CA

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-server-ordererOrg1`

   `./fabric-ca-server start`

   Open new terminal tab

   `cd Hyperledger-Fabric-Prod-Network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register ordererOrg1osn2 identity

   `./fabric-ca-client register -d --id.name ordererOrg1osn2 --id.secret ordererOrg1osn2pw -u https://kky-dell:7057 --mspdir ./ordererOrg1-ca/rcaadmin/msp --id.type orderer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll ordererOrg1osn2 identity

   `./fabric-ca-client enroll -u https://ordererOrg1osn2:ordererOrg1osn2pw@kky-dell:7057 --mspdir ./ordererOrg1.example.com/ordererOrg1osn2/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _ordererOrg1.example.com/ordererOrg1osn2/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7057.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _ordererOrg1.example.com/ordererOrg1osn2/msp/signcerts/cert.pem_ to _ordererOrg1-osn2-cert.pem_

   ii) rename file inside _ordererOrg1.example.com/ordererOrg1osn2/msp/keystore_ to _ordererOrg1-osn2-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir ordererOrg1.example.com/ordererOrg1osn2/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem ordererOrg1.example.com/ordererOrg1osn2/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp`

8. Prepare the orgMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn2/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/cacerts ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp`

9. Prepare the localMsp files

   `cp ordererOrg1.example.com/ordererOrg1osn2/msp/config.yaml ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/cacerts ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/tlscacerts ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/keystore ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp && cp -R ordererOrg1.example.com/ordererOrg1osn2/msp/signcerts ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp`

10. Create the following folder structure

    ```
    ├── organizations
      └── ordererOrganizations
            └── ordererOrg1.example.com
               ├── msp
                  ├── cacerts
                  └── tlscacerts
               ├── orderers
                  └── osn2.ordererOrg1.example.com
                        ├── msp
                        └── tls
                        └── storage
                        └── admin-client
                           └── etcdraft
                              └── write-ahead-logs
                              └── snapshot
    ```

    `cd Hyperledger-Fabric-Prod-Network`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/tls`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/storage/etcdraft/write-ahead-logs`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/storage/etcdraft/snapshot`

   `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/admin-client`

11. Prepare i) TLS certificates, ii) orderer local MSP files, and iii) orderer org MSP files.

   `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/tls/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp/signcerts/ordererOrg1-osn2-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp/keystore/ordererOrg1-osn2-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/tls`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/localMsp/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp/cacerts/kky-dell-7057.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1osn2/orgMsp/msp/config.yaml organizations/ordererOrganizations/ordererOrg1.example.com/msp`

    `cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/keystore/ordererOrg1-admin-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/signcerts/ordererOrg1-admin-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/admin-client && cp -R fabric-ca-client/ordererOrg1.example.com/ordererOrg1admin/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com/admin-client`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Deploying the ordererOrg1 osn0.

1. Copy the orderer binary and orderer.yaml file from _fabric_ into _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com_  folder. Then add the location of the orderer binary to PATH environment variable

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric/bin/orderer organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com && cp -R fabric/config/orderer.yaml organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com`

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

2. Edit _orderer.yaml_ in _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com_ with the respective values:

   ```
   General.ListenAddress: 192.168.0.146

   General.ListenPort: 7050

   General.TLS.Enabled: true

   General.TLS.PrivateKey: tls/ordererOrg1-osn0-key.pem

   General.TLS.Certificate: tls/ordererOrg1-osn0-cert.pem

   General.LocalMSPDir: msp

   General.BootstrapMethod: none

   General.LocalMSPID: ordererOrg1MSP

   FileLedger.Location: storage

   ChannelParticipation.Enabled: true

   Operations.ListenAddress: 192.168.0.146:8443

   Admin.ListenAddress: 192.168.0.146:8564

   Admin.TLS.Enabled: true

   Admin.TLS.Certificate: tls/ordererOrg1-osn0-cert.pem

   Admin.TLS.PrivateKey: tls/ordererOrg1-osn0-key.pem

   Admin.TLS.ClientAuthRequired: true

   Admin.TLS.ClientRootCAs: admin-client/tls-ca-cert.pem

   Consensus.WALDir: storage/etcdraft/write-ahead-logs

   Consensus.SnapDir: storage/etcdraft/snapshot
   ```

3. Start the orderer

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn0.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

   `export FABRIC_CFG_PATH=$PWD`

   `sudo ./orderer start`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Deploying the ordererOrg1 osn1.

1. Copy the orderer binary and orderer.yaml file from _fabric_ into _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com_  folder. Then add the location of the orderer binary to PATH environment variable

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric/bin/orderer organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com && cp -R fabric/config/orderer.yaml organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com`

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

2. Edit _orderer.yaml_ in _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com_ with the respective values:

   ```
   General.ListenAddress: 192.168.0.146

   General.ListenPort: 8050

   General.TLS.Enabled: true

   General.TLS.PrivateKey: tls/ordererOrg1-osn1-key.pem

   General.TLS.Certificate: tls/ordererOrg1-osn1-cert.pem

   General.LocalMSPDir: msp

   General.BootstrapMethod: none

   General.LocalMSPID: ordererOrg1MSP

   FileLedger.Location: storage

   Operations.ListenAddress: 192.168.0.146:8453

   Admin.ListenAddress: 192.168.0.146:8546

   Admin.TLS.Enabled: true

   Admin.TLS.Certificate: tls/ordererOrg1-osn1-cert.pem

   Admin.TLS.PrivateKey: tls/ordererOrg1-osn1-key.pem

   Admin.TLS.ClientAuthRequired: true

   Admin.TLS.ClientRootCAs: admin-client/tls-ca-cert.pem

   ChannelParticipation.Enabled: true

   Consensus.WALDir: storage/etcdraft/write-ahead-logs

   Consensus.SnapDir: storage/etcdraft/snapshot
   ```

3. Start the orderer

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn1.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

   `export FABRIC_CFG_PATH=$PWD`

   `sudo ./orderer start`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################

Deploying the ordererOrg1 osn2.

1. Copy the orderer binary and orderer.yaml file from _fabric_ into _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com_  folder. Then add the location of the orderer binary to PATH environment variable

   `cd Hyperledger-Fabric-Prod-Network`

   `cp -R fabric/bin/orderer organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com && cp -R fabric/config/orderer.yaml organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com`

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

2. Edit _orderer.yaml_ in _organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com_ with the respective values:

   ```
   General.ListenAddress: 192.168.0.146

   General.ListenPort: 9050

   General.TLS.Enabled: true

   General.TLS.PrivateKey: tls/ordererOrg1-osn2-key.pem

   General.TLS.Certificate: tls/ordererOrg1-osn2-cert.pem

   General.BootstrapMethod: none

   General.LocalMSPDir: msp

   General.LocalMSPID: ordererOrg1MSP

   FileLedger.Location: storage

   Operations.ListenAddress: 192.168.0.146:8463

   Admin.ListenAddress: 192.168.0.146:8456

   Admin.TLS.Enabled: true

   Admin.TLS.Certificate: tls/ordererOrg1-osn2-cert.pem

   Admin.TLS.PrivateKey: tls/ordererOrg1-osn2-key.pem

   Admin.TLS.ClientAuthRequired: true

   Admin.TLS.ClientRootCAs: admin-client/tls-ca-cert.pem

   ChannelParticipation.Enabled: true

   Consensus.WALDir: storage/etcdraft/write-ahead-logs

   Consensus.SnapDir: storage/etcdraft/snapshot
   ```

3. Start the orderer

   `cd organizations/ordererOrganizations/ordererOrg1.example.com/orderers/osn2.ordererOrg1.example.com`

   `export PATH=$PWD:$PATH`

   `export FABRIC_CFG_PATH=$PWD`

   `sudo ./orderer start`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.

#############################################################################
