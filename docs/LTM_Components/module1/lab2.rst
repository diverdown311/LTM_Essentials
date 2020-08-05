Lab 2: Work with SNAT, Profiles, and Monitors
---------------------------------------------

In this lab you will gain experience using SNAT Auto Map for inbound
requests as well as outbound requests from internal users. You’ll also
use an HTTP and stream profile to make global modifications to text
within a web site. Finally you’ll see how using health monitors ensures
that you the BIG-IP knows which web servers are available for client
requests.  SNAT's provide a mapping between nodes, often internal devices
and a SNAT address.   SNATs can be configured many different ways including
one-to-one mappings, many-to-one mappings, or all to one mappings.  In all cases
a SNAT must be enabled on the VLAN where the node's traffic arrives on the BIG-IP system.
In this lab we will focus on the SNAT Automap feature which automatically maps the source
address of an allowed host to an address from a defined group.   Often times this is a Self-IP
address on the BIG-IP.

Task 1 – Use SNAT AutoMap
A virtual server configured on a BIG-IP system typically translates the destination IP address
of an incoming packet to load balance requests.  Normally the source IP address remains unchanged.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. In the Configuration Utility, open the **Pool List** page and click
   **Create**.

#. For this lab we will use the existing **LAMP** pool which contains the below members.

   +---------------+------------------------------------+
   | Form field    | Value                              |
   +===============+====================================+
   | Name          | LAMP                               |
   +---------------+------------------------------------+
   | New Members   | Address: 10.1.20.252               |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+
   |               | Address 10.1.20.250                |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+
  

#. Open the **Virtual Server List** page and click on the **LAMP** virtual server


#. From the Windows Jump Host open putty, and then connect to **BIGIP01 10.1.1.4** and log
   in as **root** / **admin.F5demo.com**.

   |image7|

#. At the CLI type (or copy and paste):

   ``tcpdump -i external port 80``

#. Open a second putty session and connect to **BIGIP02 10.1.1.6**.

#. At the CLI type (or copy and paste):

   ``tcpdump -i internal port 80``

#. Use a new tab to access **http://10.1.10.200**, and then close the
   tab.

   The page displays as expected.

#. Examine the **tcpdump** in windows.

   On the external VLAN the communication is between the client IP
   address (**10.1.10.190**) and the virtual server (**10.1.10.200**).

   On the internal VLAN the communication is between the client IP
   address (**10.1.10.190**) and a back-end web server (**10.1.20.x**).

#. In both **tcpdump** sessions press the **Enter** key several times to
   move the log entries to the top of the window.

#. On the internal VLAN the requests are from the client IP address to a
   back-end web server, however there are no responses from the web
   server.

#. Press the **Enter** key several times to move the log entries to the
   top of the window.

#. In the Configuration Utility, click **LAMP**.

#. From the **Source Address Translation** list select **Auto Map** which has already
   been configured when the virtual was created.  Notice there are several options to include
   SNAT, Auto Map, and None.   

   |image8|

#. Use an incognito window to access **http://10.1.10.200**, and then
   close the window.

   SNAT Auto Map ensures that responses to server request are always sent
   back to the BIG-IP system.

#. Examine the **tcpdump** window.

   On the external VLAN the communication is still between the client IP
   address (**10.1.10.190**) and the virtual server (**10.1.10.200**).
  
   On the internal VLAN the communication is now between the BIG-IP
   internal self IP address (**10.1.20.245**) and a back-end web
   server (**10.1.20.x**).

Task 2 – Create a SNAT for Internal Resources

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Press the **Enter** key several times to move the log entries to the
   top of the window.

#. On the Windows server, change the default gateway to **10.1.20.245**
   (the BIG-IP internal floating self IP address).

#. On the Windows server, use Internet Explorer to access
   **www.f5.com**.

   The request fails as the internal resource has no access to the WAN (or
   the Internet).

#. Close the page, then on the Windows desktop examine the **tcpdump**
   windows.

   No requests are being sent to the Internet by the BIG-IP system on
   behalf of the internal resource.

#. In the Configuration Utility, open the **Local Traffic > Address
   Translation > SNAT List** page and click **Create**.

#. Use the following information for the new SNAT, and then click
   **Finished**.

   +-------------------------+--------------------------------+
   | Form field              | Value                          |
   +=========================+================================+
   | Name                    | internal\_snat                 |
   +-------------------------+--------------------------------+
   | Translation             | IP Address: 10.1.10.245        |
   +-------------------------+--------------------------------+
   | Origin                  | Address List                   |
   +-------------------------+--------------------------------+
   | Address/Prefix Length   | 10.1.20.0/24 (Click **Add**)   |
   +-------------------------+--------------------------------+

#. On the Windows server, use Internet Explorer to access
   **www.f5.com**.

   The internal user now has public access to the internet using the SNAT
   IP address of **10.1.10.100**.

#. On the Windows desktop, examine the **tcpdump** windows.

   On the external VLAN the communication is between the SNAT IP address
   (**10.1.10.100**) and the Internet resources.

   On the internal VLAN the communication is between the internal client
   (**10.1.20.251**) and the Internet resources.

#. Close the **putty** sessions.

Task 3 – Use Profiles with a Virtual Server

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Profiles are a powerful configuration tool providing an easy way to define
traffic policies and apply those policies across virtual servers.   Through
a profile you can also change a setting for traffic across many different
applications.   

Profiles provide

- A centralized place to define specific traffic behavior such as compression, SSL, 
  and authentication that can be applied to multiple virtual servers.
  
- A centralized place to change any setting and have them applied to all applications
  using an existing profile.  A profile tells a virtual server how to process packets
  it receives through the BIG-IP system.


#. From the Jump Host use a new tab to access **http://10.1.10.200**, and then select the
   links at the top of the page and examine the text on each page.

  Instead of updating all the web site code we’ll use profiles on the BIG-IP system to update the web site.

#. Close the tab.

#. In the Configuration Utility, open the **Local Traffic > Profiles >
   Other > Stream** page and click **Create**.

#. Use the following information for the profile, and then click
   **Finished**.

   +--------------+---------------------+
   | Form field   | Value               |
   +==============+=====================+
   | Name         | name\_change        |
   +--------------+---------------------+
   | Source       | Investments         |
   +--------------+---------------------+
   | Target       | Financials          |
   +--------------+---------------------+

#. Open the **Virtual Server List** page and click **LAMP**.

#. From the **Configuration** list select **Advanced**.

   |image9|

#. From the **HTTP Profile** list select **http**.

#. From the **Stream Profile** list select **name\_change**.

   |image10|

#. In the **Acceleration** section, from the **HTTP Compression
   Profile** list select **httpcompression**.

#. From the **Web Acceleration Profile** list select
   **optimized-caching**, and then click **Update**.

#. Use an incognito window to access **http://10.1.10.200**, and then
   select the links at the top of the page.

   Although the logo need to be updated, all the text on all pages now
   references **Financials**.

Task 4 – Work with Monitors

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

During this section of the lab we will review many of the available
monitors and how to customze them.  The BIG-IP system includes a set of
pre-defined monitor templates for address, service, content, and interactive checks.

#. From the Windows Jump host open a new tab in a browser and enter the 
   following URL **http://10.1.10.200/peruggia**

   We’re going to use this web page to identify if the web server is up or down.

#. Close the health check page.

#. In the Configuration Utility, open the **Local Traffic > Monitors**
   page and click **Create**.

#. Use the following information for the monitor, and then click
   **Finished**.

   +--------------------------+---------------------------------+
   | Form field               | Value                           |
   +==========================+=================================+
   | Name                     | LAMP_monitor                    |
   +--------------------------+---------------------------------+
   | Type                     | http                            |
   +--------------------------+---------------------------------+
   | Interval                 | 4                               |
   +--------------------------+---------------------------------+
   | Timeout                  | 13                              |
   +--------------------------+---------------------------------+
   | Send String              | GET /peruggia\\r\\n             |
   +--------------------------+---------------------------------+
   | Receive String           | Server\_Up                      |
   +--------------------------+---------------------------------+
   | Receive Disable String   | Server\_Down                    |
   +--------------------------+---------------------------------+

#. Open the **Pool List** page and click **LAMP**.

#. Identify the current **Availability** status of the pool.

   Unknown identifies when a pool or node doesn’t have a configured
   monitor.

#. Add **LAMP\_monitor** to the **Active** list and click **Update**.

   The **Availability** of the pool changes to **Available (Enabled)**.

#. Open the **Local Traffic > Nodes > Node List** page.

   Notice that all the nodes currently display unknown.

#. Open the **Local Traffic > Nodes > Default Monitor** page.

#. Add **gateway\_icmp** to the **Active** list and click **Update**.

#. Return to the **Nodes >ode List** page.

   All nodes now display. This means that they are all sending **icmp**
   responses.

#. Open the **Local Traffic > Network Map** page and view the status for
   **LAMP**.

   The virtual server, pool, and all three pool members display available.

#. Use your mouse to hover over the pool members.

   All two nodes also display available.

Sub-Task 1 – Take Ubuntu_LAMP1 Offline

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**, 
   then click on **LAMP** pool, click the **Members** tab then click on 
   the **LAMP_Server** then click on the **Disabled** radio button.

#. Wait 13 seconds, and then in the Configuration Utility on the
   **Network Map** page click **Update Map**.

   |image11|

#. Use your mouse to hover over the pool members.

   The first pool member is offline, while the other node displays available.

Sub-Task 2 – Disable Ubuntu_LAMP2

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**, 
   then click on **LAMP** pool, click the **Members** tab then click on 
   the **LAMP_Server2** then click on the **Disabled** radio button.

#. Wait 13 seconds, and then in the Configuration Utility on the
   **Network Map** page click **Update Map**.

#. Notice that the virtual server and pool display unavailable.

Sub- Task 4 – Bring both pool members back online

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**, 
   then click on **LAMP** pool, click the **Members** tab then click on 
   both the **LAMP_Server** and **LAMP_Server2** and then click on the **enabled** radio button.
   Follow this process for both members of the **LAMP** pool

#. Use an incognito window to access **http://10.1.10.200**.

 #. Close the page.

.. |image7| image:: /_static/class1/image9.png
   :width: 2.24402in
   :height: 0.92742in
.. |image8| image:: /_static/class1/image10.png
   :width: 2.36229in
   :height: 0.67742in
.. |image9| image:: /_static/class1/image11.png
   :width: 1.57223in
   :height: 0.61290in
.. |image10| image:: /_static/class1/image12.png
   :width: 2.44448in
   :height: 0.74194in
.. |image11| image:: /_static/class1/image13.png
   :width: 1.32536in
   :height: 0.98333in
.. |image12| image:: /_static/class1/image14.png
   :width: 2.25614in
   :height: 1.44256in