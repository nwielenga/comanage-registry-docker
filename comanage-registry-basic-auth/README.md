# COmanage Registry Basic Auth

Intended to build a COmanage Registry image
using the official PHP 7 with Apache image as the foundation
and providing Apache HTTP Server Basic Auth as the authentication
mechanism. 

Basic Auth is only suitable for COmanage Registry deployments
not operating in a federated identity context, or for an introduction
to COmanage Registry. 

See other templates in this repository for examples on how to build images 
that support federated identity deployments.

## Build

```
export COMANAGE_REGISTRY_VERSION=develop
sed -e s/%%COMANAGE_REGISTRY_VERSION%%/${COMANAGE_REGISTRY_VERSION}/g Dockerfile.template  > Dockerfile
docker build -t comanage-registry-basic-auth:${COMANAGE_REGISTRY_VERSION} .
```

You can (and should) use build arguments to bootstrap the first
platform administrator. The administrator username is the value
COmanage Registry expects to read from $REMOTE\_USER after
the administrator authenticates using whichever authentication
method is provided:

```
export COMANAGE_REGISTRY_VERSION=develop

export COMANAGE_REGISTRY_ADMIN_GIVEN_NAME=Karel
export COMANAGE_REGISTRY_ADMIN_FAMILY_NAME=Novak
export COMANAGE_REGISTRY_ADMIN_USERNAME=karel.novak@my.org

sed -e s/%%COMANAGE_REGISTRY_VERSION%%/${COMANAGE_REGISTRY_VERSION}/g Dockerfile.template  > Dockerfile
docker build \
  --build-arg COMANAGE_REGISTRY_ADMIN_GIVEN_NAME=${COMANAGE_REGISTRY_ADMIN_GIVEN_NAME} \
  --build-arg COMANAGE_REGISTRY_ADMIN_FAMILY_NAME=${COMANAGE_REGISTRY_ADMIN_FAMILY_NAME} \
  --build-arg COMANAGE_REGISTRY_ADMIN_USERNAME=${COMANAGE_REGISTRY_ADMIN_USERNAME} \
  -t comanage-registry:${COMANAGE_REGISTRY_VERSION} .
```
## Run

### Database

COmanage Registry requires a relational database. See the 
[PostgreSQL example for COmanage Registry](../comanage-registry-postgres/README.md).

### Network

Create a user-defined network bridge with

```
docker network create --driver=bridge \
  --subnet=192.168.0.0/16 \
  --gateway=192.168.0.100 \
  comanage-registry-internal-network
```

### Configuration

Create a directory to hold persistent COmanage Registry configuration and
other state such as local plugins and other customizations. In that directory
create a `Config` directory and in it place a `database.php` and `email.php`
configuration file:

```
mkdir -p /opt/comanage-registry/Config

cat >> /opt/comanage-registry/Config/database.php <<"EOF"
<?php

class DATABASE_CONFIG {

  public $default = array(
    'datasource' => 'Database/Postgres',
    'persistent' => false,
    'host' => 'comanage-registry-database',
    'login' => 'registry_user',
    'password' => 'password',
    'database' => 'registry',
    'prefix' => 'cm_',
  );

}
EOF

cat >> /opt/comanage-registry/Config/database.php <<"EOF"
<?php

class EmailConfig {

  public $default = array(
    'transport' => 'Smtp',
    'host' => 'tls://smtp.gmail.com',
    'port' => 465,
    'username' => 'account@gmail.com',
    'password' => 'password'
  );
}
EOF
```

### Container

```
docker run -d --name comanage-registry \
  -v /opt/comanage-registry:/local \
  -v /opt/passwords:/etc/apache2/passwords \
  --network comanage-registry-internal-network \
  -p 80:80 -p 443:443 \
  comanage-registry:${COMANAGE_REGISTRY_VERSION}
```

### Authentication

Mount or COPY in a password file created with `htpasswd`.

```
COPY passwords /etc/apache2/passwords
```

### Logging

Both Apache HTTP Server and COmanage Registry log to the stdout and
stderr of the container.

### HTTPS Configuration

Mount or COPY in an X.509 certificate file, associated private key file,
and certificate signing chain file.

```
COPY cert.pem /etc/apache2/cert.pem
COPY privkey.pem /etc/apache2/privkey.pem
COPY chain.pem /etc/apache2/chain.pem
```