# SOPS Encryption

### Generate GPG keys

Create gpg key with no password

    export KEY_NAME="cluster0.yourdomain.com"
    export KEY_COMMENT="flux secrets"

    gpg --batch --full-generate-key <<EOF
    %no-protection
    Key-Type: 1
    Key-Length: 4096
    Subkey-Type: 1
    Subkey-Length: 4096
    Expire-Date: 0
    Name-Comment: ${KEY_COMMENT}
    Name-Real: ${KEY_NAME}
    EOF

View gpg key

    gpg --list-secret-keys "${KEY_NAME}"
    
    sec   rsa3072 2020-09-06 [SC]
        2BBDE18E48819FBF60185DBA413EA63E1D611582

Store the key fingerprint as an environment variable

    export KEY_FP=2BBDE18E48819FBF60185DBA413EA63E1D611582

### Added the GPG private key as a kubernetes secret

Create kubernetes secret with the gpg private key

    gpg --export-secret-keys --armor "${KEY_FP}" |
    kubectl create secret generic sops-gpg \
    --namespace=flux-system \
    --from-file=sops.asc=/dev/stdin

View the kubernetes secret

    kubectl -n flux-system describe secrets sops-gpg

### Configure the Flux cluster settings

Add the sops decryption to apps/infrastructure cluster settings

    apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
    kind: Kustomization
    metadata:
    name: infrastructure
    namespace: flux-system
    spec:
    # content omitted for brevity
    decryption:
        provider: sops
        secretRef:
        name: sops-gpg

### Add the gpg public key to the repo

    gpg --export -a "${KEY_NAME}" > public.key

Then commit public.key

### Create the encrypted kubernetes secret YAML

Create the kubernetes secret

    kubectl -n <NAMESPACE> create secret generic <SECRET_NAME> \
    --from-literal=admin-user=admin \
    --from-literal=admin-password=password \
    --dry-run=client \
    -o yaml > path/to/secret.yaml

SOPS encrypt the secret

    sops --encrypt \
    --pgp="${KEY_FP}" \
    --encrypted-regex '^(data|stringData)$' \
    --in-place path/to/secret.yaml

### Add the encrypted kubernetes secret YAML to the kustomization files