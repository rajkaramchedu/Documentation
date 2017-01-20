.. _superlan-connect:

=================
Super LAN Connect
=================

The goal of Super LAN Connect app is to minimize the complexity of LAN device discovery. 

The SmartThings platform will discover a LAN-connected or a Cloud-connected device only when a Service Manager SmartApp for that specific device is present. 

This means that if you want to integrate multiple LAN devices, such as a Samsung SmartCam and a Bose Speaker, then you will need multiple Service Manager SmartApps, i.e., a separate Service Manager SmartApp for each LAN device.  
On the contrary, the platform does not have any such Service Manager SmartApp requirement for a ZigBee or a Z-Wave device.

The new Super LAN Connect app eliminates the Service Manager SmartApp requirement for some LAN-connected devices, thereby making for a much smoother and quicker LAN device discovery.
See :ref:`superlan-devices`.

Impact on the developer
-----------------------

For the LAN devices in the :ref:`superlan-devices` if you have made any customizations to either your Service Manager SmartApp or your Device Handler, then your LAN device integration will be impacted.
See below.

===================== =============================== =========
If you have											  Then
----------------------------------------------------- ---------
Custom Device Handler Custom Service Manager SmartApp Impact is
===================== =============================== =========
Yes 				  No 							  Your custom LAN Device Handler will be overwritten with SmartThings version of the LAN Device Handler.
Yes 				  Yes 							  Not impacted
===================== =============================== =========

.. _superlan-devices:

List of Supported LAN Devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently a limited number of LAN devices, as shown below, can be discovered with the Super LAN Connect app.

========== ======================================
LAN Device Super LAN Connect-supported LAN Device      
========== ======================================
Samsung    Samsung speakers  
Sonos	   Sonos speakers
Bose	   Bose speakers
Wemo	   Wemo switch
           Wemo motion sensor
Hue		   Hue
========== ======================================
