# CP4I-Commandline-CheatSheet

## Cloudctl

```
# Login with fetched route
cloudctl login -a $(oc get routes -n kube-system icp-console -ojsonpath='{.spec.host}') -n eventstreams --skip-kubectl-config

# Or single command to login as default CP4I admin 
cloudctl login -n integration -a $(oc get routes -n kube-system icp-console -ojsonpath='{.spec.host}') -u admin -p $(oc get secrets -n kube-system platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode ) --skip-kubectl-config

# List local charts pushed in if offline install
cloudctl catalog charts --repo local-charts
```

## Cloudctl for Managing EventStreams
```
# Choose instance if there are multiple deployed
cloudctl es init -n eventstreams


# Permissions need to be added to IAM team in common services for new ES instances
# to access the ES UI properly. Otherwise you will see 403 error even if user has
# cluster admin role via IAM teams

cloudctl iam teams

# Install if it isn't already 
cloudctl plugin install <es-plugin-path>

cloudctl es init -n eventstreams

# Add the team to the ES release
cloudctl es iam-add-release-to-team --team "Team Name"

# Download the ES cert using the CLI instead of the UI

# JKS
cloudctl es certificates

# PEM
cloudctl es certificates --format pem
```


## Finding Admin Creds
```
# Default Admin Password

# 2020.2
oc get secrets -n ibm-common-services platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode && echo "" 

# 2020.1
oc get secrets -n kube-system platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode && echo "" 
```


## Deploying Capabilities
```
# Copying ibm-entitlement-key from one namespace to another
oc get secret ibm-entitlement-key --namespace=src-ns --export -o yaml | kubectl apply --namespace=target-ns -f -

# Get Secret for deploying local-chart into the target namespace
oc get secret -n mq | grep 'deployer-dockercfg'
```


## Helm CLI via Web Terminal

```
helm init --client-only

# Find home user directory
ls -lah ~/

# Use the user id
helm repo add local-charts https://icp-console.your.cluster.url:443/helm-repo/charts --ca-file /home/65551/.helm/ca.pem --cert-file /home/65551/.helm/cert.pem --key-file /home/65551/.helm/key.pem

helm repo list

helm search local-charts/ --versions
```


## Handy Helm Commands
```
# List all charts in local-chart repo (must be added first like using prior steps)
helm search local-charts

# Search for a specific chart
helm search local-charts/ibm-eventstreams-icp4i-prod

# Upgrade the number of brokers in the Kafka cluster
helm upgrade --reuse-values --set kafka.brokers=3 es-admin-deployed local-charts/ibm-eventstreams-icp4i-prod --tls
```



## Openshift Internal Registry
```
# Get exposed registry route
oc get route default-route -n openshift-image-registry -ojsonpath='{.spec.host}'

# As kubeadmin
docker login $(oc get route default-route -n openshift-image-registry -ojsonpath='{.spec.host}') -u kubeadmin -p $(oc whoami -t) 

# As another user
docker login $(oc get route default-route -n openshift-image-registry -ojsonpath='{.spec.host}') -u $(oc whoami) -p $(oc whoami -t) 
```

## Accessing IBM Entitled Registry
```
# Entitlement key for deployment use
oc create secret docker-registry ibm-entitlement-key \
    --docker-username=cp \
    --docker-password=<api-key> \
    --docker-server=cp.icr.io \
    --namespace=my-namespace

# Login locally
docker login cp.icr.io --username cp --password <api-key>
```



## EventStreams REST API
```
# Store the variables for a shorter curl
# All these info can be fetched from the ES UI Home page
REST_ROUTE=<rest_api_route>
ES_API_KEY=<api_key>
CA_CERT=<es_cert_pem_path>
TOPIC=<topic_name>

# REST API test
curl -k -v -X POST -H "Authorization: Bearer $ES_API_KEY" -H "Content-Type: text/plain" -H "Accept: application/json" -d 'test message' --cacert $CA_CERT "$REST_ROUTE/topics/$TOPIC/records"

# Simple Loop to keep sending
for ((i=1;i<=500;i++)); do curl -k -X POST -H "Authorization: Bearer $ES_API_KEY" -H "Content-Type: text/plain" -H "Accept: application/json" -d "test message $i" --cacert es-cert.pem "$REST_ROUTE/topics/multipartition/records"; sleep 1; done
```
