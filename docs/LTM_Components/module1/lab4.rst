Lab 4: Configure High-Availability
----------------------------------

In this lab you will set up a high availability pair using two BIG-IP
systems. You’ll then test failover between the two HA members.

Task 1 – Configure Device Trust and a Device Group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows 10 Jump Host open up two browser tabs and log into both the **BIGIP01** bookmark and the **BIGIP02** system.

#. From the Navigation pane, expand the **Device Management** menu. 

#. Select the **Device Trust** pane, then click on the **Device Trust Members**

#. On **BIGIP01** click Add, and enter the following information


   +----------------+--------------------------------+
   | Form field     | Value                          |
   +================+================================+
   | Device Type    | Peer                           |
   +----------------+--------------------------------+
   | Device IP      | 10.1.1.6                       |
   | Address        |                                |
   +----------------+--------------------------------+
   | Administrator  | admin                          |
   | Username       |                                |
   +----------------+--------------------------------+
   | Administrator  | admin.F5demo.com               |
   | Password       |                                |
   +----------------+--------------------------------+
   
   
#. Click on **Retrieve Device Information**

n **BIGIP02** click Add, and enter the following information


   +----------------+--------------------------------+
   | **Form field** | **Value**                      |
   +==============+==================================+
   | Device Type    | Peer                           |
   +----------------+--------------------------------+
   | Device IP      | 10.1.1.4                       |
   | Address        |                                |
   +----------------+--------------------------------+
   | Administrator  | admin                          |
   | Username       |                                |
   +----------------+--------------------------------+
   | Administrator  | admin.F5demo.com               |
   | Password       |                                |
   +----------------+--------------------------------+
   
   
#. Click on **Retrieve Device Information**
   
#. Note the status of both BIG-IP systems. Each BIG-IP system should trust each other 
   through a public/private encryption key exchange.  Note the new status of both BIG-IP systems.


Task 2 – Configure a **Sync-Failover** Group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


#. From the Navigation Pane, expand **Device Management, Device Groups** and click
   **Create**. (ENSURE you are on **BIGIP01**.)

#. Create a device group using the following information, and then click
   **Finished**.

   +--------------+--------------------------------+
   | Form field   | Value                          |
   +==============+================================+
   | Name         | syncfailover                   |
   +--------------+--------------------------------+
   | Group Type   | Sync-Failover                  |
   +--------------+--------------------------------+
   | Members      | bigip01.f5demo.com             |
   |              | bigip02.f5demo.com             |
   +--------------+--------------------------------+
   | Sync Type    | Manual with Incremental Sync   |
   +--------------+--------------------------------+
   
#. Click Finished

#. From the Navigation Pane under **Device Management** click on **Traffic Groups**

#. Within the **Failover Order** section select both **BIGIP01** and **BIGIP02** and click the
   left arrow button.   Both BIGIP objects should now be in the **Preferred Order** section with
   **BIGIP01** listed above **BIGIP02**.   Click Save.

#. Note the new status of **bigip01.f5demo.com**.

#. Click **Awaiting Initial Sync**.

   This directs you to the **Device Management > Overview** page.

#. In the **Devices** section, leave the **bigip01.f5demo.com (Self)**
   option selected.

#. For **Sync Options** leave **Push the selected device configuration
   to the group** selected and click **Sync**.

#. Note the status of **bigip01.f5demo.com**.

   Both BIG-IP systems should no be in sync with each other.

**On bigip02.f5demo.com**

#. Attempt to log in as **admin** / **admin**.

   Why do you think your login failed?

#. Log in as **admin** / **admin.F5demo.com**.

#. Examine the **Virtual Server List** and **Pool List** pages.

#. On BIGIP01 from the Navigation pane click on Local Traffic, Virtual Servers, Virtual Server List

#. Click on the **LAMP** Virtual Server, and change the **HTTP Profile (Client) to **http**

#. Click on **Resources** 

#. Change the **Default Persistence Profile** to **cookie** and then click on **Update**

#. Note the status message of **Changes Pending** in the top left corner of BIGIP01**

#. Click **Changes Pending**.

#. In the **Devices** section leave the default options selected and
   click **Sync**.

#. On **BIGIP02** examine the **LAMP** Virtual Server

Task 3 – Traffic Groups

A traffic-group is a collection of related configuration objects which together, process a particular type of traffic.
These include  Self IPs, Virtual IPs, iApps, SNATs and VLANs objects.  A Local Traffic group – is local significant only;
it has no scope beyond the device where it’s configured. A default local traffic group is created automatically during 
the initial setup wizard called traffic-group-local-only – it contains the non-floating self IPs associated to the external
and internal VLANs.

There are also floating traffic groups, and as the name suggests, these can be automatically reassigned (float) between
devices part of the same device-group. Objects which belong to a floating traffic group are called failover objects

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the **Navigation Pane*, on BIGIP01, click on Network, Self-IP's, and create the following Floating Self-IP addresses:

   +--------------+-------------------------------------------+
   | Form field   | Value                                     |
   +==============+===========================================+
   | Name         | external_selfip_floating                  |
   +--------------+-------------------------------------------+
   | IP Address   | 10.1.10.248                               |
   +--------------+-------------------------------------------+
   | Netmask      | 255.255.255.0                             |
   +--------------+-------------------------------------------+
   | Traffic Group| traffic-group-1 (floating)                |
   +--------------+-------------------------------------------+


   +--------------+-------------------------------------------+
   | Form field   | Value                                     |
   +==============+===========================================+
   | Name         | _internal_selfip_floating                 |
   +--------------+-------------------------------------------+
   | IP Address   | 10.1.20.248                               |
   +--------------+-------------------------------------------+
   | Netmask      | 255.255.255.0                             |
   +--------------+-------------------------------------------+
   | Traffic Group| traffic-group-1 (floating)                |
   +--------------+-------------------------------------------+


Task 4 – Test Failover

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**On BIGIP01**

#. From the **Navigation Pane*, on BIGIP01, click on Device Management, Devices

#. Scroll to the bottom and click on **Force Offline**

#. Notice in the top left corner the device status should reflect **FORCED OFFLINE**


**On BIGIP02**

#. Note the status of **BIGIP02**.

#. BIGIP02 status as displayed in the top left corner of the GUI should reflect **ONLINE ACTIVE**, (In Sync)

#. Verify that the Self IP objects are now active on BIGIP02 by navigating to the Network menu object and clicking on
   the Self IP objects.
   
#. Virtual Servers, Pools, Pool members and any SNAT object should also exist on BIGIP02 which is now the active BIG-IP device

**On BIGIP01**

#. On the **Devices** page click **Release Offline** and then **OK**.

**On BIGIP01**

#. Note the status of **BIGIP01**.

This concludes Lab 4


   |image19|



.. |image17| image:: /_static/class1/image19.png
   :width: 1.70088in
   :height: 0.61232in
.. |image18| image:: /_static/class1/image20.png
   :width: 1.70088in
   :height: 0.60540in
.. |image19| image:: /_static/class1/image21.png
   :width: 3.98717in
   :height: 1.04839in
