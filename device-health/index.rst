.. _device-health:

Device Health
=============

Device Health represents the integrity of the device while it is on SmartThings platform.
It comprises of the device's battery life, its network connectivity, and its ability to issue tamper alerts.

Note that the Device Health does not refer to any device bugs, or to any issues arising from pairing or unpairing of devices, nor does it refer to any problems of updating device states: these phenomena constitute basic device behavior and are distinct from Device Health-specific features described in this guide.

----

End-user opt-in
---------------
For the end-user, Device Health is an opt-in feature now available in SmartThings mobile app.
In the mobile app find *Device Health* option under the menu (in either Android or iOS version) and toggle it ON.
This will add all your devices for Device Health check.
You may have a device that is not yet supported for Device Health check.
In the *Device Health* screen of SmartThings mobile app, tap on *Learn More* to find the `latest list of supported devices for Device Health checks <https://support.smartthings.com/hc/en-us/articles/214529303-Device-Health>`_.

.. note::

  Device Health is not supported by Hub v1.

----

Managing a device for Device Health
-----------------------------------
SmartThings offers a microservice called Device Watch that helps developers manage the health of the devices connected to the SmartThings platform. 

Device Health starts with enrolling a device in Device Watch for health check. 
The Device Watch microservice then monitors the device and reports the status of the health events.

Enrolling a device is done by adding the health check-specific code in its Device Handler. 

Tracked vs. untracked devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Device Watch health check can be done in two ways: 

- In one way, the Device Watch microservice will track a device and declare whether a device is offline, for example--this is called enrolling a device as a *tracked* device. ZigBee and Z-Wave devices are Hub-connected and report to the Hub periodically. These reporting intervals are used to track these devices.
- In the second way, the device itself will have to explicitly send an Event, such as being offline, to the Device Watch microservice--this is called enrolling the device as an *untracked* device. Some LAN- and Cloud-connected devices do not report at any regular interval to Hub, or do not report at all *unless* being explicitly asked to do so. For these devices there are no periodic reporting intervals for the Device Watch microservice to rely on. As a result, these devices are enrolled in Device Watch as untracked devices.

----

.. _devicehealth_tracked: 

Enroll as a tracked device
--------------------------

To enroll the device as a tracked device we have to use the Health Check capability of the device. 
The Health Check capability has one attribute, the ``checkInterval`` attribute, and one command, the ``ping`` command. 

The ``checkInterval`` attribute is a numeric value representing seconds.  

The ``checkInterval`` value is set by sending a device Event. The Device Watch microservice listens for this Event to register the device in its tracking system. The Device Watch microservice will use the ``checkInterval`` attribute to determine the health check status of the device. 

In the Device Handler, proceed as follows:

- Include the ``Health Check`` capability in the ``metadata`` section of the Device Handler. 
- Set the ``checkInterval`` attribute to a numeric value in seconds, and send this attribute with :ref:`device_handler_ref_sendevent` method to Device Watch. 

.. code-block:: groovy
	
    metadata {
    	definition(...) {
       	    capability "Health Check"
        	...
        }
    }

    ...
    
    def updated() {
    	// checkInterval is set to 32 minutes. 
    	// If Device Watch receives no device events for 32 mins, it will call ping()
    	sendEvent(name: "checkInterval", value: 2 * 15 * 60 + 2 * 60, displayed: false, data: [protocol: "zwave", hubHardwareId: device.hub.hardwareID])
    }

- On receiving this Event, Device Watch will register the device and starts tracking the device. Tracking is done by counting down the ``checkInterval`` value from the time the last Event reported by the device. 
- If the device does not report any Event for the duration of ``checkInterval`` seconds, then Device Watch will ask the ``ping()`` command in the Device Handler to ping the device. 
- Make sure you implement the ``ping()`` method in the Device Handler. See an example for a ZigBee device below:

.. code-block:: groovy

	// ping() is used by Device Watch in an attempt to reach the Device
	 
	def ping() {
	    return refresh()
	}

	def refresh() {
	    zigbee.onOffRefresh() + zigbee.onOffConfig()
	}

- If the device does not respond to the ``ping()`` command *for one full minute* (one minute is the current standard for the Device Watch microservice), then the device is considered offline and the Device Watch microservice will set the status of the device to offline. 

.. _devicehealth_untracked: 

Enroll as an untracked device
-----------------------------

To enroll the device as an untracked device, you send a ``DeviceWatch-Enroll`` Event to the Device Watch microservice, instead of sending an Event with ``checkInterval`` value.

In the Device Handler, proceed as follows:

- Include the ``Health Check`` capability in the ``metadata`` section of the Device Handler. 
- In the ``installed()`` section of the Device Handler, send an Event with the name ``DeviceWatch-Enroll``. 
- Whenever you make a decision on the status of the device, explicitly send an Event with ``DeviceWatch-DeviceStatusUpdate`` name. See an example Device Handler for a cloud device below:

.. code-block:: groovy
	
    metadata {
    	definition(...) {
       	    capability "Health Check"
        	...
        }
    }

    ...
    
    // Enroll the cloud device in Device Watch
    void installed() {
    	sendEvent(name: "DeviceWatch-Enroll", value: "{\"protocol\": \"cloud\", \"scheme\":\"untracked\", \"hubHardwareId\": \"${device?.hub?.hardwareID}\"}")
    }

	...

    def refresh() {
    	log.debug "Executing 'refresh'"
    	def resp = parent.apiGET("/lights/${selector()}")
    	if (resp.status == 404) {
    	    state.online = false
    	    // Send the Event to explicitly report the device status as offline
    	    sendEvent(name: "DeviceWatch-DeviceStatusUpdate", value: "offline", displayed: false)
    	    log.warn "$device is Offline"
    	    return []
    	}

	    ...

	    def data = resp.data[0]
	    log.debug("Data: ${data}")

	    ...

	    if (data.connected) {
	    sendEvent(name: "DeviceWatch-DeviceStatus", value: "online", displayed: false)
	    log.debug "$device is Online"
	    } else {
	    sendEvent(name: "DeviceWatch-DeviceStatus", value: "offline", displayed: false)
	    log.warn "$device is Offline"
	    }
    }

.. note::

  A good practice is to enroll ZigBee and Z-Wave devices as tracked devices (by sending ``checkInterval`` event); and LAN- and Cloud-connected devices as untracked devices (by sending ``DeviceWatch-Enroll`` event).

----

Device Watch Events
^^^^^^^^^^^^^^^^^^^
The Events to Device Watch microservice are to be sent in the Device Handler using the ``sendEvent()`` method with the following options:

======================== =====================
Device Watch Event       Possible String Values for the ``value`` option of ``sendEvent()`` method       
======================== ===================== 
DeviceWatch-Enroll       ``protocol``: "zigbee", "zwave", "cloud", "lan"; ``scheme``: "tracked", "untracked"; ``hubHardwareId``: "${device?.hub?.hardwareID}" (Hub hardware Id)  				   
DeviceWatch-DeviceStatus "online" or "offline"
======================== ===================== 

----

Health states
-------------

The Device Watch microservice currently indicates two health states for a device that is enrolled in Device Watch: *online* and *offline*.

============ =================== ======================================
Health state As displayed on IDE As displayed on SmartThings mobile app
============ =================== ======================================
Online       ONLINE              No health-specific indicator
Offline 	 OFFLINE             Unavailable, with a red mark 
============ =================== ======================================

.. note::

  If for some reason the Hub goes offline, then all devices that were paired with the Hub will have their health status set to offline.

----

.. _devicehealth_checkinterval:

Setting checkInterval
---------------------

What is an appropriate value for the ``checkInterval`` attribute? 
In other words, how long should Device Watch wait before deciding on the health status of a device? 

If we know the reporting interval of a ZigBee, Z-Wave device, then the ``checkInterval`` value should be set to a multiple of the reporting interval to prevent false positives. 
For example, if a thermometer device sends us a temperature report at least every 5 minutes, we should set the ``checkInterval`` to 12 minutes. 
This ensures that the device will be marked offline only when we miss two expected temperature reports, with an additional buffer of two minutes in case an event is delayed or the timer on the device isn't accurate.

See the following diagram:

.. image:: ../img/device-health/checkIntervalTypes.png
    :scale: 70

- For a Hub-polled Z-Wave device with a reporting interval of 15 minutes, set the ``checkInterval`` to 32 minutes.
- For a Sleepy Z-Wave device with 4 hours reporting interval, set the ``checkInterval`` to 8 hours and two minutes.
