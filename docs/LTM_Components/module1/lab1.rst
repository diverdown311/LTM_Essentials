Lab 1: Explore the F5 BIG-IP network configuration
------------------------------------------

It is assumed that you have already licensed each of the BIG-IP devices. In this lab you will explore network settings on the BIG-IP.
You will also configure a default route, internal and external vlans, and finally you will configure Self-IP 
Addresses and map them to individual VLAN's.

Task 1 – Connect to the Win10 Jumphost via RDP and log into the BIG-IP01 by clicking on the **ACCESS** drop down menu within 
the lab environment.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows 10 Jumphost using a web browser (Chrome), log into BIG-IP01 (10.1.1.4) system with the below credentials.

-  admin

-  admin.F5demo.com

#. While logged into the BIG-IP01 gui click on the Network menu object, then click on the Interfaces object.   Note that there are two interfaces labeled 1.1 and 1.2

#. Network interface 1.1 is assigned to the external network while interface 1.2 is assigned to the internal network

#. Click on the VLANs object then click on the **Create** button at the top right and enter the name **external**.  Select interface 1.1, click the Tagging drop down menu and ensure the interface is set to **Untagged** then click on the **Add** button.

#. Follow the same process for creating a network interface named **internal** while ensuring interface 1.2 is selected as **Untagged**.

#. Now we will configure two Self-IP Addresses.

#. From within the **Network** menu click on Self IPs object and click **Create**.

#. Click **Create** .

   The **Network** menu is where you configure elements for routing and
   switching.

#. Name the first Self IP object **external_selfip**.

#. Assign the 10.1.10.245 IP Address, with a Netmask of 255.255.255.0

#. Ensure the VLAN/Tunnel is set to **External**, while the Port Lockdown should be configured for **Allow None**.

#. Follow the same process for the **Internal** interface with the following IP Address and Netmask.

   10.1.20.245/255.255.255.0

#. Click Finished after configuring each of the Self IP Addresses

#. The next task is to configure a Network Default Route.

#. Under the Network menu click on the Routes object and click the Add button.

#. Name the Network Default Route object **default_route**

#. The Destination and Netmask should be 0.0.0.0/0 and 0.0.0.0/0

#. Ensure Use Gateway is selected and Gateway Address is set to IP Address.

#. Enter the address 10.1.10.1 and click Finished

Task 2 – Create a Pool BIG-IP object and a Virtual Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will be configuring a Pool with two member objects.  A pool is a group of pool members.   With few exceptions, all the members of a given pool
host the same content.   Pools are named, and like most objects on BIG-IP systems, their names can begin with a letter or underscore as well as numbers but
should not contain spaces.  Pools also have thier own load balancing method, monitors, and other features defined when a pool is created or modified.
When a new connection is initiated to a virtual server that is mapped to a pool, various criteria, including the pool's load balancing method may be used
to determine which member to use for that request.

#. Open the **Local Traffic > Pools > Pool List** page and click
   **Create**.

   |image2|

#. Use the following information for the new pool. For fields that are
   not specified, leave them set to the default settings.

   +---------------+------------------------------------+
   | Form field    | Value                              |
   +===============+====================================+
   | Name          | LAMP1                              |
   +---------------+------------------------------------+
   | New Members   | Node Name: LAMP1                   |
   |               | Address: 10.1.20.252               |
   |               | Service Port: 80 (Click **Add**)   |
   +---------------+------------------------------------+
   | Name          | LAMP2                              |
   +---------------+------------------------------------+
   | New Members   | Node Name: LAMP2                   |
   |               | Address: 10.1.20.250               |
   |               | Service Port: 80 (Click **Add**)   |
   +---------------+------------------------------------+
   
   
#. Click **Finished**.

#. Open the **Local Traffic > Virtual Servers > Virtual Server List**
   page and click **Create**.

#. Use the following information for the new virtual server, and then
   click **Finished**.

   +-----------------------------+-----------------+
   | Form field                  | Value           |
   +=============================+=================+
   | Name                        | LAMP            |
   +-----------------------------+-----------------+
   | Destination Address/ Mask   | 10.1.10.200     |
   +-----------------------------+-----------------+
   | Service Port                | 80              |
   +-----------------------------+-----------------+
   | Source Address Translation  | Auto  Map       |
   +-----------------------------+-----------------+
   | Resources > Default Pool    | LAMP            |
   +-----------------------------+-----------------+

#. Use a new tab to access **http://10.1.10.200**.

#. Use **Ctrl + F5** to reload the page several times.

   You should notice the LAMP Pool displaying the respective page elements from both members.
   That’s all it takes to create a basic web application on the BIG-IP system.

#. Close the tab.

#. In the Configuration Utility, open the **Local Traffic > Pools >
   Statistics** page.

#. Expand the **LAMP** by clicking on the **+** icon.

   |image3|

   You use the **Statistics** page to identify the amount of traffic sent
   to the pool members. Notice that the requests are evenly distributed
   across all three web servers.

#. Select the **LAMP** checkbox, and then click **Reset**.

   |image4|

Task 3 – Create a Forwarding Virtual Server

An IP forwarding virtual server accepts traffic that matches the virtual server address and forwards it to the destination IP address
that is specified in the request rather than load balancing the traffic to a pool. Address translation is disabled when you create an
IP forwarding virtual server, leaving the destination address in the packet unchanged. When creating an IP forwarding virtual server,
as with all virtual servers, you can create either a host IP forwarding virtual server, which forwards traffic for a single host address,
or a network IP forwarding virtual server, which forwards traffic for a subnet.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Use a new tab to attempt direct access to an internal web server at
   **http://10.1.20.252**.

   Currently you are unable to access resources on the internal network
   from the external Windows workstation.

#. Open the **Start** menu and type **cmd**, then right-click
   **cmd.exe** and select **Run as administrator**, and then click
   **Yes**.

#. At the command prompt, type (or copy and paste):

   ``route add 10.1.20.0 mask 255.255.255.0 10.1.10.245``

   This adds a route to the **10.1.20.0** network through the external self
   IP address (**10.1.10.245**) of the BIG-IP system.

#. Reload the page directed at **http://10.1.20.252**.

   The request fails again, as the BIG-IP system does not have a listener
   to forward this request to the internal network.

#. In the Configuration Utility, open the **Local Traffic > Virtual
   Servers > Virtual Server List** page and click **Create**.

#. Use the following information for the new virtual server, and then
   click **Finished**.

   +-----------------------------+--------------------+
   | Form field                  | Value              |
   +=============================+====================+
   | Name                        | forward\_virtual   |
   +-----------------------------+--------------------+
   | Type                        | Forwarding (IP)    |
   +-----------------------------+--------------------+
   | Source Address/ Mask        | 0.0.0.0/0          |
   | Destination Address/ Mask   | 10.1.20.0/24       |
   +-----------------------------+--------------------+
   | Service Port                | \* All Ports       |
   +-----------------------------+--------------------+
   | Protocol                    | \* All Protocols   |
   +-----------------------------+--------------------+
   | Source Address Translation  | Auto Map           |
   +--------------------------------------------------+

   This virtual server provides access to the **10.1.20.0/24** network on
   all ports and all protocols.

#. Reload the page directed at **http://10.1.20.252**.

   The request is successful. The BIG-IP system doesn’t act as a full
   proxy, it simply forwards requests to the internal network.

You now have access to all ports and all protocols on the **10.1.20.0**
network.

Task 4 – Create a Reject Virtual Server
A Reject virtual server rejects any traffic destined for the virtual server IP address.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. In the Configuration Utility, on the **Virtual Server List** page
   click **Create**.

#. Use the following information for the new virtual server, and then
   click **Finished**.

   +-----------------------------+-----------------------+
   | Form field                  | Value                 |
   +=============================+=======================+
   | Name                        | reject\_server   |
   +-----------------------------+-----------------------+
   | Type                        | Reject                |
   +-----------------------------+-----------------------+
   | Source Address/ Mask        | 0.0.0.0/0             |
   | Destination Address/ Mask   | 10.1.20.252           |
   +-----------------------------+-----------------------+
   | Service Port                | \* All Ports          |
   +-----------------------------+-----------------------+
   | Protocol                    | \* All Protocols      |
   +-----------------------------+-----------------------+

#. Reload the page directed at **http://10.1.20.252**.

#. Although you still have access to the **10.1.20.0** network, you no
   longer have access to **10.1.20.252** (LAMP Server).

#. Close the **Browser Tab**.

#. In the command prompt type the following, and then close the command
   prompt.

   ``route DELETE 10.1.20.0``

#. In the Configuration Utility, select the **forward\_virtual** and
   **reject\_win\_server** checkboxes and then click **Delete** and
   **Delete** again.

Task 5 – Use Different Pool Options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Open the **Local Traffic > Pools > Pool List** page and click
   **LAMP pool**, and then open the **Members** page.

   |image5|

   Currently the pool is using the default load balancing method: **Round
   Robin**.

#. From the **Load Balancing Method** list select **Ratio (member)**,
   and then click **Update**.

#. Examine the **Current Members** section.

   Currently the LAMP pool member has a ratio of (**1**).

#. If there are multiple pool members by selecting **Ratio (member)** it
   is possible to assign ratio values to each member of the pool.  The effect
   this would have is that requests would be distributed to members of a pool
   based on the ratio value assigned.   For example, if there were three pool
   members a ratio value of **10 - 5 - 1** could be assigned to each pool member
   respectivey.   

#. In this scenario, requests would be distributed to the three pool members in a 
**10 – 5 – 1** ratio.

.. |image1| image:: /_static/class1/image3.png
   :width: 5.32107in
   :height: 0.55645in
.. |image2| image:: /_static/class1/image4.png
   :width: 5.06779in
   :height: 0.86290in
.. |image3| image:: /_static/class1/image5.png
   :width: 3.32258in
   :height: 0.68200in
.. |image4| image:: /_static/class1/image6.png
   :width: 4.03226in
   :height: 1.21631in
.. |image5| image:: /_static/class1/image7.png
   :width: 3.10484in
   :height: 0.51346in
.. |image6| image:: /_static/class1/image8.png
   :width: 1.65833in
   :height: 0.99709in
