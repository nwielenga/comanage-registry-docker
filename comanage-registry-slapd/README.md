<!--
COmanage Registry Docker documentation

Portions licensed to the University Corporation for Advanced Internet
Development, Inc. ("UCAID") under one or more contributor license agreements.
See the NOTICE file distributed with this work for additional information
regarding copyright ownership.

UCAID licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with the
License. You may obtain a copy of the License at:

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# OpenLDAP slapd for COmanage Registry

A simple example demonstrating how to create an image and container
based on OpenLDAP slapd to use with COmanage Registry containers. 

## Build

```
docker build -t comanage-registry-slapd .
```

## Run

Create a user-defined network bridge with

```
docker network create --driver=bridge \
  --subnet=192.168.0.0/16 \
  --gateway=192.168.0.100 \
  comanage-registry-internal-network
```

and then mount a directory such as `/tmp/slapd-data`
to `/var/lib/ldap` inside the container to persist
data and `/tmp/slapd-config` to `/etc/ldap/slapd.d`
inside the container to persist the configuration, eg.

```
docker run -d --name comanage-registry-slapd \
  -v /tmp/slapd-data:/var/lib/ldap \
  -v /tmp/slapd-config:/etc/ldap/slapd.d \
  --network comanage-registry-internal-network \
  -p 389:389 \
  sphericalcowgroup/comanage-registry-slapd

```

The following environment variables can be set:

* `OLC_SUFFIX`: Directory suffix (default is `dc=my,dc=org`)
* `OLC_ROOT_DN`: DN for the directory admin (default is `cn=admin,dc=my,dc=org`)
* `OLC_ROOT_PW`: Password for the root DN (default is `password`)

For example

```
docker run -d --name comanage-registry-slapd \
  -v /tmp/slapd-data:/var/lib/ldap \
  -v /tmp/slapd-config:/etc/ldap/slapd.d \
  --network comanage-registry-internal-network \
  -e OLC_SUFFIX=dc=my,dc=org \
  -e OLC_ROOT_DN=cn=admin,dc=my,dc=org \
  -e OLC_ROOT_PW=password \
  -p 389:389 \
  comanage-registry-slapd
```

To support TLS mount or copy in an X.509 certificate, private key,
and CA signing certificate or chain file like this:

```
docker run -d --name comanage-registry-slapd \
  -v /tmp/slapd-data:/var/lib/ldap \
  -v /tmp/slapd-config:/etc/ldap/slapd.d \
  -v my.crt:/etc/ldap/slapd.crt \
  -v my.key:/etc/ldap/slapd.key \
  -v chain.pem:/etc/ldap/slapd.ca.crt \
  --network comanage-registry-internal-network \
  -e OLC_SUFFIX=dc=my,dc=org \
  -e OLC_ROOT_DN=cn=admin,dc=my,dc=org \
  -e OLC_ROOT_PW=password \
  -p 389:389 -p 636:636 \
  sphericalcowgroup/comanage-registry-slapd
```

You may also use environment variables that point to files, for example

```
docker run -d --name comanage-registry-slapd \
  --network comanage-registry-internal-network \
  -v /tmp/slapd-data:/var/lib/ldap \
  -v /tmp/slapd-config:/etc/ldap/slapd.d \
  -e SLAPD_CERT_FILE=/run/secrets/slapd_cert_file \
  -e SLAPD_PRIVKEY_FILE=/run/secrets/slapd_privkey_file \
  -e SLAPD_CHAIN_FILE=/run/secrets/slapd_chain_file \
  -e OLC_SUFFIX=dc=my,dc=org \
  -e OLC_ROOT_DN=cn=admin,dc=my,dc=org \
  -e OLC_ROOT_PW_FILE=/run/secrets/olc_root_pw \
  -p 389:389 -p 636:636 \
  sphericalcowgroup/comanage-registry-slapd
```

