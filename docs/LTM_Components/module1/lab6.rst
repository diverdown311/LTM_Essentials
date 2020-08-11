Task 3 – Re-create the Application using iApp
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Open the **Virtual Server List** page, then select the
   **http\_virtual** and **https\_virtual** checkboxes, and then click
   **Delete** twice.

#. Open the **Pool List** page, then select the **http\_pool** and
   **https\_pool** checkboxes, and then click **Delete** twice.

#. Open the **Node List** page, then select the **node1**, **node2**,
   and **node3** checkboxes, and then click **Delete** twice.

#. Open the **iApps > Application Services > Applications** page and
   click **Create**.

#. Create an application using the following information, and then click
   **Finished**.

   +-----------------------------------+---------------------------------------+
   | Form field                        | Value                                 |
   +===================================+=======================================+
   | User Name                         | https\_app                            |
   +-----------------------------------+---------------------------------------+
   | Template                          | f5.http                               |
   +-----------------------------------+---------------------------------------+
   | Network > Do you want to use the  | Yes                                   |
   | latest TCP profiles?              |                                       |
   +-----------------------------------+---------------------------------------+
   | SSL Encryption > How should the   | Terminate SSL from clients, plaintext |
   | BIG-IP system handle SSL traffic? | to servers                            |
   +-----------------------------------+---------------------------------------+
   | Virtual Server and Pools > What   | 10.1.10.20                            |
   | IP address do you want to use     |                                       |
   +-----------------------------------+---------------------------------------+
   | FQDN                              | www.f5demo.com                        |
   +-----------------------------------+---------------------------------------+
   | Web servers                       | 10.1.20.11: 80 (Click **Add**)        |
   |                                   | 10.1.20.12: 80 (Click **Add**)        |
   |                                   | 10.1.20.13: 80                        |
   +-----------------------------------+---------------------------------------+
   | Application Health > What HTTP    | /index.php                            |
   | URI                               |                                       |
   +-----------------------------------+---------------------------------------+
   | Expected Response                 | Welcome                               |
   +-----------------------------------+---------------------------------------+
   
#. Open the **Virtual Server List** page.

   iApp created two virtual servers for the web application. The port 80
   virtual server is used to redirect requests to the port 443 virtual
   server.

#. Open the **Pool List** page.

   iApp created a pool with three pool members and a monitor attached
   (which you can identify by it being identified as available).

#. Open the **Monitors** page and click **https\_app\_http\_monitor**.

   iApp created the custom HTTP monitor for the web application.

#. Use a new tab to access **http://10.1.10.20**.

   Notice that the request is redirected to **https**. The requests are
   sent to the web servers on port **80**, identifying that SSL offload is
   taking place.

#. Close the tab.

.. |image13| image:: /_static/class1/image15.png
   :width: 3.49562in
   :height: 0.60484in
.. |image14| image:: /_static/class1/image16.png
   :width: 4.39805in
   :height: 0.60484in
.. |image15| image:: /_static/class1/image17.png
   :width: 4.50934in
   :height: 0.38567in
.. |image16| image:: /_static/class1/image18.png
   :width: 3.65323in
   :height: 0.78965in
