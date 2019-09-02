.. sectionauthor:: Ajay Chenampara <achenamp@redhat.com>
.. _docs admin: jduncan@redhat.com

==================================================
Configuring Ansible Tower
==================================================

Overview
---------
In this lab will you'll be working with Ansible Tower to make it how we interface with our playbooks, roles, and infrastructure for the rest of the workshop. We'll configure Tower with your inventory, credentials, and tell it how to interface with GOGS to manage playbooks and roles.

Your control node already has Tower deployed at \https://|control_public_ip|. We've also pre-applied a license key to your tower instance, so it's ready to be configured. Your Tower username is ``admin`` and your Tower password is |student_pass|.

Configuring Ansible Tower
--------------------------

There are a number of constructs in the Ansible Tower UI that enable multi-tenancy, notifications, scheduling, etc. Today we're going to focus on a few of the key constructs that are essential to any workflow.

-  Credentials
-  Projects
-  Inventory
-  Job Template

Let's start with adding a Credential.

Creating a Credential
^^^^^^^^^^^^^^^^^^^^^^

Credentials are utilized by Tower for authentication when launching jobs against machines, synchronizing with inventory sources, and importing project content from a version control system.

There are many `types of credentials <http://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html#credential-types>`__ including machine, network, and various cloud providers. In this workshop, we'll create a *machine* credential.

- Select the gear icon |Gear button|, then select CREDENTIALS.
- Click on ADD |Add button|


Use this information to complete the credential form.

+------------------------+---------------------------------------+
| NAME                   | Ansible Workshop Credential           |
+========================+=======================================+
| DESCRIPTION            | Credentials for Ansible Workshop      |
+------------------------+---------------------------------------+
| ORGANIZATION           | Default                               |
+------------------------+---------------------------------------+
| TYPE                   | Machine                               |
+------------------------+---------------------------------------+
| USERNAME               | |student_name|                        |
+------------------------+---------------------------------------+
| PASSWORD               | |student_pass|                        |
+------------------------+---------------------------------------+
| PRIVILEGE ESCALATION   | Sudo (This is the default)            |
+------------------------+---------------------------------------+

.. figure:: ./_static/images/at_cred_detail.png
   :alt: Adding a Credential

   Adding a Credential

- Select SAVE |Save button|

With your credential created, next you'll create a project to point back to your GOGS instance.

Creating a Project
^^^^^^^^^^^^^^^^^^^

A Project is a logical collection of Ansible playbooks, represented in Tower. You can manage playbooks and playbook directories by either placing them manually under the Project Base Path on your Tower server, or by placing your playbooks into a source code management (SCM) system supported by Tower, including Git, Subversion, and Mercurial.

- Click on PROJECTS
- Select ADD |Add button|

Complete the form using the following entries

================== ===================================================
NAME               Ansible Workshop Project
================== ===================================================
DESCRIPTION        workshop playbooks
ORGANIZATION       Default
SCM TYPE           Git
SCM URL            \http://|control_public_ip|/|student_name|/playbook.git
SCM BRANCH
SCM UPDATE OPTIONS [x] Clean [x] Delete on Update [x] Update on Launch
================== ===================================================

.. figure:: ./_static/images/at_project_detail.png
   :alt: Defining a Project

   Defining a Project

- Select SAVE |Save button|

Creating an Inventory
^^^^^^^^^^^^^^^^^^^^^^

An inventory is a collection of hosts against which jobs may be launched. Inventories are divided into groups and these groups contain the actual hosts. Groups may be sourced manually, by entering host names into Tower, or from one of Ansible Tower’s supported cloud providers.
An Inventory can also be imported into Tower using the ``tower-manage`` command and this is how we are going to add an inventory for this workshop.

- Click on INVENTORIES
- Select ADD |Add button|
- Complete the form using the following entries

+----------------+------------------------------+
| NAME           | Ansible Workshop Inventory   |
+================+==============================+
| DESCRIPTION    | Ansible Inventory            |
+----------------+------------------------------+
| ORGANIZATION   | Default                      |
+----------------+------------------------------+

.. figure:: ./_static/images/at_inv_create.png
   :alt: Create an Inventory

   Creating an Inventory

- Select SAVE |Save button|

Look in your ``.ansible.cfg`` file to find the path to your default inventory. This is the inventory we'll import into Tower. Your default inventory is the ``inventory`` parameter.

.. parsed-literal::

  $ cat ~/.ansible.cfg
  [defaults]
  stdout_callback = yaml
  connection = smart
  timeout = 60
  deprecation_warnings = False
  host_key_checking = False
  retry_files_enabled = False

  inventory = /home/|student_name|/devops-workshop/lab_inventory/hosts

To import the inventory, we'll use the ``tower-manage`` utility on your control node/Tower server.

.. parsed-literal::

    sudo tower-manage inventory_import --source=/home/|student_name|/devops-workshop/lab_inventory/hosts --inventory-name="Ansible Workshop Inventory"

You should see output similar to the following:

.. figure:: ./_static/images/at_tm_stdout.png
   :alt: Importing an inventory with tower-manage

   Importing an inventory with tower-manage

Feel free to browse your inventory in Tower. You should now notice that
the inventory has been populated with Groups and that each of those
groups contain hosts.

.. figure:: ./_static/images/at_inv_group.png
   :alt: Inventory with Groups

   Inventory with Groups

Ansible Tower is now configured with everything we need to continue building out our infrastructure-as-code environment in today's workshop!

Configure Ansible Tower
```````````````````````````````````````


Set Up Credentials
^^^^^^^^^^^^^^^^^^^

Gogs and student machine Credentials

Add a Project
^^^^^^^^^^^^^

Point Tower to the Gogs server


Create a Stig template
^^^^^^^^^^^^^^^^^^^^^^^


Create a Site A template
^^^^^^^^^^^^^^^^^^^^^^^^

Create a Site B template
^^^^^^^^^^^^^^^^^^^^^^^^

Create a NGINX template
^^^^^^^^^^^^^^^^^^^^^^^^


Create a Workflow
^^^^^^^^^^^^^^^^^


Create a STIG Check template
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
s

.. |Browse button| image:: ./_static/images/at_browse.png
.. |Submit button| image:: ./_static/images/at_submit.png
.. |Gear button| image:: ./_static/images/at_gear.png
.. |Add button| image:: ./_static/images/at_add.png
.. |Save button| image:: ./_static/images/at_save.png