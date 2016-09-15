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
collaborating, are constituting an operation publishing platform. These projects
require a python 2.7 environment to be able to run, and their dependencies are
meant to be installed via the *pip* package manager.

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
enables to deploy each DIBBS subservices in a dedicated Docker container. On
systems based on debian and RHEL, *docker_reload.sh* will try to install Docker
leveraging *yum* or *apt*. On OSX, it assumes Docker to be already installed.

.. note::
    Installation of Docker on a recent OSX system (>10.10.3) is very simple with
    the official application. More details can be found at the official docker
    `installation guide for OSX
    <https://docs.docker.com/engine/installation/mac/>`__


Run the *docker_reload.sh* script with superuser security priviledges::

    sudo bash docker_reload.sh

Manual installation
-------------------

In this section we show how to install the projects that are composing the DIBBS
platform. A fully configured DIBBS platform requires the following projects to
be configured:

  * **Central Authentication System**
  * **Architecture Portal**
  * **Appliance Registry**
  * **Resource Manager**
  * **Operation Registry**
  * **Operation Manager**


These projects are all Django projects and they share the same installation
instructions. In the remaining of this section, detailled instructions will be
given for the **Central Authentication System** only.

First clone the repository containing the source code::

  git clone https://github.com/DIBBS-project/resource_manager.git

Then from its source folder, run this command to initialise/reset the database::

  bash reset.sh

Lastly, run this command to launch the Django web application::

  python manage.py runserver 0.0.0.0


Getting started
===============

Initializing data
-----------------

First, you will need to create a user account on an OpenStack infrastructure. In
the rest of this tutorial, we will assume that you have a `ChameleonCloud
<https://www.chameleoncloud.org/>`__ account with the following characteristics:

+----------+-----------------+
| username | "<username>"    |
+----------+-----------------+
| project  | "<projectname>" |
+----------+-----------------+

Create an initialization file based on JSON format as follows::

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

Now that the configuration file is created, run the following command to initialise infrastructures and appliances in the DIBBS platform::

    python create_appliances.py infrastructure_description.json



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

which results in::

    common
    hadoop
    hadoop_old
    hadoop_urbanflow
    mongodb

API description
===============

Detailled descriptions of the REST APIs of each components of the DIBBS
architecture can be found in the following table:

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

which results in::

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

