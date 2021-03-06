.. sectionauthor:: Chris Reynolds <creynold@redhat.com>
.. _docs admin: creynold@redhat.com

=================================
Load-balancing your clusters
=================================

In exercise we're going to deploy an :nginx:`nginx<>` reverse proxy and load balancer. This proxy will take incoming http requests over port 8081
and forward them to one of the 4 web servers that we have deployed. Like most of your other services in today's workshop, you'll be containerizing nginx.

The first

Adding a load balancer inventory group
---------------------------------------

Like your other services, your load balancer needs a group in your Ansible inventory at ``~/playbook/hosts``. Add a group named ``lb`` with the IP address of your control node.

.. parsed-literal::
  [gogs]
  |control_public_ip|

  [registry]
  |control_public_ip|

  [dev]
  |node_1_ip|
  |node_2_ip|

  [prod]
  |node_3_ip|
  |node_4_ip|

  [lb]
  |control_public_ip|

With your inventory group created, it's time to move on to customizing your nginx configuration and building a custom container image.

Creating the nginx container image
-----------------------------------

Nginx has a built-in module that acts as a reverse proxy load balancer named ``proxy_pass``. You'll need to configure that module as well as an ``upstream`` server group for both dev and prod in your ``nginx.conf`` configuration file at ``/etc/nginx/conf.d/default.conf``.

Setting up nginx.conf
```````````````````````

The first step to create your customized nginx container is to create the proper configuration file.

Create a directory named ``~/playbook/nginx-lb`` on your control node. Inside that directoy create a file named ``default.conf`` with the following contents:

.. parsed-literal::

  upstream prod {
    server |node_1_ip|:8080;
    server |node_2_ip|:8080;
  }

  upstream dev {
    server |node_3_ip|:8080;
    server |node_4_ip|:8080;
  }

  server {
      listen 80;
      location /dev {
        proxy_pass \http://dev;
      }
      location /prod {
        proxy_pass \http://prod;
      }
  }

This configuration creates an ``nginx`` loadbalancer that passes requests on |control_public_ip| for ``/dev`` between your dev nodes and requests for ``/prod`` to your production nodes.

Next, create a :dockerfile:`Dockerfile<>` to build an ``nginx`` container image that includes the custom configuration.

Creating the nginx Dockerfile
``````````````````````````````

Create a file at ``~/plabybook/nginx-lb/Dockerfile with the following contents. *Note: The capitalization of ``Dockerfile`` is important*

.. parsed-literal::

  FROM nginx
  USER root
  MAINTAINER |student_name|
  RUN rm /etc/nginx/conf.d/default.conf
  COPY nginx.conf /etc/nginx/conf.d/default.conf

With all of the artifacts created, next you'll write Ansible playbooks to build the container image and deploy your nginx load balancer on your control node.

Creating the nginx container image
``````````````````````````````````
Similar to your development website, you'll create a playbook to build and push your container image and a second playbook to deploy it.

  .. code-block:: bash

    $ cd ~/playbook
    $ vim nginx-lb-build.yml

The load balancer build playbook should have the following content:

.. parsed-literal::

  ---
  - name: Ensure apache is installed and started via role
    hosts: localhost
    become: yes
    roles:
      - apache-simple

    tasks:

     - name: build a new docker image
       command: "docker build -t nginx-lb /home/|student_name|/playbook/nginx-lb"

     - name: Tag and push to registry
       docker_image:
         name: apache-simple
         repository: |control_public_ip|:5000/|student_name|/nginx-lb
         push: yes
         source: local
         tag: latest

To build your container image and push it to your registery, run the playbook using the ``ansible-playbook`` command. Be sure to save the commit your new code to GOGS as well.

.. code-block:: bash

  $ git add -A
  $ git commit -a -m "adding load balancer code"
  $ git push origin master
  $ cd ~/playbook
  $ ansible-playbook -i hosts nginx-lb-build.yml

With the successful completion of this playbook run, your container image is now available in your container registry. Let's deploy it on the proper nodes with another playbook.

Deploying your nginx load balancer
-----------------------------------

Create a playbook named ``~/playbook/nginx-lb-deploy.yml`` with the following content.

.. parsed-literal::

  ---
  - name: deploy nginx load balancer
    hosts: lb
    become: yes

    tasks:
      - name: launch nginx-lb container on lb nodes
        docker_container:
          name: nginx-lb
          image: |control_public_ip|:5000/|student_name|/nginx-lb
          ports:
            - "8082:80"
          restart_policy: always

Run the playbook on your control node using ``ansible-playbook``.

.. code-block:: bash

  $ ansible-playbook -i hosts nginx-lb-deploy.yml

After a successful completion, confirm your load balancer is deployed by testing both dev and prod endpoints.

.. parsed-literal::

  $ curl \http://|control_public_ip|:8082/dev
  $ curl \http://|control_public_ip|:8082/prod

Summary
--------

This lab is the completion of your infrastructure. In the next lab you'll take what you've created and make it work from inside Ansible Tower. Ansible Tower gives you an API in front of your Ansible code so you can interact with it to control your infrastructure from other services like your CI/CD tooling or even your help desk service.
