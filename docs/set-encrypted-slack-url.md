# Set Encrypted Slack URL

### Create the kubernetes secret

    kubectl -n flux-system create secret generic slack-url \
    --from-literal=address=https://hooks.slack.com/services/XXX/XXX/XXX \
    --dry-run=client \
    -o yaml > infrastructure/notifications/slack-url-secret.yaml

### SOPS encrypt the secret 

See sops-encryption.md for ${KEY_FP}

    sops --encrypt \
    --pgp="${KEY_FP}" \
    --encrypted-regex '^(data|stringData)$' \
    --in-place infrastructure/notifications/slack-url-secret.yaml