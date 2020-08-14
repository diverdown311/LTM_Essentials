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
   +--------------==+--------------------------------+
   | Administrator  | admin                          |
   | Username       |                                |
   +----------------+--------------------------------+
   | Administrator  | admin.F5demo.com               |
   | Password       |                                |
   +----------------+--------------------------------+
   
   
#. Click on **Retrieve Device Information**

n **BIGIP02** click Add, and enter the following information


   +----------------+--------------------------------+
   | Form field     | Value                          |
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

Task 3 – Test Failover

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows 10 Jump Host using Google Chrome log into both BIG-IP systems.

**On BIGIP01**

#. Navigate to **Device Management**, click on **Devices**, click on the **bigip01.f5demo.com** object.

#. Scroll to the bottom and click on **Force Offline**

#. Notice in the top left corner the device status should reflect **FORCED OFFLINE**


**On BIGIP02**

#. Navigate to **Local Traffic > Virtual Servers** and right-click on
   **Statistics** and open the page in a new tab.

#. Use both statistics tabs (click **Refresh**) to identify which BIG-IP
   system processed the incoming request.

#. In the Configuration Utility tab for **B**, open the
   **Device Management > Traffic Groups** page and click
   **traffic-group-1**.

   For this traffic group the **Current Device** is **bigipB.f5demo.com**.

#. Click **Force to Standby** twice, and then view the value in the
   **Active Device** column.

#. In the Lorax Investments page, edit the URL to **http://10.1.10.20**,
   then use both statistics tabs to identify which BIG-IP system
   processed the incoming request.

**On bigipA.f5demo.com**

#. In the Configuration Utility tab, open the **Device Management >
   Devices** page and click **bigipA.f5demo.com (Self)**.

#. Click **Force Offline** and then **OK**.

**On bigipB.f5demo.com**

#. Note the status of **bigipB.f5demo.com**.

**On bigipA.f5demo.com**

#. On the **Devices** page click **Release Offline** and then **OK**.

**On bigipB.f5demo.com**

#. Note the status of **bigipB.f5demo.com**.

When **bigipA.f5demo.com** comes back online it doesn’t become the
active device.



Task 4 – Create an Active / Active Pair
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



**On bigipA.f5demo.com**

#. Open the **Device Management > Traffic Groups** page and click
   **Create**.

#. Create a traffic group using the following information, and then
   click **Create Traffic Group**.

   +-------------------+--------------------------+
   | Form field        | Value                    |
   +===================+==========================+
   | Name              | traffic-group-2          |
   +-------------------+--------------------------+
   | Failover Method   | Preferred Device Order   |
   +-------------------+--------------------------+
   | Preferred Order   | bigipA.f5demo.com        |
   |                   | bigipB.f5demo.com        |
   +-------------------+--------------------------+

#. Open the **Local Traffic > Virtual Servers > Virtual Address List**
   page and click **10.1.10.25**.

#. From the **Traffic Group** list select **traffic-group-2
   (floating)**, and then click **Update**.

   |image19|

#. Click **Changes Pending**.

#. Leave the default options selected and click **Sync**.

#. Note the status of both BIG-IP systems.

   You now have an active / active pair.

#. Reset both statistics pages.

#. Access **https ://10.1.10.20** and identify which BIG-IP processed
   the request.

#. Access **http://10.1.10.25** and identify which BIG-IP is processed
   the request.

That concludes the hands-on exercises for the Introduction to ADC
Deployments with LTM lab session.

.. |image17| image:: /_static/class1/image19.png
   :width: 1.70088in
   :height: 0.61232in
.. |image18| image:: /_static/class1/image20.png
   :width: 1.70088in
   :height: 0.60540in
.. |image19| image:: /_static/class1/image21.png
   :width: 3.98717in
   :height: 1.04839in
