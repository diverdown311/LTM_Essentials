Lab 6 – Introduct to iApps and FAST Templates
F5 iApps are a powerful features on every BIG-IP system
that provides a better way to architect application delivery.
iApp technology abstracts the many individual components required
to deliver an application grouping resources together in templates
associated with a specific application.  iApps have been available
on the BIG-IP system for a number of years, and consist of 
three main components:

#. Application Services
#. Templates
#. Devcentral Ecosystem


Recently F5 introduced the next phase of evoltion for the BIG-IP
ADC platform known ad FAST (F5 Application Services Templates).  FAST
technology was developed for the following reasons:

#. Cosistent, cross-platform declarative APIs
#. Seamless integration and insertion into CI/CD pipelines
#. Modern development languages like Node.js and Python
#. Templates and automation

In short, FAST will enable and empower customers while they
navigate their digital transformation journey, and ensure 
their apps are available, performant, and secure.


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Task 1 – Configure the F5 iApp for NIST
(National Institute of Standards and Technology) Special
Publication 800-53 (Revision 4): Security and Privacy
Controls for Federal Information Systems and Organizations.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. The F5 NIST sp800-53 iApp has already been uploaded to BIG-IP01

#. Under the iApps menu, click on Templates, and then click on
   the f5.nist_sp800-53.v1.0.0 iApp template to view the properties.
   This particular iApp only requires the LTM module. Please review 
   each section of the iApp to become familiar with the various
   components of an iApp.
   
#. For this part of the lab we will simply provision a new application
   service leveraging the NIST sp800-53 the purpose of which is to configure
   a BIG-IP system to be compliant with security controls outlined in the NIST
   SP800-53 special publicliation.

#. Under the iApps menu, click on **Application Services** then click on
   the + sign to the right of **Applications**.
   
#. Name the Application **NIST**, click the down arrow to the right of
   **Template** and select the **f5.nist_sp800-53.v1.0.0** iApp.
   
#. Under the **Usage Banner -- AC-8** enter a banner message.

#. Enter **time.google.com** in the NTP Server field.

#. Leave all other fields with their default settings.

#. Click **Finished**




