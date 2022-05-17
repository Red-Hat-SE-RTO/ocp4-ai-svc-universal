# OpenShift Assisted Installer Service, Universal Deployer

This set of resources handles an idempotent way to deploy OpenShift via the Assisted Installer Service to any number of infrastructure platforms.

## Operations

The following tasks performed by the different Playbooks are:

### bootstrap.yaml

- Preflight for binaries, create asset generation directory, set HTTP Headers & Authentication
- Preflight checks for supported OpenShift versions on the Assisted Installer Service
- Preflight to do connectivity tests to infrastructure platforms
- Query the AI Svc, check for existing cluster with the same name, set facts if so
- Set needed facts, or create a new cluster with new SSH keys
- Configure cluster on the AI Svc with Cluster/InfraEnv deployment specs and ISO Params
- Download the generated Discovery ISO
- Upload the generated Discovery ISO to the target infrastructure platforms
- Create & boot the needed infrastructure resources on the target infrastructure platforms
- Wait for the hosts to report into the AI Svc
- Set Host Names and Roles on the AI Svc
- Set Network VIPs on the AI Svc
- Wait for the hosts to be ready
- Start the cluster installation on the AI Svc
- Wait for the cluster to be fully installed
- Pull cluster credentials from the AI Svc

## Prerequisites

- Ansible Automation - `python3 -m pip install ansible`
- cURL - `dnf install curl`
- Git - `dnf install git`
- jq - `dnf install jq`

### One-time | Installing oc

There are a few Ansible Tasks that use the `command` module to execute commands best/easiest serviced by the `oc` binary - thusly, `oc` needs to be available in the system path

```bash
## Create a binary directory if needed
sudo mkdir -p /usr/local/bin
sudo echo 'export PATH="/usr/local/bin:$PATH"' > /etc/profile.d/usrlibbin.sh
sudo chmod a+x /etc/profile.d/usrlibbin.sh
source /etc/profile.d/usrlibbin.sh

## Download the latest oc binary
mkdir -p /tmp/bindl
cd /tmp/bindl
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
tar zxvf openshift-client-linux.tar.gz

## Set permissions and move it to the bin dir
sudo chmod a+x kubectl
sudo chmod a+x oc

sudo mv kubectl /usr/local/bin
sudo mv oc /usr/local/bin

## Clean up
cd -
rm -rf /tmp/bindl
```

## Usage

### Clone the repo

```bash
git clone http://github.com/kenmoini/ocp4-ai-svc-universal.git
cd ocp4-ai-svc-universal
```
### One-time | Installing Needed Pip Packages

Before running this Ansible content, you will need to install the `kubernetes` and `openshift` pip packages, among others - you can do so in one shot by running the following command:

```bash
python3 -m pip install --upgrade -r requirements.txt
```

### One-time | Installing Ansible Collections

In order to run this Playbook you'll need to have the needed Ansible Collections already installed - you can do so easily by running the following command:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Modify the Variables files

- Copy `example_vars/cluster-config.yaml` to the working directory, ideally with a prefix of the cluster name - modify as needed.

```bash
cp example_vars/cluster-config.yaml CLUSTER_NAME.cluster-config.yaml
```

- Copy the other relevant files from `example_vars/` to `vars/` or the working directory and modify as you see fit, such as those for infrastructure credentials.

### Running the Playbook

With the needed variables altered, you can run the Playbook with the following command:

```bash
ansible-playbook -e "@CLUSTER_NAME.cluster-config.yaml" bootstrap.yaml
```

### Destroying the Cluster

If you are done with the cluster or some error occurred you can quickly delete it from your infrastructure environments, the Assisted Installer Service, and the local assets that were generated during creation:

```bash
ansible-playbook -e "@CLUSTER_NAME.cluster-config.yaml" destroy.yaml
```
