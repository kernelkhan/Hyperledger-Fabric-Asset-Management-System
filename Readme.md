
# Blockchain-Based Asset Management System

This project is a blockchain solution for managing and tracking financial assets, built on Hyperledger Fabric. It features a Go-based smart contract (chaincode) to handle asset transactions and a containerized REST API server, also in Go, to interact with the blockchain network.

This system fulfills the requirements of creating, updating, and querying assets, ensuring security, transparency, and immutability of all records.

---

## 📋 Prerequisites

Before you begin, ensure you have the following installed and configured on your system (preferably a Linux environment or WSL 2 on Windows):

* **Docker and Docker Compose:** The entire network runs in containers.
* **Go Programming Language:** Version 1.24 or higher is required for the API server.
* **Hyperledger Fabric Samples:** You must have the `fabric-samples` repository cloned, as this project relies on its `test-network` script. This project directory (`fabric-api-server`) should be placed alongside the `fabric-samples` directory.

Your directory structure should look like this:



Fabric/
├── fabric-api-server/  \<-- This project
│   ├── main.go
│   ├── go.mod
│   ├── go.sum
│   └── Dockerfile
└── fabric-samples/     \<-- The official Hyperledger Fabric samples
└── test-network/





## 🚀 Quickstart: Running the Entire Project

The following script is designed to be run from a **single, clean terminal**. It will perform all necessary steps: clean the Docker environment, start the Fabric network, deploy the chaincode, build the API server image, and run the final container.

**Open a terminal and paste the entire command block below.**


#  STEP 1: FULL CLEANUP AND NETWORK RESTART 
echo "--- Cleaning up Docker and starting fresh network ---"
cd /mnt/d/Fabric/fabric-samples/test-network
./network.sh down
docker system prune -af

# STEP 2: START NETWORK & DEPLOY CHAINCODE 
echo "--- Starting Fabric network and deploying chaincode ---"
./network.sh up createChannel -c mychannel -ca
./network.sh deployCC -ccn account -ccp ../asset-transfer-basic/chaincode-go -ccl go -i '{"function":"InitLedger","Args":[]}'

#  STEP 3: PREPARE AND BUILD THE API SERVER 
echo "--- Preparing and building the API server Docker image ---"
cd /mnt/d/Fabric/fabric-api-server

# CRITICAL: Update crypto material with the fresh set from the new network
echo "--- Updating crypto material ---"
rm -rf ./organizations
cp -r /mnt/d/Fabric/fabric-samples/test-network/organizations ./

# CRITICAL: Tidy go modules to match the code
echo "--- Tidying Go modules ---"
go mod tidy

# Build the final image, using --no-cache to be safe
echo "--- Building Docker image ---"
docker build --no-cache -t fabric-api-server .

#  STEP 4: RUN THE API SERVER 
echo "---"
echo "---"
echo "--- BUILD SUCCESSFUL. STARTING API SERVER CONTAINER. ---"
echo "--- Leave this terminal running. ---"
echo "--- Open a NEW terminal and run the final curl command. ---"
echo "---"
echo "---"
docker run --rm -p 8080:8080 --network fabric_test --name api-server fabric-api-server


After the script finishes, the last line you see will be `Starting server on port 8080`. **Leave this terminal running.**



## 🧪 Testing the API

To test that the entire system is working, **open a new, separate terminal** and run the following `curl` command. This will query the blockchain for the asset with the MSISDN `1234567890`, which was created by the `InitLedger` function.


curl http://localhost:8080/accounts/1234567890


#### ✅ Expected Successful Output

You should see a JSON response similar to this, confirming that the API server queried the blockchain and retrieved the asset data:

```json
{"BALANCE":100,"DEALERID":"d1","MPIN":"1234","MSISDN":"1234567890","REMARKS":"Initial balance","STATUS":"active","TRANSAMOUNT":0,"TRANSTYPE":"credit"}
```

-----

## 🏛️ Project Architecture

This project consists of three main components:

### 1\. Hyperledger Fabric Network

  * Managed by the `test-network` script from `fabric-samples`.
  * Consists of two peer organizations (Org1, Org2) and an orderer organization.
  * Creates a single channel named `mychannel`.
  * All components run in Docker containers on a dedicated network named `fabric_test`.

### 2\. Smart Contract (Chaincode)

  * Written in Go.
  * The source code is located in the `fabric-samples/asset-transfer-basic/chaincode-go` directory.
  * Implements functions to manage assets, including `InitLedger` to populate initial data and `ReadAccount` to query an asset.

### 3\. REST API Server

  * A Go application that acts as a gateway to the blockchain network.
  * Uses the Fabric Gateway SDK to connect to the peer and invoke smart contract transactions.
  * Containerized using the provided `Dockerfile`.
  * Exposes a simple REST endpoint to query asset data.

