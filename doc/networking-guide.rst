.. _networking-guide:

==========================
Enabling Provider Networks
==========================
Provider networks allow to connect compute instances directly to physical
networks avoiding tunnels. This is necessary for example for some performance
critical applications. Only administrators of OpenStack can create such
networks. For provider networks compute hosts must have external bridge
created and configured by Ansible tasks like it is already done for tenant
DVR mode networking. Normal tenant non-DVR networking does not need external
bridge on compute hosts and therefore operators don't need additional
dedicated network interface.

To enable provider networks modify the configuration
file ``/etc/kolla/globals.yml``:

::

    enable_neutron_provider_networks: "yes"

===========================
Enabling Neutron Extensions
===========================

Overview
========
Kolla deploys Neutron by default as OpenStack networking component. This guide
describes configuring and running Neutron extensions like LBaaS,
Networking-SFC, QoS, etc.

Networking-SFC
==============

Preparation and deployment
--------------------------

Modify the configuration file ``/etc/kolla/globals.yml`` and change
the following:

::

    enable_neutron_sfc: "yes"

Networking-SFC is an additional Neutron plugin. For SFC to work, this plugin
has to be installed in ``neutron-server`` container as well. Modify the
configuration file ``/etc/kolla/kolla-build.conf`` and add the following
contents:

::

    [neutron-server-plugin-networking-sfc]
    type = git
    location = https://github.com/openstack/networking-sfc.git
    reference = mitaka

Verification
------------

Verify the build and deploy operation of Networking-SFC container. Successful
deployment will bring up an SFC container in the list of running containers.
Run the following command to login into the ``neutron-server`` container:

::

    docker exec -it neutron_server bash

Neutron should provide the following CLI extensions.

::

    #neutron help|grep port

    port-chain-create                 [port_chain] Create a Port Chain.
    port-chain-delete                 [port_chain] Delete a given Port Chain.
    port-chain-list                   [port_chain] List Port Chains that belong
                                      to a given tenant.
    port-chain-show                   [port_chain] Show information of a
                                      given Port Chain.
    port-chain-update                 [port_chain] Update Port Chain's
                                      information.
    port-pair-create                  [port_pair] Create a Port Pair.
    port-pair-delete                  [port_pair] Delete a given Port Pair.
    port-pair-group-create            [port_pair_group] Create a Port Pair
                                      Group.
    port-pair-group-delete            [port_pair_group] Delete a given
                                      Port Pair Group.
    port-pair-group-list              [port_pair_group] List Port Pair Groups
                                      that belongs to a given tenant.
    port-pair-group-show              [port_pair_group] Show information of a
                                      given Port Pair Group.
    port-pair-group-update            [port_pair_group] Update Port Pair
                                      Group's information.
    port-pair-list                    [port_pair] List Port Pairs that belongs
                                      to a given tenant.
    port-pair-show                    [port_pair] Show information of a given
                                      Port Pair.
    port-pair-update                  [port_pair] Update Port Pair's
                                      information.

For setting up a testbed environment and creating a port chain, please refer
to the following link:

    https://wiki.openstack.org/wiki/Neutron/ServiceInsertionAndChaining

For the source code, please refer to the following link:

    https://github.com/openstack/networking-sfc


Neutron VPNaaS (VPN-as-a-Service)
=================================

Preparation and deployment
--------------------------

Modify the configuration file ``/etc/kolla/globals.yml`` and change
the following:

::

    enable_neutron_vpnaas: "yes"

Verification
------------

VPNaaS is a complex subject, hence this document provides directions for a
simple smoke test to verify the service is up and running.

On the network node(s), the ``neutron_vpnaas_agent`` should be up (image naming
and versioning may differ depending on deploy configuration):

::

    docker ps --filter name=neutron_vpnaas_agent
    CONTAINER ID        IMAGE
    COMMAND             CREATED             STATUS              PORTS
    NAMES
    97d25657d55e
    operator:5000/kolla/oraclelinux-source-neutron-vpnaas-agent:4.0.0
    "kolla_start"       44 minutes ago      Up 44 minutes
    neutron_vpnaas_agent

Kolla-Ansible includes a small script that can be used in tandem with
``tools/init-runonce`` to verify the VPN using two routers and two Nova VMs:

::

    tools/init-runonce
    tools/init-vpn

Verify both VPN services are active:

::

    neutron vpn-service-list
    +--------------------------------------+----------+--------------------------------------+--------+
    | id                                   | name     | router_id                            | status |
    +--------------------------------------+----------+--------------------------------------+--------+
    | ad941ec4-5f3d-4a30-aae2-1ab3f4347eb1 | vpn_west | 051f7ce3-4301-43cc-bfbd-7ffd59af539e | ACTIVE |
    | edce15db-696f-46d8-9bad-03d087f1f682 | vpn_east | 058842e0-1d01-4230-af8d-0ba6d0da8b1f | ACTIVE |
    +--------------------------------------+----------+--------------------------------------+--------+

Two VMs can now be booted, one on vpn_east, the other on vpn_west, and
encrypted ping packets observed being sent from one to the other.

For more information on this and VPNaaS in Neutron refer to the VPNaaS area on
the OpenStack wiki:

    https://wiki.openstack.org/wiki/Neutron/VPNaaS/HowToInstall
    https://wiki.openstack.org/wiki/Neutron/VPNaaS

Networking-ODL
==============

Preparation and deployment
--------------------------

Modify the configuration file ``/etc/kolla/globals.yml`` and enable
the following:

::

    enable_opendaylight: "yes"

Networking-ODL is an additional Neutron plugin that allows the OpenDaylight SDN Controller
to utilize its networking virtualization features. For OpenDaylight to work, the
Networking-ODL plugin has to be installed in the ``neutron-server`` container. In this case,
one could use the neutron-server-opendaylight container and the opendaylight container by pulling
from Kolla dockerhub or by building them locally.

Further OpenDaylight globals.yml configurable options with their defaults include:
::

    opendaylight_release: "0.6.1-Carbon"
    opendaylight_mechanism_driver: "opendaylight_v2"
    opendaylight_l3_service_plugin: "odl-router_v2"
    opendaylight_acl_impl: "learn"
    enable_opendaylight_qos: "no"
    enable_opendaylight_l3: "yes"
    enable_opendaylight_legacy_netvirt_conntrack: "no"
    opendaylight_port_binding_type: "pseudo-agentdb-binding"
    opendaylight_features: "odl-mdsal-apidocs,odl-netvirt-openstack"
    opendaylight_allowed_network_types: '"flat", "vlan", "vxlan"'

Clustered OpenDaylight Deploy
-----------------------------
High availability clustered OpenDaylight requires modifying the inventory file and placing
three or more hosts in the OpenDaylight or Networking groups. Note: The OpenDaylight role
will allow deploy of one or three plus hosts for OpenDaylight/Networking role.

Verification
------------

Verify the build and deploy operation of Networking-ODL containers. Successful
deployment will bring up an Opendaylight container in the list of running containers on
network/opendaylight node.

For the source code, please refer to the following link:

    https://github.com/openstack/networking-odl
