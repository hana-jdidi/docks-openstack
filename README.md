**Installation openstack rocky service par service**

  

  

_**1)environnement :**_

  

a)sécurité :

pour accepter les caractères spéciaux entré dans le mot de passe :

`$ openssl rand -hex 10 `
→ 5254d6acef68517f5a33

## b)Configurer les interfaces réseau

Fixez votre adresse IP 

## Configurer la résolution de nom[¶]

Définir le hostname du nœud à `controller`./_etc/hostname_

Éditer le fichier `/etc/hosts` pour qu’il contienne ce qui suit :

c)paquets openstack :

`# apt-get install software-properties-common`

 `#add-apt-repository cloud-archive:rocky `

  

base de Finaliser l’installation

Mettre à jour les packages sur tous les nœuds :

`# apt-get update && apt-get dist-upgrade`

`# reboot`

`# apt-get install python-openstackclient`

d)donnée SQL
*install le paquet

`# apt-get install mariadb-server python-pymysql`

*Créer et éditer le fichier `/etc/mysql/mariadb.conf.d/99-openstack.cnf` comme suit :

` [mysqld] ` 
` bind-address = 192.168.1.251`
` default-storage-engine = innodb `
` innodb_file_per_table = on `
` max_connections = 4096 `
` collation-server = utf8_general_ci `
` character-set-server = utf8 `

#redémarrer le service SQL
`# service mysql restart`

#pour sécuriser le service base de donnée
`# mysql_secure_installation`

et repondrez aux questions par y

d)File de message :

*install le paquet :

`#apt-get install rabbitmq-server`

*Ajouter l’utilisateur openstack

`# rabbitmqctl add_user openstack mind`

Creating user "openstack" ...

#Permet la configuration, les accès en lecture et écriture pour l’utilisateur alpenstock.

`# rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

Setting permissions for user "openstack" in vhost "/" ...

e)memcached :

*install le paquet :

`# apt-get install memcached python-memcache`

*éditer le fichier /etc/memcached.conf

`-l 192.168.1.251`

*restart le service memcached

`# service memcached restart`


f)etcd :un magasin de clé-valeur distribué fiable pour verrouiller des clés distribuées, stocker des configurations, tracer 

l’état des services.

*install le paquet :

`#apt-get install etcd`

*éditer le fichier comme suit `/etc/default/etcd` :

`ETCD_NAME="controller"

ETCD_DATA_DIR="/var/lib/etcd"

ETCD_INITIAL_CLUSTER_STATE="new"

ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"

ETCD_INITIAL_CLUSTER="controller=http://192.168.1.251:2380"

ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.251:2380"

ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.251:2379"

ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"

ETCD_LISTEN_CLIENT_URLS="http://192.168.1.251:2379"'`

*Activer et démarrer le service etcd

`# systemctl enable etcd`

`# systemctl start etcd`

# 2)service keystone

# utiliser` `le client bade de donnée` 

``# mysql`

# crier base de donnée keystone 

`MariaDB [(none)]>CREATE DATABASE keystone;`

# donnée le` `privilège` `:

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'mind';`

# exit base de donnée 

`exit` 

# installer les paquets :

`# apt-get install keystone  apache2 libapache2-mod-wsgi`

# éditer le fichier `/etc/keystone/keystone.conf` comme suit :

`[database]
# ...
connection = mysql+pymysql://keystone:mind@controller/keystone


[token]
# ...
provider = fernet`


# Remplir la base de données du service d'identité

`# su -s /bin/sh -c "keystone-manage db_sync" keystone`

# Remplir la base de données

`# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone`

`# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone`

# Démarrer le service d'identité


`# keystone-manage bootstrap --bootstrap-password mind \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne`

# configuration du service apache2 :

-éditer le ficher `/etc/apache2/apache2.conf` comme suit: 

ajoutez `ServerName controller`

-rédemarrer le service 

`# service apache2 restart `

-Configurer le compte administratif

`$ export OS_USERNAME=admin

$ export OS_PASSWORD=mind

$ export OS_PROJECT_NAME=admin

$ export OS_USER_DOMAIN_NAME=Default

$ export OS_PROJECT_DOMAIN_NAME=Default

$ export OS_AUTH_URL=http://controller:5000/v3

$ export OS_IDENTITY_API_VERSION=3`

# creation d’un domaine ,projet ,utilisateur et rôle
-Bien que le domaine «par défaut» existe déjà à l’étape de l’amorçage de keystone dans ce guide, un moyen formel de créer un nouveau domaine serait:
` openstack domain create --description "An Example Domain" example`


 # -Créer le projet de service

`$ openstack project create --domain default \

--description "Service Project" service`

# -crée le projet myproject et l'utilisateur myuser :



`$ openstack project create --domain default \

--description "Demo Project" myproject`

`$ openstack user create --domain default \

--password-prompt myuser`

-créer le role

`$openstack rôle create myrole`



# Ajoutez le rôle myrole au projet myproject et à l'utilisateur myuser

`$ openstack role add --project myproject --user myuser myrole`



# vérification :

-Désactiver les variables temporaires d'environnement

`$ unset OS_AUTH_URL OS_PASSWORD`



# -demandez un jeton d'authentification en tant que utilisateur admin :

`$ openstack --os-auth-url http://controller:5000/v3 \`

`  --os-project-domain-name Default --os-user-domain-name Default \

--os-project-name admin --os-username admin token issue`



# -demandez un jeton d'authentification en tant que utulisateur myuser :

`$ openstack --os-auth-url http://controller:5000/v3 \`

` --os-project-domain-name Default --os-user-domain-name Default \

--os-project-name myproject --os-username myuser token issue`



# Créer des scripts d'environnement client OpenStack :

#  -crée et éditer le fichier admin-openrc comme suit :

`export OS_PROJECT_DOMAIN_NAME=Default`

export OS_USER_DOMAIN_NAME=Default

export OS_PROJECT_NAME=admin

export OS_USERNAME=admin

export OS_PASSWORD=mind

export OS_AUTH_URL=http://controller:5000/v3

export OS_IDENTITY_API_VERSION=3

export OS_IMAGE_API_VERSION=2

# -crere et éditer le fichier demo-openrc comme suit :

`export OS_PROJECT_DOMAIN_NAME=Default

export OS_USER_DOMAIN_NAME=Default

export OS_PROJECT_NAME=myproject

export OS_USERNAME=myuser

export OS_PASSWORD=mind

export OS_AUTH_URL=http://controller:5000/v3

export OS_IDENTITY_API_VERSION=3

export OS_IMAGE_API_VERSION=2`

# Chargez le fichier admin-openrc pour renseigner les variables d'environnement avec l'emplacement du service Identité et les 
informations d'identification du projet et de l'utilisateur admin :

`$ . admin-openrc`

# Demander un jeton d'authentification



`$openstack token issue`



# **3)service glance:**


# *les paquets

-connecter a la base de donnée et créer une base de donnée glance

`#mysql`

`MariaDB [(none)]> CREATE DATABASE glance;`



# -donnée le privilège:

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \

IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \

IDENTIFIED BY 'mind';`

# exit base de donnée  

`exit` 

# Sourcez les informations d'identification:

`$ . admin-openrc`



# creation de l’utulisateur glance:

`$ openstack user create --domain default --password-prompt glance`



# Ajouter le rôle d'administrateur au projet d'utilisateur et de service de projet:

`$ openstack role add --project service --user glance admin`



-création de service glance:

`$ openstack service create --name glance \

--description "OpenStack Image" image`

-Créez les points de terminaison de l'API du service image



`$ openstack endpoint create --region RegionOne \

image public http://controller:9292

$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292

$ openstack endpoint create --region RegionOne \

image admin http://controller:9292`

-installer le paquet:

`# apt-get install glance`



-éditer le fichier `/etc/glance/glance-api.conf` comme suit:`

`configurez la connection avec la base de données dans la section database

`[database]

# ...

connection = mysql+pymysql://glance:mind@controller/glance`

configurer l’access au service keystone dans la section keystone_authtoken

`[keystone_authtoken]

# ...

www_authenticate_uri = http://controller:5000

auth_url = http://controller:5000

memcached_servers = controller:11211

auth_type = password

project_domain_name = Default

user_domain_name = Default

project_name = service

username = glance

password = mind



[paste_deploy]

# ...

flavor = keystone

[glance_store]

# ...

stores = file,http

default_store = file

filesystem_store_datadir = /var/lib/glance/images/`

-éditer le fichier `/etc/glance/glance-registry.conf` comme suit :

`[database]

# ...

connection = mysql+pymysql://glance:mind@controller/glance

[keystone_authtoken]

# ...

www_authenticate_uri = http://controller:5000

auth_url = http://controller:5000

memcached_servers = controller:11211

auth_type = password

project_domain_name = Default

user_domain_name = Default

project_name = service

username = glance

password = mind



[paste_deploy]

# ...

flavor = keystone`

emplir base de donnée

`# su -s /bin/sh -c "glance-manage db_sync" glance`

-restart le service d’image :

`# service glance-registry restart

# service glance-api restart`

*verification :

`$ . admin-openrc`

-télécharger l’image

`$ wget [http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img](http://download.cirros-cloud.net/0.4.0/cirros-
0.4.0-x86_64-disk.img)`

`-`Téléchargez l'image sur le service Image à l'aide du format de disque QCOW2, du format de conteneur nu et de la visibilité 
publique afin que tous les projets puissent y accéder.

`$ openstack image create "cirros" \

--file cirros-0.4.0-x86_64-disk.img \

--disk-format qcow2 --container-format bare \

--public`

`-`Confirmez le téléchargement de l'image et validez les attributs :

`$ openstack image list`

_**4)service nova :**_

_**nova controller**_ 

***** Pour créer les bases de données, procédez comme suit:

-Utilisez le client d'accès à la base de données

`# mysql`

-créer la base de donnée 

`MariaDB [(none)]> CREATE DATABASE nova_api;

MariaDB [(none)]> CREATE DATABASE nova;

MariaDB [(none)]> CREATE DATABASE nova_cell0;

MariaDB [(none)]> CREATE DATABASE placement;` 

-donnée le privilège :

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \

IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \

IDENTIFIED BY 'mind';



MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \

IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \

IDENTIFIED BY 'mind';



MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \

IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \

IDENTIFIED BY 'mind';


MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \

IDENTIFIED BY 'mind';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \

IDENTIFIED BY 'mind';`


`exit`
`$ . admin-openrc`

-créer utilisateur nova :

`$ openstack user create --domain default --password-prompt nova`

-Ajouter le rôle d'administrateur à l'utilisateur nova :

`$openstack role add --project service --user nova admin`

-Créer l'entité de service nova :

`$ openstack service create --name nova \

--description "OpenStack Compute" compute`

-Create the Compute API service endpoints:

`$ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1

$ openstack endpoint create --region RegionOne \

compute internal http://controller:8774/v2.1

$ openstack endpoint create --region RegionOne \

compute admin http://controller:8774/v2.1`

-Créez un utilisateur de service

`$ openstack user create --domain default --password-prompt placement`

-Ajouter l'utilisateur de placement au projet de service avec le rôle admin

`$ openstack role add --project service --user placement admin`

-Créer l'entrée de l'API d'emplacement dans le catalogue de services:

`$ openstack service create --name placement \

--description "Placement API" placement`

-Créer les points de terminaison du service API de placement

`$ openstack endpoint create --region RegionOne \

placement public http://controller:8778

$ openstack endpoint create --region RegionOne \

placement internal http://controller:8778

$ openstack endpoint create --region RegionOne \

placement admin http://controller:8778`



-installer les paquets 

`# apt-get install nova-api nova-conductor nova-consoleauth \

nova-novncproxy nova-scheduler nova-placement-api`

-éditer le fichier `/etc/nova/nova.conf` comme suit :

`[api_database]

# ...

connection = mysql+pymysql://nova:mind@controller/nova_api



[database]

# ...

connection = mysql+pymysql://nova:mind@controller/nova



[placement_database]

# ...

connection = mysql+pymysql://placement:mind@controller/placement

[DEFAULT]

# ...

transport_url = rabbit://openstack:RABBIT_PASS@controller

[api]

# ...

auth_strategy = keystone



[keystone_authtoken]

# ...

auth_url = http://controller:5000/v3

memcached_servers = controller:11211

auth_type = password

project_domain_name = default

user_domain_name = default

project_name = service

username = nova

password = mind

[DEFAULT]

# ...

my_ip = 192.168.1.251

[DEFAULT]

# ...

use_neutron = true

firewall_driver = nova.virt.firewall.NoopFirewallDriver

[vnc]

enabled = true

# ...

server_listen = $my_ip

server_proxyclient_address = $my_ip

[glance]

# ...

api_servers = http://controller:9292

[oslo_concurrency]

# ...

lock_path = /var/lib/nova/tmp

[placement]

# ...

region_name = RegionOne

project_domain_name = Default

project_name = service

auth_type = password

user_domain_name = Default

auth_url = http://controller:5000/v3

username = placement

password = mind`

-remplir la base de donnée nova_api

`# su -s /bin/sh -c "nova-manage api_db sync" nova`

-Enregistrez la base de données cell0:

`# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova`

-crer le cll1 :

`# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova`

109e1d4b-536a-40d0-83c6-5f121b82b650

-remplir la base de donnée nova 

`# su -s /bin/sh -c "nova-manage db sync" nova`

-Vérifiez que nova cell0 et cell1 sont correctement enregistrés:

`# su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova`

-finaliser l’installation 

`# service nova-api restart

# service nova-consoleauth restart

# service nova-scheduler restart

# service nova-conductor restart

# service nova-novncproxy restart`



# **nova compute**_

installation et configuration :

-installer le paquet :

`# apt-get install nova-compute`

-éditer le fichier /etc/nova/nova.conf comme suit

`[DEFAULT]

# ...

transport_url = rabbit://openstack:mind@controller

[api]

# ...

auth_strategy = keystone



[keystone_authtoken]

# ...

auth_url = http://controller:5000/v3

memcached_servers = controller:11211

auth_type = password

project_domain_name = default

user_domain_name = default

project_name = service

username = nova

password = mind

[DEFAULT]

# ...

my_ip = 192.168.1.251

[DEFAULT]

# ...

use_neutron = true

firewall_driver = nova.virt.firewall.NoopFirewallDriver

[vnc]

# ...

enabled = true

server_listen = 0.0.0.0

server_proxyclient_address = $my_ip

novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]

# ...

api_servers = http://controller:9292



[oslo_concurrency]

# ...

lock_path = /var/lib/nova/tmp

[placement]

# ...

region_name = RegionOne

project_domain_name = Default

project_name = service

auth_type = password

user_domain_name = Default

auth_url = http://controller:5000/v3

username = placement

password = mind`



-Déterminez si votre nœud de traitement prend en charge l'accélération matérielle pour les machines virtuelles:


`$egrep -c (vmx|svm) /proc/cpuinfo`

-Si cette commande renvoie une valeur supérieure ou égale à 1, votre nœud de calcul prend en charge l'accélération matérielle, 
qui ne nécessite généralement aucune configuration supplémentaire.



Si cette commande renvoie la valeur zéro, votre noeud de traitement ne prend pas en charge l'accélération matérielle et vous 
devez configurer libvirt pour qu'il utilise QEMU au lieu de KVM.



Modifiez la section [libvirt] du fichier /etc/nova/nova-compute.conf comme suit:

`[libvirt]

# ...

virt_type = qemu`

- restart le service compute 

`# service nova-compute restart`

`$ . admin-openrc`

-puis confirmez qu'il y a des hôtes de compute dans la base de données:

`$ openstack compute service list --service nova-compute`

- Découvrez les hôtes de compute

`# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`



-editer le fichier `/etc/nova/nova.conf`:

`[scheduler]

discover_hosts_in_cells_interval = 300`
*vérification :



`$ . admin-openrc`

-Répertoriez les composants de service pour vérifier le lancement et l'enregistrement de chaque processus

`$ openstack compute service list`

-les services installés

`$ openstack catalog list`

-les images téléchargée 

`$ openstack image list`

- Vérifiez que les cellules et l'API de positionnement fonctionnent correctement:

`# nova-status upgrade check`

# 5)service neutron**_ 

_**controller node**_

*crier base de donnée neutron 

`# mysql`

`MariaDB [(none)]>CREATE DATABASE neutron;`

*donnée le privilège :

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \

IDENTIFIED BY 'mind';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \

IDENTIFIED BY 'mind';`

`$ . admin-openrc`

*création de neutron user :

`$ openstack user create --domain default --password-prompt neutron`

*Ajoutez le rôle admin à l'utilisateur neutron:

`$ openstack role add --project service --user neutron admin`

*création de service neutron :

`$ openstack service create --name neutron \

--description "OpenStack Networking" network`




*Créer les points de terminaison de l'API de service de mise en réseau

`$ openstack endpoint create --region RegionOne \

network public http://controller:9696

$ openstack endpoint create --region RegionOne \

network internal http://controller:9696

$ openstack endpoint create --region RegionOne \

network admin http://controller:9696`

pour option self-service :

*installer les composons

`# apt-get install neutron-server neutron-plugin-ml2 \

neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \

neutron-metadata-agent`

*editer le fichier `/etc/neutron/neutron.conf` comme suit

`[database]

# ...

connection = mysql+pymysql://neutron:mind@controller/neutron

[DEFAULT]

# ...

core_plugin = ml2

service_plugins = router

allow_overlapping_ips = true

[DEFAULT]

# ...

transport_url = rabbit://openstack:mind@controller

[DEFAULT]

# ...

auth_strategy = keystone



[keystone_authtoken]

# ...

www_authenticate_uri = http://controller:5000

auth_url = http://controller:5000

memcached_servers = controller:11211

auth_type = password

project_domain_name = default

user_domain_name = default

project_name = service

username = neutron

password = mind

[DEFAULT]

# ...

notify_nova_on_port_status_changes = true

notify_nova_on_port_data_changes = true



[nova]

# ...

auth_url = http://controller:5000

auth_type = password

project_domain_name = default

user_domain_name = default

region_name = RegionOne

project_name = service

username = nova

password = mind
[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp`
*éditer le fichier `/etc/neutron/plugins/ml2/ml2_conf.ini` :
`[ml2]
# ...
type_drivers = flat,vlan,vxlan
`[ml2]
# ...
tenant_network_types = vxlan
[ml2]
# ...
mechanism_drivers = linuxbridge,l2population
`[ml2]
# ...
extension_drivers = port_security
[ml2_type_flat]
# ...
flat_networks = provider
[ml2_type_vxlan]
# ...
vni_ranges = 1:1000
[securitygroup]
# ...
enable_ipset = true`
-éditer le fichier`/etc/neutron/plugins/ml2/linuxbridge_agent.ini` comme suit :
`[linux_bridge]
physical_interface_mappings = provider:eno1
[vxlan]
enable_vxlan = true
local_ip = 192.168.1.251
l2_population = true
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver`
*Assurez-vous que le noyau de votre système d'exploitation Linux prend en charge les filtres de pont réseau en vérifiant que toutes les valeurs sysctl suivantes sont définies sur 1:
`systcl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables`
éditer  `/etc/neutron/l3_agent.ini` :
`[DEFAULT]
# ...
interface_driver = linuxbridge`
*éditer le fichier `/etc/neutron/dhcp_agent.ini` comme suit
`[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true `
-éditer le fichier `/etc/neutron/metadata_agent.ini`
`[DEFAULT]
# ...
nova_metadata_host = controller
metadata_proxy_shared_secret = mind`
*éditer le fichier `/etc/nova/nova.conf` comme suit :
`[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = mind
service_metadata_proxy = true
metadata_proxy_shared_secret = mind`
*finaliser l’installation
  remplir la base de donnée
`# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron`
  redémarrer nova api
  `# service nova-api restart`
redémarrer le service reseau
`# service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart`
  -restart le l3_agent
`# service neutron-l3-agent restart`
_**compute node**_
`*installer paquet` 
`# apt-``get` `install neutron-linuxbridge-agent`
editer le fichier `/etc/neutron/neutron.conf` comme suit :
`[DEFAULT]
# ...
transport_url = rabbit://openstack:mind@controller
[DEFAULT]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = mind`

pour option self-service:
*éditer le fichier `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` come suit
`[linux_bridge]
physical_interface_mappings = provider:eno1
[vxlan]
enable_vxlan = true
local_ip = 192.168.1.251
l2_population = true
[securitygroup]
# ...
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver`
*Assurez-vous que le noyau de votre système d'exploitation Linux prend en charge les filtres de pont réseau en vérifiant que toutes les valeurs sysctl suivantes sont définies sur 1:
`sysctl net.bridge.bridge-nf-call-iptables`
`sysctl net.bridge.bridge-nf-call-ip6tables`
*******éditer** le fichier `/etc/nova/nova.conf` comme suit:
`[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = mind`
*finaliser l’installation :
 -restart le service compute
`# service nova-compute restart`
 -restart l’agent bidge 
`# service neutron-linuxbridge-agent restart`

*vérification :
-lister les agents pour vérifier le lancement réussi des agents neutroniques:
`$ . admin-openrc
$ openstack extension list --network

$ openstack network agent list`
 
# 6)service horizon :
*installer le paquet :
` # apt-get install openstack-dashboard`
*éditer le fichier `/etc/openstack-dashboard/local_settings.py` comme suit :
`OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ‘*’
SESSION_ENGINE = 'django.contrib.sessions.backends.cache

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "admin"
TIME_ZONE = "Africa/Tunis"`
*ajouter la ligne suivant dans le fichier /etc/apache2/conf-available/openstack-dashboard.conf s’il n’existe pas :
`WSGIApplicationGroup %{GLOBAL}`
 -finaliser l’installation
`# service apache2 reload`
-vérification:
`[http://controller/horizon](http://controller/horizon)`
→ Authentifiez-vous à l'aide des informations d'identification 
admin,demo,default

#**7)service cinder**_ 
création de la base de donnée cinder
`# mysql
MariaDB [(none)]> CREATE DATABASE cinder;`
*donnée le privilège  
`MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'mind';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'mind';`
`exit`
`$ . admin-openrc`
*création d’utilisateur cinder :
`$ openstack user create --domain default --password-prompt cinder`
*Ajoutez le rôle admin à l'utilisateur cinder
`$ openstack role add --project service --user cinder admin`
-Créez les entités de service cinderv2 et cinderv3:
`$ openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2`
`$ openstack service create --name cinderv3 \
  --description "OpenStack Block Storage" volumev3`
`$ openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev3 public http://controller:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev3 internal http://controller:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev3 admin http://controller:8776/v3/%\(project_id\)s`
*installer et configurer les composants :
 -installer le paquet:
`# apt-get install cinder-api cinder-scheduler`
*éditer le fichier `/etc/cinder/cinder.conf` comme suit :
`[database]
# ...
connection = mysql+pymysql://cinder:mind@controller/cinder
[DEFAULT]
# ...
transport_url = rabbit://openstack:mind@controller
[DEFAULT]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = mind
[DEFAULT]
# ...
my_ip = 192.168.1.251
[oslo_concurrency]
# ...
lock_path = /var/lib/cinder/tmp`
-remplir la base :
`# su -s /bin/sh -c "cinder-manage db sync" cinder`
-éditer le fichier `/etc/nova/nova.conf` comme suit:
`[cinder]
os_region_name = RegionOne`
 -redémarrer le service nova 
`# service nova-api restart`
-redémarrer le service de stockage :
`# service cinder-scheduler restart
# service apache2 restart`

