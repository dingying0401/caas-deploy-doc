# harbor

---

安装harbor

## 安装步骤

> 生成harbor的配置文件

```
cat > harbor.cfg << EOF

## Configuration file of Harbor

#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname = $CAAS_DOMAIN_HARBOR

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = http

#Maximum number of job workers in job service  
max_job_workers = 3 

#Determine whether or not to generate certificate for the registry's token.
#If the value is on, the prepare script creates new root cert and private key 
#for generating token to access the registry. If the value is off the default key/cert will be used.
#This flag also controls the creation of the notary signer's cert.
customize_crt = on

#The path of cert and key files for nginx, they are applied only the protocol is set to https
ssl_cert = /caas_data/harbor_data/cert/server.crt
ssl_cert_key = /caas_data/harbor_data/cert/server.key

#The path of secretkey storage
secretkey_path = /caas_data/harbor_data/

#Admiral's url, comment this attribute, or set its value to NA when Harbor is standalone
admiral_url = NA

#Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
log_rotate_count = 50
#Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes. 
#If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G 
#are all valid.
log_rotate_size = 200M

#NOTES: The properties between BEGIN INITIAL PROPERTIES and END INITIAL PROPERTIES
#only take effect in the first boot, the subsequent changes of these properties 
#should be performed on web ui

#************************BEGIN INITIAL PROPERTIES************************

#Email account settings for sending out password resetting emails.

#Email server uses the given username and password to authenticate on TLS connections to host and act as identity.
#Identity left blank to act as username.
email_identity = 

email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

##The initial password of Harbor admin, only works for the first time when Harbor starts. 
#It has no effect after the first launch of Harbor.
#Change the admin password from UI after launching Harbor.
harbor_admin_password = Caas12345 

##By default the auth mode is db_auth, i.e. the credentials are stored in a local database.
#Set it to ldap_auth if you want to verify a user's credentials against an LDAP server.
auth_mode = ldap_auth

#The url for an ldap endpoint.

ldap_url = ldap://$CAAS_DOMAIN_LDAP



#A user's DN who has the permission to search the LDAP/AD server. 
#If your LDAP/AD server does not support anonymous search, you should configure this DN and ldap_search_pwd.
ldap_searchdn = cn=admin,dc=general,dc=com

#the password of the ldap_searchdn
ldap_search_pwd = generalcom

#The base DN from which to look up a user in LDAP/AD
ldap_basedn = ou=users,dc=general,dc=com

#Search filter for LDAP/AD, make sure the syntax of the filter is correct.
#ldap_filter = (objectClass=person)

# The attribute used in a search to match a user, it could be uid, cn, email, sAMAccountName or other attributes depending on your LDAP/AD  
ldap_uid = cn 

#the scope to search for users, 0-LDAP_SCOPE_BASE, 1-LDAP_SCOPE_ONELEVEL, 2-LDAP_SCOPE_SUBTREE
ldap_scope = 2 

#Timeout (in seconds)  when connecting to an LDAP Server. The default value (and most reasonable) is 5 seconds.
ldap_timeout = 5

#Verify certificate from LDAP server
ldap_verify_cert = true

#Turn on or off the self-registration feature
self_registration = on

#The expiration time (in minute) of token created by token service, default is 30 minutes
token_expiration = 30

#The flag to control what users have permission to create projects
#The default value "everyone" allows everyone to creates a project. 
#Set to "adminonly" so that only admin user can create project.
project_creation_restriction = everyone

#************************END INITIAL PROPERTIES************************

#######Harbor DB configuration section#######

#The address of the Harbor database. Only need to change when using external db.
db_host = mysql

#The password for the root user of Harbor DB. Change this before any production use.
db_password = root123

#The port of Harbor database host
db_port = 3306

#The user name of Harbor database
db_user = root

##### End of Harbor DB configuration#######

#The redis server address. Only needed in HA installation.
redis_url =

##########Clair DB configuration############

#Clair DB host address. Only change it when using an exteral DB.
clair_db_host = postgres

#The password of the Clair's postgres database. Only effective when Harbor is deployed with Clair.
#Please update it before deployment. Subsequent update will cause Clair's API server and Harbor unable to access Clair's database.
clair_db_password = password

#Clair DB connect port
clair_db_port = 5432

#Clair DB username
clair_db_username = postgres

#Clair default database
clair_db = postgres

##########End of Clair DB configuration############

#The following attributes only need to be set when auth mode is uaa_auth
uaa_endpoint = uaa.mydomain.org
uaa_clientid = id
uaa_clientsecret = secret
uaa_verify_cert = true
uaa_ca_cert = /path/to/ca.pem


### Docker Registry setting ###
#registry_storage_provider can be: filesystem, s3, gcs, azure, etc.
registry_storage_provider_name = filesystem
#registry_storage_provider_config is a comma separated "key: value" pairs, e.g. "key1: value, key2: value2".
#Refer to https://docs.docker.com/registry/configuration/#storage for all available configuration.
registry_storage_provider_config =
# caas web portal api address
web_url = $CAAS_DOMAIN_PORTAL_API

EOF
```

> 安装harbor

```
cat > harbor.yaml << EOF

---
- hosts: storages
  tasks:
    - name: harbor dir create
      file: path=/caas_data/harbor_data state=directory
    - name: unarchive harbor-offline-installer-stage config
      unarchive: src=../images/harbor/harbor-offline-installer-stage.tgz  dest=/opt/
    - name: unarchive clair-db-fetched
      unarchive: src=../images/harbor/clair-db-fetched.gz dest=/caas_data/harbor_data/
    - name: copy harbor config
      copy: src=./harbor.cfg dest=/opt/harbor/ force=true
    - name: install harbor 
      shell: /opt/harbor/install.sh --with-clair

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./harbor.yaml
```

Next: [nfs](/nfs.md)

