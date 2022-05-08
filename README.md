# Helm charts for deploying the Composer service with an OSBuild Worker VM
This repository includes the Helm charts to deploy the OSBuild Composer service with an OSBuild Worker VM on Openshift

## Generate the Certificates
Use a patched version of the certificate generation script from the osbuild-composer repository to generate the certificates and soft link to the ones needed by the chart

### Export some environment variables
```shell
export CHARTS_REPO_DIR=<>
export CERTS_DIR=<>
```
### Create the certificates and link to them
```shell
# Clone the repo
git clone https://github.com/osbuild/osbuild-composer.git
cd osbuild-composer

# Patch the script and run it
patch -p1 < ${CHARTS_REPO_DIR}/tools/certs_fix.patch
./tools/gen-certs.sh test/data/x509/openssl.cnf ${CERTS_DIR} /tmp/cadir

# Create the certificates directory
mkdir -p ${CHARTS_REPO_DIR}/composer/files/certs/

# Link the created certificates into the composer chart
ln -s ${CERTS_DIR}/ca-crt.pem ${CHARTS_REPO_DIR}/composer/files/certs/
ln -s ${CERTS_DIR}/composer-crt.pem ${CHARTS_REPO_DIR}/composer/files/certs/
ln -s ${CERTS_DIR}/composer-key.pem ${CHARTS_REPO_DIR}/composer/files/certs/

# Create the certificates directory
mkdir -p ${CHARTS_REPO_DIR}/worker/files/certs/

# Link the created certificates into the worker chart
ln -s ${CERTS_DIR}/ca-crt.pem ${CHARTS_REPO_DIR}/worker/files/certs/
ln -s ${CERTS_DIR}/worker-crt.pem ${CHARTS_REPO_DIR}/worker/files/certs/
ln -s ${CERTS_DIR}/worker-key.pem ${CHARTS_REPO_DIR}/worker/files/certs/
```

## Generate SSH Keys for the OSBuild Worker VM
```shell
ssh-keygen -t rsa -b 4096 -C cloud-user@osbuild-worker -f ~/.ssh/osbuild-worker
```

From the root of the repository, create symlinks to the key pair you just created:

```shell
# Create the certificates directory
mkdir -p ${CHARTS_REPO_DIR}/worker/files/ssh/

ln -s ~/.ssh/osbuild-worker ${CHARTS_REPO_DIR}/worker/files/ssh/osbuild-worker
ln -s ~/.ssh/osbuild-worker.pub ${CHARTS_REPO_DIR}/worker/files/ssh/osbuild-worker.pub
```

## Deploying the Composer Service
### Using the example
You can use the provided example either as is or update it to your need
```shell
# Deploy the PostgreSQL Server
oc new-app --env-file example/psql.env postgresql:13-el8

# Install the chart
helm install <REALEASE_NAME> -f example/values.yaml ./composer
```

### The long way
#### PostgreSQL Server

##### Create an Environment file
```
POSTGRESQL_USER=<PSQL_USERNAME>
POSTGRESQL_PASSWORD=<PSQL_PASSWORD>
POSTGRESQL_DATABASE=<PSQL_DB_NAME>
```

##### Deploy PostgreSQL
```shell
oc new-app --env-file <ENV_FILENAME>  postgresql:13-el8
```

#### Values file
Create a `values.yaml` file for Helm values. The example lists the mandatory values
```yaml
deployment:
  composer:
    image_tag: fc87b17
    pgsslmode: disable
db:
  user: <PSQL_USERNAME>
  password: <PSQL_PASSWORD>
  name: <PSQL_DB_NAME>
```

#### Deploy the Composer Service
```shell
helm install <REALEASE_NAME> -f values.yaml ./composer
```

## Deploying the OSBuild Worker VM

### Base Image
Will be covered later

### Values File
Complete the values file under `example/worker/values.yaml` with your credentials

### Deploy the OSBuild Worker VM
```shell
# Install the chart
helm install <REALEASE_NAME> -f example/worker/values.yaml ./worker
```
