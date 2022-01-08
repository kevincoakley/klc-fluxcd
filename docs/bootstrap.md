# Bootstrap Kubernetes Cluster

### Setup the environment

Create github.sh:

    export GITHUB_TOKEN=<your-token>
    export GITHUB_USER=<your-username>

Run:

    source github.sh

### Run pre-installation checks

    flux check --pre

### Bootstrap the cluster 

    flux bootstrap github \
        --owner=$GITHUB_USER \
        --repository=<GITHUB REPO NAME> \
        --branch=main \
        --personal \
        --path=clusters/<ENVIRONMENT>

This command will create a new GtiHub repo if <GITHUB REPO NAME> doesn't already exist. 