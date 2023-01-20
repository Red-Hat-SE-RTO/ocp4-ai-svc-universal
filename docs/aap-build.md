# Instructions for building a AAP Execution Enviornment


### Prerequisites
* podman/docker 
* [ansible-builder](https://ansible-builder.readthedocs.io/en/stable/)  ```pip install ansible-builder```

**Clone the repo and cd into the directory**
```
git clone https://github.com/Red-Hat-SE-RTO/ocp4-ai-svc-universal.git
cd ocp4-ai-svc-universal
git checkout aap
```

**Build the container image**
```
ansible-builder build -v 3
```