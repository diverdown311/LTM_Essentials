Class 1: Introduction to the F5 LTM Essentials Lab
========================================================

Welcome to the F5 BIG-IP LTM Essentials lab session.
This series of lab is intended to guide you through the basic components
of the F5 BIG-IP Local Traffic Manager (LTM) module. This guide is
intended to complement lecture material provided during the lab.
The first section introduces students to the following BIG-IP concepts:

-  Pools
-  Members
-  Nodes
-  Virtual Servers
-  SNAT's
-  SSL Offload
-  Monitors
-  High-Availability

Additionally, this course also introduces basic troubleshooting concepts and hands on labs.

The lab environment consists of the following components:

#. bigip01.f5demo.com   Management IP Address:  10.1.1.4
#. bigip02.f5demo.com   Management IP Address:  10.1.1.6
#. Ubuntu_LAMP1         Internal IP Address:    10.1.20.252
#. Ubuntu_LAMP2         Internal IP Address:    10.1.20.250
#. Windows 10 Jumphost  IP Address:             10.1.10.190

It is recommended that all lab work be completed via the Windows 10 Jumphost; however, it is entirely possible to access each of the BIG-IP systems via
TMUI interface and perform configuration work on each of the BIG-IP appliances.    



.. toctree::
   :maxdepth: 1
   :glob:

   labinfo
   module*/module*

