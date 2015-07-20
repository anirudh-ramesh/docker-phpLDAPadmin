# osixia/phpldapadmin

A docker image to run phpLDAPadmin.
> [phpldapadmin.sourceforge.net](http://phpldapadmin.sourceforge.net)


## Quick start

Run a phpLDAPadmin docker image by replacing `ldap.example.com` with your ldap host or IP :

    sudo docker run -p 443:443 \
               -e LDAP_HOSTS=ldap.example.com \
               -d osixia/phpldapadmin

That's it :) you can access phpLDAPadmin on **https://localhost**


## Examples

### OpenLDAP & phpLDAPadmin in 1'

Example script:

    #!/bin/bash -e

    # Run a ldap server, save the container id in LDAP_CID and get its IP:
    LDAP_CID=$(docker run -h ldap.example.org -d osixia/openldap:1.0.0)
    LDAP_IP=$(docker inspect -f "{{ .NetworkSettings.IPAddress }}" $LDAP_CID)

    # Run phpLDAPadmin and set ldap host to ldap ip
    PHPLDAP_CID=$(docker run -h phpldapadmin.example.org -e LDAP_HOSTS=$LDAP_IP -d osixia/phpldapadmin:0.6.0)

    # We get phpLDAPadmin container ip
    PHPLDAP_IP=$(docker inspect -f "{{ .NetworkSettings.IPAddress }}" $PHPLDAP_CID)

    echo "Go to: https://$PHPLDAP_IP"
    echo "Login DN: cn=admin,dc=example,dc=org"
    echo "Password: admin"

### HTTPS

#### Use autogenerated certificate
By default HTTPS is enable, a certificate is created with the container hostname (set by -h option eg: phpldapadmin.my-compagny.com).

	docker run -h phpldapadmin.my-compagny.com -d osixia/phpldapadmin

#### Use your own certificate

Add your custom certificate, private key and CA certificate in the directory **image/service/phpldapadmin/assets/apache2/ssl** adjust filename in **image/env.yaml** and rebuild the image ([see manual build](#manual-build)).

Or you can set your custom certificate at run time, by mouting your a directory containing thoses files to **/osixia/service/phpldapadmin/assets/apache2/ssl** and adjust there name with the following environment variables :

	docker run -v /path/to/certifates:/osixia/service/phpldapadmin/assets/apache2/ssl \
	-e SSL_CRT_FILENAME=my-phpldapadmin.crt \
	-e SSL_KEY_FILENAME=my-phpldapadmin.key \
	-e SSL_CA_CRT_FILENAME=the-ca.crt \
	-d osixia/phpldapadmin

Ommit the -e SSL_CA_CRT_FILENAME variable for self signed certificates

#### Disable HTTPS
Add -e HTTPS=false to the run command :

    docker run -e HTTPS=false -d osixia/phpldapadmin

## Environment Variables

Environement variables defaults are set in **image/env.yaml**. You can modify environment variable values directly in this file and rebuild the image ([see manual build](#manual-build)). You can also override those values at run time with -e argument or by setting your own env.yaml file as a docker volume to `/etc/env.yaml`. See examples below.

- **LDAP_HOSTS**: Set phpLDAPadmin server config. Defaults to :

		   - ldap.example.org:
		    - server:
		      - tls: true
		    - login:
		      - bind_id: cn=admin,dc=example,dc=org
		  - ldap2.example.org
		  - ldap3.example.org

	This will be converted in the phpldapadmin config.php file to :

		$servers->newServer('ldap_pla');
		$servers->setValue('server','name','ldap.example.org');
		$servers->setValue('server','host','ldap.example.org');
		$servers->setValue('server','tls',true);
		$servers->setValue('login','bind_id','cn=admin,dc=example,dc=org');
		$servers->newServer('ldap_pla');
		$servers->setValue('server','name','ldap2.example.org');
		$servers->setValue('server','host','ldap2.example.org');
		$servers->newServer('ldap_pla');
		$servers->setValue('server','name','ldap3.example.org');
		$servers->setValue('server','host','ldap3.example.org');

	If you want to set this variable at docker run command convert the yaml in python :
	
		docker run -e LDAP_HOSTS="[{'ldap.example.org': [{'server': [{'tls': True}]},{'login': [{'bind_id': 'cn=admin,dc=example,dc=org'}]}]}, 'ldap2.example.org', 'ldap3.example.org']" -d osixia/phpldapadmin

	To convert yaml to python online :
	http://yaml-online-parser.appspot.com/

Apache config :
- **SERVER_ADMIN**: Server admin email. Defaults to `webmaster@example.org`

HTTPS options :
- **HTTPS**: Use apache ssl config. Defaults to `true`
- **SSL_CRT_FILENAME**: Apache ssl certificate filename. Defaults to `phpldapadmin.crt`
- **SSL_KEY_FILENAME**: Apache ssl certificate private key filename. Defaults to `phpldapadmin.key`
- **SSL_CA_CRT_FILENAME**: Apache ssl CA certificate filename. Defaults to `ca.crt`

Ldap client TLS/LDAPS options :

More information at : 16.2.2. Client Configuration http://www.openldap.org/doc/admin24/tls.html

- **USE_LDAP_CLIENT_SSL**: Enable ldap client tls config, ldap serveur certificate check and set client  certificate. Defaults to `true`
- **LDAP_REQCERT**: Set ldap.conf TLS_REQCERT. Defaults to `demand`
- **LDAP_CA_CRT_FILENAME**: Set ldap.conf TLS_CACERT to /osixia/service/phpldapadmin/ssl/$LDAP_CA_CRT_FILENAME. Defaults to `ldap-ca.crt`
- **LDAP_CRT_FILENAME**: Set .ldaprc TLS_CERT to /osixia/service/phpldapadmin/ssl/$LDAP_CRT_FILENAME. Defaults to `ldap-client.crt`
- **LDAP_KEY_FILENAME**: Set .ldaprc TLS_KEY to /osixia/service/phpldapadmin/ssl/$LDAP_KEY_FILENAME. Defaults to `ldap-client.key`

### Set environment variables at run time :

Environment variable can be set directly by adding the -e argument in the command line, for example :

	docker run -h phpldapadmin.example.org -e LDAP_HOSTS="ldap.example.org" \
	-d osixia/phpldapadmin

Or by setting your own `env.yaml` file as a docker volume to `/etc/env.yaml`

	docker run -h ldap.example.org -v /data/my-ldap-env.yaml:/etc/env.yaml \
	-d osixia/openldap

## Manual build

Clone this project :

	git clone https://github.com/osixia/docker-phpLDAPadmin
	cd docker-phpLDAPadmin

Adapt Makefile, set your image NAME and VERSION, for example :

	NAME = osixia/phpldapadmin
	VERSION = 0.6.0

	becomes :
	NAME = billy-the-king/phpldapadmin
	VERSION = 0.1.0

Build your image :

	make build

Run your image :

	docker run -d billy-the-king/phpldapadmin:0.1.0

## Tests

We use **Bats** (Bash Automated Testing System) to test this image:

> [https://github.com/sstephenson/bats](https://github.com/sstephenson/bats)

Install Bats, and in this project directory run :

	make test
