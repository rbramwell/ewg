eCommerce Sample Heat Template
==============================

These heat templates deploy OpenCart on a 3-Tier LAMP architecture.  There are
two versions of the primary template, one which creates a static environment
which does not require ceilometer, and one which provides autoscaling of the
web and services layers based on CPU load, which does require ceilometer.


**The OpenCart 3-Tier LAMP Architecture Sample**

========  ======================  ====================================
Layer     Function                Details
========  ======================  ====================================
Web       Reverse Proxy Server    Apache + mod_proxy
Services  Application Server      Apache, PHP, MySQL Client, OpenCart
Data      Database Server         MySQL
========  ======================  ==================================== 

-----------------
Heat File Details
-----------------

The template uses a nested structure, with two different primary yaml files,
both of which utilize the same 4 nested files.  The templates were tested
using Mitaka release of OpenStack, and Ubuntu server 14.04 and Centos7.

**eCommerceStatic.yaml:** If you want a static environment, run this yaml
file.  This will create a static environment, with two load balanced web
servers, and two load balanced application servers, and a single database
server using cinder block storage for the database files.  

REQUIRED PARAMETERS:

  * ssh_key_name, image_id, public_network_id

OPTIONAL PARAMETERS:

  * db_instance_flavor, app_instance_flavor, web_instance_flavor,
    db_server_name, db_name,  db_username, db_password, db_root_password,
    app_server_name, web_server_name, admin_username, admin_password,
	admin_email, dns_nameserver
  
**eCommerceAutoScaling.yaml:**  If you want a dynamic autoscaling environment,
run this yaml file.  This yaml files sets up heat autoscaling groups.  

  REQUIRED PARAMETERS:

  * ssh_key_name, image_id, public_network_id

  OPTIONAL PARAMETERS:

  * db_instance_flavor, app_instance_flavor, web_instance_flavor,
    db_server_name, db_name,  db_username, db_password, db_root_password,
    app_server_name, web_server_name, admin_username, admin_password,
	admin_email, dns_nameserver

The following 4 yaml files are called by the primary files above, and are by
default expected to be in a lib subdirectory:

**setup_network.yaml:**
  This file creates 3 separate private networks, one for each tier.  In
  addition it creates two load balancers (using neutron LBaaS V1), one which
  has a public IP that connects the web private network to the public
  network, and one with a private IP that connects the web network to the
  services network.  The template also creates a router connecting the
  services network to the database network.  In addition to the networks and
  routers, the template creates 3 security groups, one for each of the tiers.  

**launch_web_layer.yaml:**
  This template file launches the web layer nodes.  In addition to launching
  instances, it installs and configures Apache and Apache modproxy which is
  used to redirect traffic to the application nodes.

**launch_services_layer.yaml:**
  This template file launches the services layer nodes.  In addition to
  launching the instances, it installs Apache, PHP, MySQL client, and
  OpenCart.  
**launch_sql_layer.yaml:**
  This template file launches the database layer node and installs MySQL.
  In addition it creates a cinder block device to store the database files.
  The template also creates the required users and databases for the OpenCart
  application.

-------------------------------
Running the heat template files
-------------------------------

First you need to source your credential file.  You may download a copy of the
credential file from Horizon under Project>Compute>Access & Security>API
Access

**Example to setup the static environment**

  openstack stack create --template eCommerceStatic.yaml --parameter
  ssh_key_name=mykey --parameter image_id=ubuntu --parameter
  dns_nameserver="8.8.8.8,8.8.4.4" --parameter
  public_network_id=external_network OpenCart

**Example to setup the autoscaling environment**
  openstack stack create --template eCommerceAutoScaling.yaml --parameter
  ssh_key_name=mykey --parameter image_id=centos --parameter
  dns_nameserver="8.8.8.8,8.8.4.4" --parameter
  public_network_id=external_network OpenCart

