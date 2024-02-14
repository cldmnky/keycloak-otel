# Notes for setting up prereqs in the lab cluster

## MetalLB

1. Install the metalLB operator
2. Install the metalLB helm chart:

  ```shell
  cd ../home-lab/charts/metallb
  helm upgrade -i metallb . \
    --namespace metallb-system \
    --set addressPool=10.202.62.1-10.202.62.254
  cd -
  ```

## ExternalDNS

1. Install the ExternaDNS Operator
2. Install the ExternalDNS helm chart:

  ```shell
  cd ../home-lab/charts/external-dns
  EXTERNALDNS_SECRET_ACCESS_KEY=$(cat ../../.pull-secrets/external-dns | jq -r '.AccessKey.SecretAccessKey')
  EXTERNALDNS_ACCESS_KEY=$(cat ../../.pull-secrets/external-dns | jq -r '.AccessKey.AccessKeyId')
  EXTERNALDNS_ZONE=$(cat ../../.pull-secrets/hostedZoneID)
  oc new-project external-dns-operator
  helm upgrade -i external-dns . \
    --set sealedSecrets=false \
    --namespace external-dns-operator \
    --set aws.hostedZoneID=${EXTERNALDNS_ZONE} \
    --set aws.domainName=hosted.blahonga.me \
    --set aws.access_key_id=${EXTERNALDNS_ACCESS_KEY} \
    --set aws.secret_access_key=${EXTERNALDNS_SECRET_ACCESS_KEY} \
    --set installOperator=false
  # Hack the node role to allow externaldns to run on hypershift
  oc label node hosted-54b821a5-cwv7c node-role.kubernetes.io/master=""
  cd -
  ```

## Cert-Manager

1. Install the cert-manager operator
2. Install the helm chart:

  ```shell
  cd  ../home-lab/charts/cert-manager
  CERTMANAGER_SECRET_ACCESS_KEY=$(cat ../../.pull-secrets/certmanager | jq -r '.AccessKey.SecretAccessKey' | base64)
  CERTMANAGER_ACCESS_KEY=$(cat ../../.pull-secrets/certmanager | jq -r '.AccessKey.AccessKeyId' | base64)
  helm upgrade -i  cert-manager . \
    --namespace cert-manager \
    --set sealedSecrets=false \
    --set AWS.secretAccessKey=${CERTMANAGER_SECRET_ACCESS_KEY} \
    --set AWS.accessKeyID=${CERTMANAGER_ACCESS_KEY} \
    --set "testCerts={hosted.blahonga.me,*.hosted.blahonga.me}"
  cd -
  ```