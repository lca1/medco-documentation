---
description: Deployment of profile test-network.
---

# Network Deployment

This profile deploys an arbitrary set of MedCo nodes independently in different machines that together form a MedCo network. This deployment assumes each node is deployed in a single dedicated machine. All the machines have to be reachable between each other. Nodes should agree on a network name and individual indexes beforehand \(to be assigned a unique ID\).

{% hint style="warning" %}
The next set of steps must be **fully executed individually by each node of the network**.
{% endhint %}

## Pre-requisites

First step is to get the MedCo Deployment latest release at each node. Adapt `${MEDCO_SETUP_DIR}` to where you wish to install MedCo.

```bash
export MEDCO_SETUP_DIR=~/medco MEDCO_SETUP_VER=v2.0.1
git clone --depth 1 --branch ${MEDCO_SETUP_VER} https://github.com/ldsec/medco.git ${MEDCO_SETUP_DIR}

#add ownership rights to current user
sudo groupadd medco
sudo groupadd docker
sudo chown :medco -R "${MEDCO_SETUP_DIR}" # medco group is the owner of the installation directory
sudo chmod g+wrx "${MEDCO_SETUP_DIR}" -R #add rights to medco group on installation directory
sudo usermod -aG docker,medco "${USER}" #add current user to docker and medco groups
sudo systemctl restart docker #restart docker to take group modifications into account
su ${USER} #relog so that groups modifications are taken into account
```

## Generation of the deployment Profile

Next the _compose and configuration profiles_ must be generated using a script, executed in two steps.

* **Step 1**: each node generates its keys and certificates, and shares its public information with the other nodes
* **Step 2**: each node collects the public keys and certificates of the all the other nodes

### Step 1

For step 1, the network name `${MEDCO_SETUP_NETWORK_NAME}` should be common to all the nodes. `${MEDCO_SETUP_NODE_DNS_NAME}` corresponds to the machine domain name where the node is being deployed. As mentioned before the different parties should have agreed beforehand on the members of the network, and assigned an index `${MEDCO_SETUP_NODE_IDX}` to each different node to construct its UID \(starting from `0`, to `n-1`, `n` being the total number of nodes\). The nodes id's need to contain 3 digits (`0` translates to `000`, `1` to `001`).

{% hint style="warning" %}
If the following step did not complete correctly, please delete the newly created folder (network-${MEDCO_SETUP_NETWORK_NAME}-${MEDCO_SETUP_NODE_IDX}) before executing `step1.sh` again.
{% endhint %}

```bash
export MEDCO_SETUP_NETWORK_NAME=example \
    MEDCO_SETUP_NODE_IDX=0 \
    MEDCO_SETUP_NODE_DNS_NAME=medconode0.example.com
cd "${MEDCO_SETUP_DIR}/scripts/network-profile-tool"
bash step1.sh ${MEDCO_SETUP_NETWORK_NAME} ${MEDCO_SETUP_NODE_IDX} ${MEDCO_SETUP_NODE_DNS_NAME}
```

This script will generate the _compose profile_ and part of the _configuration profile_, including a file `srv${MEDCO_SETUP_NODE_IDX}-public.tar.gz`. This file should be shared with the other nodes, and all of them need to place it in their _configuration profile_ folder \(`${MEDCO_SETUP_DIR}/deployments/test-network-${MEDCO_SETUP_NETWORK_NAME}-node${MEDCO_SETUP_NODE_IDX}/configuration`\).

### Step 2

{% hint style="warning" %}
Before proceeding to this step, you need to have gathered **all** the files `srv${MEDCO_SETUP_NODE_IDX}-public.tar.gz` from the persons deploying MedCo on the other nodes.
{% endhint %}

Once all nodes have shared their `srv${MEDCO_SETUP_NODE_IDX}-public.tar.gz` file with all other nodes, step 2 can be executed:

```bash
cd "${MEDCO_SETUP_DIR}/scripts/network-profile-tool"
bash step2.sh ${MEDCO_SETUP_NETWORK_NAME} ${MEDCO_SETUP_NODE_IDX}
```

At this point, it is possible to edit the default configuration generated in `${MEDCO_SETUP_DIR}/deployments/test-network-${MEDCO_SETUP_NETWORK_NAME}-node${MEDCO_SETUP_NODE_IDX}/.env` This is needed in order [to modify the default passwords](configuration/passwords.md). When editing this file, be careful to change only the passwords and not the other values. 

The deployment profile is now ready to be used.

## MedCo Stack Deployment

Next step is to download the docker images and run the node:

```bash
cd "${MEDCO_SETUP_DIR}/deployments/test-network-${MEDCO_SETUP_NETWORK_NAME}-node${MEDCO_SETUP_NODE_IDX}"
make pull
make up
```

Wait some time for the initialization of the containers to be done, this can take up to 10 minutes. For the subsequent runs, the startup will be faster. You can use `make stop` to stop the containers and `make down` to delete them.

## Keycloak Configuration

You will need to follow two sets of instruction to make Keycloak functional and be able to log in. [Access the Keycloak administration interface](configuration/keycloak.md#accessing-the-web-administration-interface) and then:

* [Update the MedCo OIDC client](configuration/keycloak.md#medco-openid-connect-client)
* [Update the Keycloak realm keys](configuration/keycloak.md#changing-default-realm-keys)

## Test the deployment

{% hint style="warning" %}
Note that by default the certificates generated by the script are self-signed and thus, when using Glowing Bear, the browser will issue a security warning. To use your own valid certificates, see [HTTPS Configuration](configuration/https-configuration.md). If you wish anyway to use the self-signed certificates, you will need to visit individually the page of Glowing Bear of **all** nodes in your browser, and select to trust the certificate.
{% endhint %}

{% hint style="info" %}
The database is pre-loaded with some encrypted test data using a key that is pre-generated from the combination of all the participating nodes’ public keys. For the _network_ deployment profile this data will not be correctly encrypted, since the public key of each node is generated independently, and, as such, the data must be re-loaded before being able to test the system successfully.
{% endhint %}

Run first the MedCo loader \(see [Loading Data](../data-loading/)\) to load some data and be able to test this deployment. Or to load some test data by performing a simple data loading you can execute the following:

```bash
make load_test_data
```

Then access Glowing Bear in your web browser at `https://${MEDCO_SETUP_NODE_DNS_NAME}` and use the default credentials specified in [Keycloak user management](configuration/keycloak.md#user-management). If you are new to Glowing Bear you can watch the [Glowing Bear user interface walkthrough](https://glowingbear.app/) video. You can also use the [CLI client](../cli.md) to perform tests.

