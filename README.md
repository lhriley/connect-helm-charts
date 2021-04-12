# 1Password Connect Helm Charts

This repository hosts the offical 1Password Helm Charts for deploying 1Password Connect and the 1Password Connect Kubernetes Operator.

## Install Helm

Get the latest [Helm release](https://github.com/kubernetes/helm#install).
## Configuration Guide

### Deploying 1Password Connect
Installing the Helm Chart with default configurations will deploy 1Password Connect in your default Namespace. However, Using 1Password Connect in Kubernetes requires that a 1password-credentials.json file be stored as a Kubernetes Secret. This credentials file can be saved as a Kubernetes secret by adding the following to your helm install command:
            
```bash
--set-file connect.credentials={path/to/1password-credentials.json}
```
    
More information about 1Password Connect and how to generate a 1password-credentials.json file can be found at https://support.1password.com/cs/connect/.

### Deploying 1Password Connect Kubernetes Operator
In order to deploy the 1Password Connect Kubernetes Operator along side 1Password Connect set "operator.create=true" in your install command.

Please note the following:

1. This operator expects that a secret containing an API token for 1Password Connect is saved to the configured namespace {{ $namespace }}.
Creation of a secret for the token can be automated by the Helm Chart by setting `operator.token` to your token's value.

If you would prefer to create the token secret manually, the token can be saved as a Kubernetes secret using the following command:

```bash
$ kubectl create secret generic {{ $tokenName --from-literal=token=<OP_CONNECT_TOKEN> \ --namespace={{ $namespace }}
```

More information about 1Password Connect and how to generate a 1Password Connect API token can be found at https://support.1password.com/cs/connect/.

2. The Operator also requires setting permissions in Kubernetes in order to allow it to run. By setting `operator.serviceAccount.create`, `operator.roleBinding.create`, and `operator.clusterRole.create` to true, the Helm chart will take care of setting a basic service account, role binding, and cluster role for the opreator. Although this process can be automated please ensure you give the Operator the appropriate permissions for your environment.
### Configuration Values
The 1Password Connect Helm chart offers many configuration options for deployment. Please refer to the list below for information on what configuration options are available as well as what the default configuration options are.

[From the Official Helm Install Guide](https://helm.sh/docs/helm/helm_install/#helm-install):

>To override values in a chart, use either the '--values' flag and pass in a file or use the '--set' flag and pass configuration from the command line, to force a string value use '--set-string'. In case a value is large and therefore you want not to use neither '--values' nor '--set', use '--set-file' to read the single large value from file.

For example: 
```bash
$ helm install -f myvalues.yaml connect ./connect
```

or 

```bash
$ helm install --set connect.applicationName=connect connect ./connect
```

#### Values
| Key | Type | Default | Description |
|-----|------|---------|-------------|
| connect.applicationName | string | `"onepassword-connect"` | The name of 1Password Connect Application |
| connect.api.imageRepository | string | `"1password/connect-api` | The 1Password Connect API repository |
| connect.api.name | string | `"connect-api"` | The name of the 1Password Connect API container |
| connect.api.resources | object | `{}` | The resources requests/limits for the 1Password Connect API pod |
| connect.credentials | jsonString |  | Contents of the 1password-credentials.json file for Connect. Can be set be adding `--set-file connect.credentials={path/to/1password-credentials.json}` to your helm install command |
| connect.credentialsKey | string | `"op-session"` | The key for the 1Password Connect Credentials stored in the credentials secret |
| connect.credentialsName | string | `"op-credentials"` | The name of Kubernetes Secret containing the 1Password Connect credentials |
| connect.dataVolume.name | string | `"shared-data"` | The name of the shared volume used between 1Password Connect Containers |
| connect.dataVolume.type | string | `"emptyDir"` | The type of the shared volume used between 1Password Connect Containers |
| connect.dataVolume.values | object | `{}` | Desribes the fields and values for configuration of shared volume for 1Password Connect |
| connect.imagePullPolicy | string | `"IfNotPresent` | The 1Password Connect API image pull policy |
| connect.sync.imageRepository | string | `"1password/connect-sync` | The 1Password Connect Sync repository |
| connect.sync.name | string | `"connect-sync"` | The name of the 1Password Connect Sync container |
| connect.sync.resources | object | `{}` | The resources requests/limits for the 1Password Connect Sync pod |
| connect.version | string | `"0.3"` | The 1Password Connect version to pull |
| namespace | string | `"default"` | The namespace in which to deploy 1Password Connect |
| operator.autoRestart | boolean | `false` | Denotes whether the 1Password Connect Operator will automatically restart deployments based on associated updated secrets. |
| operator.create | boolean | `false` | Denotes whether the 1Password Connect Operator will be deployed |
| operator.imagePullPolicy | string | `"IfNotPresent` | The 1Password Connect Operator image pull policy |
| operator.imageRepository | string | `"1password/onepassword-operator` | The 1Password Connect Operator repository |
| operator.pollingInterval | integer | `600` | How often the 1Password Connect Operator will poll for secrets updates. |
| operator.clusterRole.create | boolean | `false` | Denotes whether or not a cluster role will be created for each for the 1Password Connect Operator |
| operator.clusterRole.name | string | `"onepassword-connect-operator"` | The name of the 1Password Connect Operator Cluster Role |
| operator.roleBinding.create | boolean | `false` | Denotes whether or not a role binding will be created for each Namespace for the 1Password Connect Operator Service Account |
| operator.roleBinding.name | string | `"onepassword-connect-operator"` | The name of the 1Password Connect Operator Role Binding |
| operator.serviceAccount.annotations | object | `{}` | Annotations for the 1Password Connect Service Account |
| operator.serviceAccount.create | boolean | `false` | Denotes whether or not a service account will be created for the 1Password Connect Operator |
| operator.serviceAccount.name | string | `"onepassword-connect-operator"` | The name of the 1Password Conenct Operator |
| operator.version | string | `"1.0.0"` | T 1Password Connect Operator version to pull |
| operator.token.key | string | `"token"` | The key for the 1Password Connect token stored in the 1Password token secret |
| operator.token.name | string | `"onepassword-token"` | The name of Kubernetes Secret containing the 1Password Connect API token |
| operator.token.value | string | `"onepassword-token"` | An API token generated for 1Password Connect to be used by the Connect Operator |
| operator.watchNamespace | {} | `"-default"` | A list of Namespaces for the 1Password Connect Operator to watch and manage |

## Preparing a Release

1. Create tgz file with chart in current directory:
```bash
helm package connect
```

2. Create the index.yaml file which references the connect chart
```bash
helm repo index .
```

3. Push tgz and updated index.yaml file to Github repo.
