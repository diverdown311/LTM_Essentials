Lab 3: Use SSL Offload and Best Practices
-------------------------------------------------

In this lab we will configure client-side SSL processing on the BIG-IP

Objective:
#. Create a self-signed certificate
#. Create a client SSL Profile
#. Modify an existing HTTP Virtual Server to use HTTPS

We will create a self-signed certificate and key for a client SSL profile to
attach to our virtual server

Task 1 – Create a self-signed certificate and key

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Go to **System**, **Certificate Management**, **Traffic Certificate Management**, 
   **SSL Certificate List** and select Create.
   
#. The default key size is 2048.

#. Enter the following data

	Name			my-selfsigned-cert
	Issuer			Self
	Common Name		www.f5demo.com
	Fill out the rest as desired
	
	
Task 2 - Configure an SSL Client Profile

#. Go to **Local Traffic**, **Profiles**, **SSL**, **Client menu and select **Create** 

#. Under **General Properties**
	Name		my_clientssl_profile

#. Under **Configuration** in the **Certificate Key Chain** section, select the **Custom**
   box and hit **Add** 
   
   In the **Add SSL Certificate to Key Chain** pop-up select
   
   **Certificate**	my-selfsigned-cert
   **Key**			my-selfsigned-cert
   
#. Select **Add** 

#. Click **Finished**

Building our New Secure Virtual Server

#. Go to **Local Traffic**, **Virtual Servers**, and hit the **Create** button.

	**Name**							LAMP_SSL
	**Source Address**					0.0.0.0/0
	**Destination Address/Mask**		10.1.10.201
	**Service Port**					443
	**SSL Profile (Client)**			my_clientssl_profile (the profile you just created)
	**Source Address Translation**		Auto Map
	**Default Pool**					LAMP
	Default all other settings
	**Finish**
	
#. Test your new secure server.  Using a browser go to https://10.1.10.201

	What port did your pool members see traffic on?
	

Task 2 - Configure HTTP Compression


#. Open the **LAMP_SSL** Virtual Server and click **Create**.

#. Use the following information for the new virtual server, and then
   click **Finished**.

   +-------------------------------------------+-------------------+
   | Form field                                | Value             |
   +===========================================+===================+
   +-------------------------------------------+-------------------+
   | HTTP Profile                              | http              |
   +-------------------------------------------+-------------------+
   | Acceleration > HTTP Compression Profile   | httpcompression   |
   +-------------------------------------------+-------------------+
 





#. Use a new tab to access **https://10.1.10.20**.

   The page fails to load. At this point the BIG-IP system is simply
   forwarding SSL requests to the downstream HTTPS web server without
   decrypting them first. This prevents the BIG-IP system from performing
   any HTTP functions (such as HTTP compression) which is why the page
   fails to load.

#. In the Configuration Utility, on the **Virtual Server List** page
   click **https\_virtual**, and then for **SSL Profile (Client)** move
   **clientssl** to the **Selected** list.

#. For **SSL Profile (Server)** move **serverssl-insecure-compatible**
   to the **Selected** list, and then click **Update**.

#. Reload the **https://10.1.10.20** page.

   The page displays as expected.

#. Identify the protocol used in the URL, and the port used on the
   downstream server (in the **Pool member address/port entry**).

   Although the BIG-IP system is now decrypting requests, it is
   re-encrypting before sending to the HTTPS web servers, which means
   they must perform the
   CPU-intensive task of decrypting requests and encrypting responses.

#. In the Configuration Utility, on the **https\_virtual** page, for
   **SSL Profile (Server)** move **serverssl-insecure-compatible** back
   to the **Available** list, and then click **Update**.

#. Open the **Resources** page.

   |image13|

#. From the **Default Pool** list select **http\_pool**, and then click
   **Update**.

#. Reload the **https://10.1.10.20** page.

#. Identify the protocol used in the URL, and the port used on the
   downstream server (in the **Pool member address/port entry**).

   The BIP-IP system is now performing SSL offload, sending all
   downstream requests using port 80. This means that the web servers no
   longer need to perform the
   CPU-intensive task of decrypting requests and encrypting responses.

#. Close the tab.










Task 2 – Configure BIG-IP Best Practices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Close the Configuration Utility, then open Internet Explorer and
   access **https://10.1.10.240**.

   Currently the BIG-IP system can be accessed by the outside world using
   the external self IP address, which is not recommended.

#. Log into the BIG-IP system, and then open the **Network > Self IPs**
   page and click **10.1.10.240**.

#. From the **Port Lockdown** list select **Allow None**, and then click
   **Update**.

#. Return to the **Self IPs** page.

   Why are you now unable to access the BIG-IP system?

#. Close Internet Explorer, then open Chrome and access
   **https://10.1.1.245** and then log into the BIG-IP system as
   **admin** / **admin**.

   It is not recommended to use the default admin account.

#. Open the **System > Users > Authentication** page and click
   **Change**.

#. From the **User Directory** list select **Remote – Active
   Directory**.

#. Use the following information, and then click **Finished**.

   +--------------------------+----------------------------------------------------+
   | Form field               | Value                                              |
   +==========================+====================================================+
   | Host                     | 10.1.20.251                                        |
   +--------------------------+----------------------------------------------------+
   | Remote Directory Tree    | DC=f5demo,DC=com                                   |
   +--------------------------+----------------------------------------------------+
   | Bind DN                  | CN=Service Account,OU=Corporate,DC=f5demo,DC=com   |
   +--------------------------+----------------------------------------------------+
   | Bind Password            | password                                           |
   +--------------------------+----------------------------------------------------+
   | Check Member Attribute   | Enabled (selected)                                 |
   +--------------------------+----------------------------------------------------+
   | Role                     | Guest                                              |
   +--------------------------+----------------------------------------------------+

#. Open the **Remote Role Groups** page and click **Create**.

   |image14|

#. Use the following information, and then click **Finished**.

   +--------------------+-----------------------------------------------------+
   | Form field         | Value                                               |
   +====================+=====================================================+
   | Group Name         | F5Admins                                            |
   +--------------------+-----------------------------------------------------+
   | Line Order         | 10                                                  |
   +--------------------+-----------------------------------------------------+
   | Attribute String   | memberOf=CN=loraxadmins,CN=Users,DC=f5demo,DC=com   |
   +--------------------+-----------------------------------------------------+
   | Assigned Role      | Administrator                                       |
   +--------------------+-----------------------------------------------------+
   | Terminal Access    | tmsh                                                |
   +--------------------+-----------------------------------------------------+

#. Create another role group using the following information, and then
   click **Finished**.

   +--------------------+---------------------------------------------------+
   | Form field         | Value                                             |
   +====================+===================================================+
   | Group Name         | F5ResourceAdmins                                  |
   +--------------------+---------------------------------------------------+
   | Line Order         | 15                                                |
   +--------------------+---------------------------------------------------+
   | Attribute String   | memberOf=CN=resadmins,CN=Users,DC=f5demo,DC=com   |
   +--------------------+---------------------------------------------------+
   | Assigned Role      | Resource Administrator                            |
   +--------------------+---------------------------------------------------+
   | Terminal Access    | Disabled                                          |
   +--------------------+---------------------------------------------------+

#. Create another role group using the following information, and then
   click **Finished**.

   +--------------------+---------------------------------------------------+
   | Form field         | Value                                             |
   +====================+===================================================+
   | Group Name         | F5Operators                                       |
   +--------------------+---------------------------------------------------+
   | Line Order         | 20                                                |
   +--------------------+---------------------------------------------------+
   | Attribute String   | memberOf=CN=operators,CN=Users,DC=f5demo,DC=com   |
   +--------------------+---------------------------------------------------+
   | Assigned Role      | Operator                                          |
   +--------------------+---------------------------------------------------+
   | Terminal Access    | Disabled                                          |
   +--------------------+---------------------------------------------------+

#. Open the **System > Users > User List** page.

#. Select the **admin** account and change the password to
   **admin-pass** and then click **Update**.

#. Log in as **bigip\_operator** / **password**.

#. Notice the user’s role at the top of the page.

   |image15|

#. Open the **Virtual Server List** page and examine the **Create**
   button.

   This user can view all virtual servers and other BIG-IP system objects,
   but can’t create or update objects.

#. Log out and then log back in as **bigip\_ra** / **password**.

#. Notice the user’s role at the top of the page.

#. Open the **Virtual Server List** page.

   This user and see and manage all virtual servers.

#. Open the **System > Users > Authentication** page and examine the
   **Change** button.

#. Log out and then log back in as **bigip\_admin** / **admin**. (NOTE:
   You are intentionally logging in with the wrong password.)

#. Log in as **bigip\_admin** / **password**.

#. Open the **System > Logs > Audit > List** page, and then sort the
   list by the **Time** column in descending order.

   |image16|

#. Examine the login and logout details for the three users.

   You can see when each user logged in, logged out, and failed to login
   correctly.

