# Sealed Secrets
In the world of GitOps, we follow the rule: "If it's not in Git, it's not real." But, this rule brings a challenge when it comes to managing and storing sensitive data like credentials. Git repositories are accessible to many people, making it crucial to ensure the security of sensitive data stored within.

While Kubernetes offers a mechanism for managing secrets, its method of storing sensitive data as a base64 string is not adequately secure. Base64 is an encoding, not an encryption method, meaning anyone can easily decode a base64 string and gain access to the underlying data. Consequently, it's not advisable to openly store Secret manifest files.

To resolve this issue, we leverage an open-source tool known as Sealed Secrets.

The Sealed Secrets project is an open-source tool by Bitnami Labs, designed to solve the problem of managing sensitive information – such as credentials, passwords, or certificates – that needs to be used in a Kubernetes environment. It lets you safely store and manage Kubernetes Secrets in Git repositories, which is otherwise risky due to their sensitive nature.

Sealed Secrets are composed of a **controller** and **kubeseal** a CLI:

- The controller is watching for SealedSecret resources.  When it detects one, it decrypts it back into a regular Kubernetes Secret using the corresponding private key. This decrypted Secret can then be used as usual in the Kubernetes cluster.


- Kubeseal: This tool converts regular Kubernetes Secrets into SealedSecrets. When a Secret is "sealed", it's encrypted with a public key, rendering it safe to store – even in a public repository.

The encryption/decryption process is one-way, meaning that once a Secret has been sealed, it cannot be viewed or retrieved except by the controller in the cluster. The only entity that can decrypt a SealedSecret is the controller running in the cluster that it was encrypted for, using the corresponding private key which never leaves the cluster.

This provides a simple and secure method to store and manage Kubernetes Secrets, particularly in a GitOps workflow, where everything, including Secrets, needs to be stored in Git repositories.

## Configuration

Download Kubeseal:
```
brew install kubseal
# Linux: download tarfile from 
https://github.com/bitnami-labs/sealed-secrets/releases/tag/v0.21.0
# Mac OS
Brew install kubeseal
```
### Command exmaple
```
kubeseal < /tmp/secret.yaml > /tmp/sealed-secrets.yaml \
    --controller-namespace sealed-secrets \
    --controller-name sealed-secrets \
    -o yaml
```
The command above will seal the secret that is located in /tmp/secret.yaml.

```--controller-namespace``` This specifies the namespace where the Sealed Secrets controller (which handles the decryption) is running.

```--controller-name sealed-secrets```: This specifies the name of the Sealed Secrets controller.

In the long term providing the controller namespace and name might be tidius, therefore you can set in your 
.bashrc file an alias as follow:
```
alias scseal='kubeseal --controller-namespace sealed-secrets --controller-name sealed-secrets'
```
