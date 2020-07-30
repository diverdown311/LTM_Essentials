Lab 2: Work with SNAT, Profiles, and Monitors
---------------------------------------------

In this lab you will experiment with using SNAT Auto Map for inbound
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
^^^^^^^^^^^^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
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
^^^^^^^^^^^^^^^^^^^^^^^^^^^
During this section of the lab we will review many of the available
monitors and how to customze them.  The BIG-IP system includes a set of
pre-defined monitor templates for address, service, content, and interactive checks.

#. Edit the URL to **http://10.1.10.200/health\_check.html**

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
   | Send String              | GET /health\_check.html\\r\\n   |
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

Sub-Task 1 – Take 10.1.20.250:80 Offline
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. On the Windows server go to **Start > Computer**, and then navigate
   to **C:\\inetpub\\wwwroot\\lorax\_public\_site\_41**.

   This is the directory is used for pool member **10.1.20.41:80**. The
   **health\_check.html** web page currently exists on this pool member.

#. Delete **health\_check.html**.

#. Wait 13 seconds, and then in the Configuration Utility on the
   **Network Map** page click **Update Map**.

   |image11|

#. Use your mouse to hover over the pool members.

   The first pool member is offline, and all three nodes display available.

Sub-Task 2 – Disable 10.1.20.42:80
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. On the Windows server navigate to
   **C:\\inetpub\\wwwroot\\lorax\_public\_site\_42**.

#. Right-click **health\_check** and select **Open with > WordPad**.

#. In the **<p>** tag, edit the text to **Server\_Down**, and then click
   **Save**.

   This file is used by pool member **10.1.20.42:80**. This pool member
   will now match the disable string identified in the monitor.

#. Wait 13 seconds, and then in the Configuration Utility on the
   **Network Map** page click **Update Map**.

   The second pool member is now disabled; however, the virtual server and
   pool still display available.

Sub-Task 3 – Take Node 10.1.20.43 Offline
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. On the Windows server, for **Local Area Connection 3** open the
   **Internet Protocol Version 4 (TCP/IPv4)** properties.

#. Click **Advanced**, and in the list of IP addresses scroll down to
   **10.1.20.43** and click **Remove**, then click **OK** three times
   and then click **Close**.

#. Wait 13 seconds, and then in the Configuration Utility on the
   **Network Map** page, click **Update Map**.

#. Use your mouse to hover over the pool members.

   |image12|

   The virtual server and pool display disabled but available. Node
   **10.1.20.43** now displays offline, which causes pool member
   **10.1.20.43:80** to display offline.

Sub- Task 4 – Bring 10.1.20.42:80 Back Online
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. On the Windows server, in the **health\_check** WordPad document,
   edit the text back to **Server\_Up**, then click **Save**, and then
   close WordPad.

#. In the Configuration Utility on the **Network Map** page click
   **Update Map**.

   Because pool member **10.1.20.42:80** is available, the virtual server
   and pool once again display available.

#. Use an incognito window to access **http://10.1.10.25**.

   The page displays, with all page elements coming from **10.1.20.42:80**.

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
