.. DIBBS documentation master file, created by
   sphinx-quickstart on Wed Sep 14 09:26:21 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to DIBBS's documentation!
=================================

.. toctree::
   :maxdepth: 2

Introduction
============

The DIBBS project provides a set of independent projects which, when they are
collaborating, are constituting a platform for publishing operations. These
projects require a python 2.7 environment to be able to run, and their
dependencies are meant to be installed via the *pip* package manager.

Installation
============

Via docker
----------

In order to ease the deployment of a DIBBS platform, we provide a set of scripts
that instantiate all the software components inside dedicated Docker containers
and perform their initialization.

First, clone the project that contains the set of initialization scripts::

    git clone https://github.com/DIBBS-project/DIBBS-Architecture-Demo.git

To deploy and run a DIBBS platform, execute the *docker_reload.sh* script, which
enables to deploy each DIBBS sub-services in a dedicated Docker container. On
systems based on Debian and RHEL, *docker_reload.sh* will try to install Docker
leveraging *yum* or *apt*. On OSX, the script assumes Docker that Docker is
already installed.

.. note::
    Installation of Docker on a recent OSX system (>10.10.3) is very simple with
    the official application. More details can be found at the official docker
    `installation guide for OSX
    <https://docs.docker.com/engine/installation/mac/>`__


Run the *docker_reload.sh* script with superuser security privileges::

    sudo bash docker_reload.sh

Manual installation
-------------------

In this section we show how to install the projects that are composing the DIBBS
platform. A fully configured DIBBS platform requires the following projects to
be configured:

  * `Central Authentication System <https://github.com/DIBBS-project/central_authentication_service>`__
  * `Architecture Portal <https://github.com/DIBBS-project/architecture_portal>`__
  * `Appliance Registry <https://github.com/DIBBS-project/appliance_registry>`__
  * `Resource Manager <https://github.com/DIBBS-project/resource_manager>`__
  * `Operation Registry <https://github.com/DIBBS-project/operation_registry>`__
  * `Operation Manager <https://github.com/DIBBS-project/operation_manager>`__

These projects are all Django projects and they share the same installation
instructions. In the remaining of this section, detailed instructions will be
given for the **Central Authentication System** only.

First clone the repository containing the source code::

  git clone https://github.com/DIBBS-project/resource_manager.git

Each project contains a *requirements.txt* file that describes what are the
requirements in terms of Python libraries. To install the dependencies, please
run the following command::

  pip install -r requirements.txt

Then from its source folder, run this command to initialize/reset the database::

  bash reset.sh

Lastly, run this command to launch the Django web application::

  python manage.py runserver 0.0.0.0


Getting started
===============

Initializing data
-----------------

First, you will need a user account on an OpenStack infrastructure. In the rest
of this tutorial, we will assume that you have a `ChameleonCloud
<https://www.chameleoncloud.org/>`__ account with the following characteristics:

+----------+-----------------+
| username | "<username>"    |
+----------+-----------------+
| project  | "<projectname>" |
+----------+-----------------+

In order to help users to get started, we explain how to configure the DIBBS
platform to interact with the KVM infrastructure hosted at Chameleon. This
process is automated thanks to the use of scripts publicly available on `Github
<https://github.com/DIBBS-project/DIBBS-Architecture-Demo>`__.

First, create a JSON file called *infrastructure_description.json* with the
following content::

    {
      "credentials": [
        {
          "name": "kvm@tacc_credentials",
          "username": "<username>",
          "project_name": "<projectname>",
          "flavor": "m1.medium",
          "infrastructure": "kvm@tacc"
        }
      ],
      "infrastructures": [
        {
          "name": "kvm@tacc",
          "contact_url": "https://openstack.tacc.chameleoncloud.org:5000/v2.0",
          "type": "openstack"
        }
      ]
    }

Now that the configuration file is created, run the following command to initialize infrastructures and appliances in the DIBBS platform::

    python create_appliances.py infrastructure_description.json

Appliances and their implementation have been created. Now run the following command to upload your OpenStack credentials in the DIBBS platform::

    python create_os_users.py infrastructure_description.json

.. note::
    For each of the credentials specified in the configuration file, the
    *create_os_users.py* script will ask you to enter the corresponding
    password. This password is then encrypted and sent to the DIBBS platform.


First interaction with the REST API
-----------------------------------

Below is a python example of how to interact directly with the Rest API::

    #!/usr/bin/env python
    import requests
    from requests.auth import HTTPBasicAuth

    appliance_registry_url = "http://127.0.0.1:8003"

    # Create an HTTP request that will fetch the existing appliances
    result = requests.get("%s/appliances" % (appliance_registry_url,),
                          auth=HTTPBasicAuth('admin', 'pass'))

    appliances = result.json()
    for appliance in appliances:
        print appliance["name"]

The execution of the preceding block of code should results in::

    common
    hadoop
    hadoop_old
    hadoop_urbanflow
    mongodb

API description
===============

Detailed descriptions of the REST APIs of each components of the DIBBS platform
can be found in the following table:

+--------------------+----------------------------------------------------+
| Project            | Link to the API description                        |
+====================+====================================================+
| Operation Registry | https://dibbs-project.github.io/operation_registry |
+--------------------+----------------------------------------------------+
| Operation Manager  | https://dibbs-project.github.io/operation_manager  |
+--------------------+----------------------------------------------------+
| Appliance Registry | https://dibbs-project.github.io/appliance_registry |
+--------------------+----------------------------------------------------+
| Resource Manager   | https://dibbs-project.github.io/resource_manager   |
+--------------------+----------------------------------------------------+

Using clients
=============

Using the official python clients
---------------------------------

It is possible to interact with the DIBBS environment using the clients provided
by the *common-dibbs* project. The project can be installed via pip:::

    pip install common-dibbs

Below is an example of a python script that interact with a deployed infrastructure:::

    #!/usr/bin/env python
    from common_dibbs.clients.ar_client import AppliancesApi
    from common_dibbs.misc import configure_basic_authentication

    appliance_registry_url = "http://127.0.0.1:8003"

    # Create an API client for Appliances
    appliances_client = AppliancesApi()
    appliances_client.api_client.host = "%s" % (appliance_registry_url,)
    configure_basic_authentication(appliances_client, "admin", "pass")

    appliances = appliances_client.appliances_get()
    for appliance in appliances:
        print appliance.name

The execution of the preceding block of code should results in::

    common
    hadoop
    hadoop_old
    hadoop_urbanflow
    mongodb

List of python clients
----------------------

The following table gives for each of the DIBBS sub projects, its corresponding python package:

+-----------------------+---------------------------------------------+
| Project               | Corresponding Python package                |
+=======================+=============================================+
| Operation Registry    | common_dibbs.clients.or_client              |
+-----------------------+---------------------------------------------+
| Operation Manager     | common_dibbs.clients.om_client              |
+-----------------------+---------------------------------------------+
| Appliance Registry    | common_dibbs.clients.ar_client              |
+-----------------------+---------------------------------------------+
| Resource Manager      | common_dibbs.clients.rm_client              |
+-----------------------+---------------------------------------------+

Generating your own clients
---------------------------

APIs of each sub project is described in YAML files (following the Swagger
format), which can be used to generate API clients in many languages. To get
more details, please take a look at the `swagger-codegen
<https://github.com/swagger-api/swagger-codegen>`__ project.

The following table summarizes where the API description files can be found:

+-----------------------+----------------------------------------------------------------------------------------------------------------+
| Project               | Link to the API description file (Swagger YAML file)                                                           |
+=======================+================================================================================================================+
| Authentication System | https://github.com/DIBBS-project/central_authentication_service/blob/master/casapp/static/swagger/swagger.yaml |
+-----------------------+----------------------------------------------------------------------------------------------------------------+
| Operation Registry    | https://github.com/DIBBS-project/operation_registry/blob/master/orapp/static/swagger/swagger.yaml              |
+-----------------------+----------------------------------------------------------------------------------------------------------------+
| Operation Manager     | https://github.com/DIBBS-project/operation_manager/blob/master/omapp/static/swagger/swagger.yaml               |
+-----------------------+----------------------------------------------------------------------------------------------------------------+
| Appliance Registry    | https://github.com/DIBBS-project/appliance_registry/blob/master/arapp/static/swagger/swagger.yaml              |
+-----------------------+----------------------------------------------------------------------------------------------------------------+
| Resource Manager      | https://github.com/DIBBS-project/resource_manager/blob/master/rmapp/static/swagger/swagger.yaml                |
+-----------------------+----------------------------------------------------------------------------------------------------------------+

Acknowledgment
==============



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

