# Hyperledger Fabric v1.0 test network-config

Required Dockerfiles and configuration files (certificates, channel definition and genesis_block)
to create a HLF network with one orderer, two organizations each with its own certification authority (ca)
and each with one peer (the state of the peers is stored in a CouchDB container)




#Add the folder where the configtxgen and cryptogen binaries live to the PATH for ease of use.
e.g.    export HLF_V1=/home/nikos/hlfLocalhost/v1/fabric-samples/bin
	export PATH=$HLF_V1:$JAVA_HOME/jre/bin:$PATH


#You can manually generate the certificates/keys and the various configuration artifacts using the configtxgen and cryptogen commands.

#Generate certificates (this consumes the file crypto-config.yaml that contains the network topology)
cryptogen generate --config=./crypto-config.yaml

#First, we need to set an environment variable to specify where configtxgen should look for the configtx.yaml configuration file:
export FABRIC_CFG_PATH=$PWD

#create the orderer genesis block:
configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block


#Next, we need to create the channel transaction artifact.

export CHANNEL_NAME=mychannel
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

#Next, define the anchor peer for Org1 on the channel that we are constructing
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

#Now, we will define the anchor peer for Org2 on the same channel:
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP


#Next, start the network (for deamon -d) consuming the docker-compose.yml config file:
docker-compose -f docker-compose.yml up

#Finally, the crypto-config folder, channel.tx, genesis.block, Org1MSPanchors.tx and Org2MSPanchors.tx need to
#be copied to the app to be able to access the certificates, channel  definition etc.
#(usually, these are copies a folder called artifacts in the root of the app)
