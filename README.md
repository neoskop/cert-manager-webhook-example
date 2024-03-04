# DNSimple Webhook for cert-manager

A [cert-manager][2] ACME DNS01 solver webhook for [DNSimple][1].

## Pre-requisites

- [cert-manager][2] >= 0.13 (The Helm chart uses the new API versions)
- Kubernetes >= 1.17.x
- Helm 3 (otherwise adjust the example below accordingly)

## Quickstart

1. Take note of your DNSimple API token from the account settings in the automation tab. 
2. Run the following commands replacing the API token placeholders and email address:
  ```bash
  $ helm repo add neoskop https://charts.neoskop.dev
  $ helm install cert-manager-webhook-dnsimple \
      --namespace cert-manager \
      --dry-run \
      --set dnsimple.token='<DNSIMPLE_API_TOKEN>' \
      --set clusterIssuer.production.enabled=true \
      --set clusterIssuer.staging.enabled=true \
      --set clusterIssuer.email=email@example.com \
      neoskop/cert-manager-webhook-dnsimple
  ```
  (Alternatively you can check out this repository and substitute the source of the install command with `./deploy/dnsimple`.)

3. Afterwards issue a certificate:

  ```bash
  $ cat << EOF | kubectl apply -f -
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: dnsimple-test
    namespace: default
  spec:
    dnsNames:
      - test.example.com
    issuerRef:
      name: cert-manager-webhook-dnsimple-production
      kind: ClusterIssuer
    secretName: dnsimple-test-tls
  EOF
  ```

## Options

The Helm chart accepts the following values:

| name                               | required | description                                     | default value                           |
| ---------------------------------- | -------- | ----------------------------------------------- | --------------------------------------- |
| `dnsimple.token`                   | ✔️       | DNSimple API Token                              | _empty_                                 |
| `clusterIssuer.email`              |          | LetsEncrypt Admin Email                         | `name@example.com`                      |
| `clusterIssuer.production.enabled` |          | Create a production `ClusterIssuer`             | `false`                                 |
| `clusterIssuer.staging.enabled`    |          | Create a staging `ClusterIssuer`                | `false`                                 |
| `image.repository`                 | ✔️       | Docker image for the webhook solver             | `neoskop/cert-manager-webhook-dnsimple` |
| `image.tag`                        | ✔️       | Docker image tag of the solver                  | `latest`                                |
| `image.pullPolicy`                 | ✔️       | Image pull policy of the solver                 | `IfNotPresent`                          |
| `logLevel`                         |          | Set the verbosity of the solver                 | _empty_                                 |
| `groupName`                        | ✔️       | Identifies the company that created the webhook | `acme.neoskop.de`                       |
| `certManager.namespace`            | ✔️       | The namespace cert-manager was installed to     | `cert-manager`                          |
| `certManager.serviceAccountName`   | ✔️       | The service account cert-manager runs under     | `cert-manager`                          |

## Test suite

All cert-manager webhooks have to pass the DNS01 provider conformance testing suite. To run that test suite on this plug-in download the test binaries:

```bash
$ mkdir -p __main__/hack
$ wget -O- https://storage.googleapis.com/kubebuilder-tools/kubebuilder-tools-1.14.1-linux-amd64.tar.gz | tar xz --strip-components=1 -C __main__/hack
```

Then set-up `testdata/dnsimple/config.json` and `testdata/dnsimple/dnsimple-token.yaml` according to the [README][3].

Execute the test suite replacing `TEST_ZONE_NAME` with a DNS name you have control over with your DNSimple account:

```bash
# Mind the trailing dot in the TEST_ZONE_NAME value
$ TEST_ZONE_NAME=example.com. go test .
```

## Release

After you committed all of your changes, run the following command to tag a new version and build and push a new Docker image tag as well as a new Helm chart:

```bash
$ ./scripts/release.sh <patch|minor|major>
```

[1]: https://dnsimple.com/
[2]: https://cert-manager.io/docs/installation/kubernetes/
[3]: ./testdata/dnsimple/README.md
