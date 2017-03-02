.. _zigbee_ref:

ZigBee Reference
================

.. note::
    As of now, any time the ZigBee library logs a message, an error will be seen. This is a known issue and will be fixed with the next deploy.

The ZigBee library contains many shorthands and conveniences for developing ZigBee Device Handlers.

ZigBee devices have fingerprints that define what the device is when it joins a ZigBee network.
Currently you define the expected fingerprint for a device in the Device Handler metadata block as part of the definition. An example would look like this:

.. code-block:: groovy

    metadata {
        // Automatically generated. Make future change here.
        definition (name: "SmartPower Outlet", namespace: "smartthings", author: "SmartThings") {
            capability "Actuator"
            capability "Switch"
            capability "Power Meter"
            capability "Configuration"
            capability "Refresh"
            capability "Sensor"

        // indicates that device keeps track of heartbeat (in state.heartbeat)
        attribute "heartbeat", "string"

        fingerprint profileId: "0104", inClusters: "0000,0003,0004,0005,0006,0B04,0B05", outClusters: "0019", manufacturer: "CentraLite",  model: "3200", deviceJoinName: "Outlet"
        fingerprint profileId: "0104", inClusters: "0000,0003,0004,0005,0006,0B04,0B05", outClusters: "0019", manufacturer: "CentraLite",  model: "3200-Sgb", deviceJoinName: "Outlet"
        fingerprint profileId: "0104", inClusters: "0000,0003,0004,0005,0006,0B04,0B05", outClusters: "0019", manufacturer: "CentraLite",  model: "4257050-RZHAC", deviceJoinName: "Outlet"
        fingerprint profileId: "0104", inClusters: "0000,0003,0004,0005,0006,0B04,0B05", outClusters: "0019"
    }

If the fingerprint declaration contains a value for both the ``manufacturer`` and ``model`` attributes, you have the option to add the ``deviceJoinName`` attribute.

===================== =========== ==========
Fingerprint Attribute Type        Value
===================== =========== ==========
deviceJoinName        ``String``  Overrides the device type name when pairing. This allows the developer to customize the device name while joining a ZigBee device.
===================== =========== ==========

----

Parse methods
-------------

zigbee.getEvent()
^^^^^^^^^^^^^^^^^

The ``getEvent()`` method will try to parse ZigBee Clusters into a map whose key/value pairs can be directly handled by the ``sendEvent()`` method.

**Signature:**
    .. code-block:: groovy

        Map(String:String) zigbee.getEvent(String description)

**Parameters:**
    - **description**: The description value passed to the parse method from the device.

**Return Values**:
    *Map: [name: <event name>, value: <event value>]*

**Example**

.. code-block:: groovy

    def parse(String description) {
        def result = zigbee.getEvent(description)

        if(result) {
            sendEvent(result)
        } else {
            // zigbee.getEvent was unable to parse description. Description must be parsed manually.
        }
    }

The ``zigbee.getEvent()`` method can parse the following event types with work being done to for additional event types:

================== =================
Event Type         Cluster value
================== =================
switch             0x0006
level              0x0008
power              0x0702 and 0x0B04
colorTemperature   0x0300
================== =================

.. note::
    Only color temperature can be parsed. Full color control is in the works.

.. note::
    For the power event type, the value can be reported in mW, W, or kW. This means that it is up to the developer to make adjustments to the value before calling sendEvent so it will be displayed correctly.

----

Low level commands
------------------

additionalParams
^^^^^^^^^^^^^^^^

There are several ZigBee methods that support an optional map parameter called additionalParams.  This is
intended to be used to support future params without affecting backward compatibility.  The following keys are
supported:

============= ========= ==============
Key            Type      Description
============= ========= ==============
mfgCode        integer   The ZigBee manufacturing code (e.g. 0x110A)
destEndpoint   integer   The destination endpoint for the given message (e.g. 0x02)
============= ========= ==============

zigbee.command()
^^^^^^^^^^^^^^^^

Send a Cluster specific Command.  This method is overloaded and has a couple of signatures.

**Signature:**
    .. code-block:: groovy

        zigbee.command(Integer Cluster, Integer Command, [String... payload])

**Parameters:**
    - **Cluster**: The Cluster ID
    - **Command**: The Command ID
    - **payload** (optional): Zero or more arguments required by the Command. Each argument should be passed as an ASCII hex string in little endian format of the appropriate width for the data type. For example, to pass the value 5 for a UINT24 (24-bit unsigned integer) you would pass “050000”.

**Signature:**
    .. code-block:: groovy

        zigbee.command(Integer Cluster, Integer Command, String payload, additionalParams=[:])

**Parameters:**
    - **Cluster**: The Cluster ID
    - **Command**: The Command ID
    - **payload**: An ASCII hex string in little endian format of the appropriate width for the data type. For example, to pass the value 5 for a UINT24 (24-bit unsigned integer) you would pass “050000”.  You can also use the `DataType.pack()`_ method described below.  If you have multiple arguments, they can just be appended in order.
    - **additionalParams**: An optional map to specify additional parameters.  See `additionalParams`_ for supported attributes.

**Examples:**
    - Send *Move To Level* Command to *Level Control* Cluster.
        .. code-block:: groovy

            zigbee.command(0x0008, 0x04, "FE", "0500")

        Where *Level* equals ``0xFE`` (full on) and *Transition Time* equals ``0x0005`` (5 seconds)

    - Send *Off* Command to *On/Off* Cluster.
        .. code-block:: groovy

            zigbee.command(0x0006, 0x00)


zigbee.readAttribute()
^^^^^^^^^^^^^^^^^^^^^^

Read the current attribute value of the specified Cluster.

**Signature:**
    .. code-block:: groovy

        zigbee.readAttribute(Integer Cluster, Integer attributeId, Map additionalParams=[:])

**Parameters:**
    - **Cluster**: The Cluster ID to read from
    - **attributeId**: The ID of the attribute to read
    - **additionalParams**: An optional map to specify additional parameters.  See `additionalParams`_ for supported attributes.
**Example:**
    - Read *CurrentLevel* attribute of the *Level Control* Cluster.
        .. code-block:: groovy

            zigbee.readAttribute(0x0008, 0x0000)

    - Read a manufacturer specific attribute on the SmartThings multi-sensor
        .. code-block:: groovy

            zigbee.readAttribute(0xFC02, 0x0010, [mfgCode: 0x110A])



zigbee.writeAttribute()
^^^^^^^^^^^^^^^^^^^^^^^

Write the attribute value of the specified Cluster.

**Signature:**
    .. code-block:: groovy

        zigbee.writeAttribute(Integer Cluster, Integer attributeId, Integer dataType, value, Map additionalParams=[:])

**Parameters:**
    - **Cluster**: The Cluster ID to write
    - **attributeId**: The attribute ID to write
    - **dataType**: The data type ID of the attribute as specified in the `ZigBee Cluster library <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__
    - **value**: The Integer value to write for data types of *boolean*, *unsigned int*, *signed int*, *general data*, and *enumerations*. Other data types are not currently supported but will be added in the future. Let us know if you need a data type that is not currently supported.
    - **additionalParams**: An optional map to specify additional parameters.  See `additionalParams`_ for supported attributes.


**Example**:
    - Write the value 0x12AB to a unsigned 16-bit integer attribute
        .. code-block:: groovy

            zigbee.writeAttribute(0x0008, 0x0010, 0x21, 0x12AB)

    - Write a manufacturer specific attribute on the SmartThings multi-sensor
        .. code-block:: groovy

            zigbee.writeAttribute(0xFC02, 0x0000, 0x20, 1, [mfgCode: 0x110A])


zigbee.configureReporting()
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Configure a ZigBee device's reporting properties. Refer to the *Configure Reporting* Command in the `ZigBee Cluster Library <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__ for more information.

**Signature:**

.. code-block:: groovy

    zigbee.configureReporting(Integer Cluster,
        Integer attributeId, Integer dataType,
        Integer minReportTime, Integer MaxReportTime,
        [Integer reportableChange],
        Map additionalParams=[:])

**Parameters:**
    - **Cluster**: The Cluster ID of the requested report
    - **attributeId**: The attribute ID for the requested report
    - **dataType**: The two byte ZigBee type value for the requested report (see `DataType`_)
    - **minReportTime**: Minimum number of seconds between reports
    - **maxReportTime**: Maximum number of seconds between reports
    - **reportableChange** (optional): Amount of change needed to trigger a report. Required for analog data types. Discrete data types should always provide *null* for this value.
    - **additionalParams**: An optional map to specify additional parameters.  See `additionalParams`_ for supported attributes.


**Examples:**
    - Discrete data type
        .. code-block:: groovy

            zigbee.configureReporting(0x0006, 0x0000, 0x10, 0, 600, null)

    - Analog data type
        .. code-block:: groovy

            zigbee.configureReporting(0x0008, 0x0000, 0x20, 1, 3600, 0x01)

    - Configure a manufacturer specific report on the SmartThings multi-sensor
        .. code-block:: groovy

            zigbee.configureReporting(0xFC02, 0x0010, 0x18, 10, 3600, 0x01, [mfgCode: 0x110A])

----

ZigBee Capabilities
-------------------

The following table outlines the Commands necessary to both configure and get updated information from ZigBee devices that support the capabilities outlined below.

.. note::
    All methods outlined in the table need the ``zigbee.`` prefix

============= ============================================================= ============================== ==============
Capability    Configure                                                     Refresh                        Notes
============= ============================================================= ============================== ==============
Battery       configureReporting(0x0001, 0x0020, 0x20, 30, 21600, 0x01)
Color Temp    configureReporting(0x0300, 0x0007, 0x21, 1, 3600, 0x10)       readAttribute(0x0300, 0x0007)  For devices that support the Color Control Cluster (0x0300)
Level         configureReporting(0x0008, 0x0000, 0x20, 1, 3600, 0x01)       readAttribute(0x0008, 0x0000)
Power         configureReporting(0x0702, 0x0400, 0x2A, 1, 600, 0x05)        readAttribute(0x0704, 0x0400)  For devices that support the Metering Cluster (0x0704)
Power         configureReporting(0x0B04, 0x050B, 0x29, 1, 600, 0x0005)      readAttribute(0x0B04, 0x050B)  For devices that support the Electrical Measurement Cluster (0x0B04)
Switch        configureReporting(0x0006, 0x0000, 0x10, 0, 600, null)        readAttribute(0x0006, 0x0000)
Temperature   configureReporting(0x0402, 0x0000, 0x29, 30, 3600, 0x0064)
============= ============================================================= ============================== ==============

**Examples:**

- Get the latest *level* value from a dimmer switch. From the table above, we find the *level* capability and look at the **Refresh** column to find the correct Command to execute.

    .. code-block:: groovy

        readAttribute(0x0008, 0x0000)

- Configure the *level* capability for a dimmer type switch. The configure reporting Command from the table above for *level* configures the device for a min reporting interval of 5 seconds, a reporting interval of 1 hour (3600 s) if there has been no activity, and a min level change of 01.

    .. code-block:: groovy

        configureReporting(0x0008, 0x0000, 0x20, 1, 3600, 0x01)

The following utility methods are available as capability based Commands.

zigbee.on()
^^^^^^^^^^^

Sends the *on* Command, ``0x01``, to the *on/off* Cluster, ``0x0006``

**Signature:**

.. code-block:: groovy

    zigbee.on()

zigbee.off()
^^^^^^^^^^^^

Sends the *off* Command, ``0x00``, to the *onoff* Cluster, ``0x0006``

**Signature:**

.. code-block:: groovy

    zigbee.off()

zigbee.setLevel()
^^^^^^^^^^^^^^^^^

Sends the *level* Command, ``0x04``, to the level control Cluster, ``0x0008`` with the passed in rate.

**Signature:**

.. code-block:: groovy

    zigbee.setLevel(Integer level, Integer rate)

**Parameters:**
    - **level**: A value between 0-100 inclusive.
    - **rate**: Time in tenths of a second. E.g. ``rate = 100 //max value`` translates to 10 seconds.

zigbee.setColorTemperature()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sets the color to the specified temperature value in K.

**Signature:**

.. code-block:: groovy

    zigbee.setColorTemperature(Integer value)

**Parameters:**
    - **value**: The temperature value to set the color to in K. Usually greater than *2700*

----

ZigBee helper commands
----------------------

zigbee.parseDescriptionAsMap()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Parses a device description into a map that contains data and capabilities.

**Signature:**

.. code-block:: groovy

    zigbee.parseDescriptionAsMap(String description)

**Parameters:**
    - **description**: The description string from the device

zigbee.convertToHexString()
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Convert the given value to a hex string of given width

**Signature:**

.. code-block:: groovy

    zigbee.convertToHexString(Integer value, Integer width)

**Parameters:**
    - **value**: Integer value to be converted
    - **width**: the minimum width of the hex string. Default value is 2

**Examples:**

.. code-block:: groovy

    zigbee.convertToHexString(10, 2) //result: 0A
    zigbee.convertToHexString(10, 4) //result: 000A



zigbee.convertHexToInt()
^^^^^^^^^^^^^^^^^^^^^^^^

Convert the given hex value to an Integer

**Signature:**

.. code-block:: groovy

    zigbee.convertHexToInt(String value)

**Parameters:**
    - **value**: The hex value to be converted to an Integer

**Example:**

.. code-block:: groovy

    zigbee.convertHexToInt("0001") //result: 1
    zigbee.convertHexToInt("000F") //result: 15
    def myInt = zigbee.convertHexToInt("12AB") // result = 4779. The result of calling Integer.parseInt()
        assert myInt == 4779 //true
        assert myInt == 0x12AB //also true


zigbee.hexNotEqual()
^^^^^^^^^^^^^^^^^^^^

Returns true if the compared hex values are not equal.

**Signature:**

.. code-block:: groovy

    zigbee.hexNotEqual(String hex1, String hex2)

**Parameters:**
    - **hex1**: Hex value to compare
    - **hex2**: Hex value to compare against first value

.. _zigbee_parse_zone_status:

zigbee.parseZoneStatus()
^^^^^^^^^^^^^^^^^^^^^^^^

Returns a ZoneStatus object (see below) withe the parsed value form the message description.  The description
should be of the form "zone status {number}" where {number} is a hex number.

**Signature:**

.. code-block:: groovy

    zigbee.parseZoneStatus(String description)

**Parameters:**
    - **description**: A zone status message description.


.. _zigbee_additional_zigbee_classes:

Additional ZigBee classes
-------------------------

There are some additional classes that are provided to make interacting with and handling Zigbee messages easier.

ZoneStatus
^^^^^^^^^^

The purpose of the ZoneStatus class is to handle the ZoneStatus attribute in the IAS Zone cluster.  It has a
single constructor that takes an int which is the ZoneStatus attribute value.

**Constructor:**

.. code-block:: groovy

   ZoneStatus(int zonestatus)

**Example:**

.. code-block:: groovy

   ZoneStatus zs = ZoneStatus(0x41) // Trouble & Alarm1

Accessing a Property/attribute
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Once you have created the ZoneStatus object you can query the individual bits in a number of ways.  First
you can get the value of each individual bit (1 or 0) by accessing the property with the same name.  The
properties are as follows:

========== ===================== ===========================================
Bit Number  Property Name         Values
========== ===================== ===========================================
0           alarm1                | 1 - opened or alarmed
                                  | 0 - closed or not alarmed
1           alarm2                | 1 - opened or alarmed
                                  | 0 - closed or not alarmed
2           tamper                | 1 - tampered
                                  | 0 - not tampered
3           battery               | 1 - low battery
                                  | 0 - battery OK
4           supervisionReports    | 1 - reports
                                  | 0 - does not report
5           restoreReports        | 1 - reports restore
                                  | 0 - does not report restore
6           trouble               | 1 - trouble/failure
                                  | 0 - OK
7           ac                    | 1 - ac/mains fault
                                  | 0 - ac/mains OK
8           test                  | 1 - sensor is in test mode
                                  | 0 - sensor is in operation mode
9           batteryDefect         | 1 - sensor detects a defective battery
                                  | 0 - sensor battery is functioning normally
========== ===================== ===========================================

See the `ZigBee Home Automation (HA) <http://www.zigbee.org/zigbee-for-developers/applicationstandards/zigbeehomeautomation/>`__
specification and the `ZigBee Cluster Library (ZCL) <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__
specification for more information.

**Example:**

.. code-block:: groovy

   ZoneStatus zs = ZoneStatus(0x41) // Trouble & Alarm1

   zs.alarm1  // 1
   zs.alarm2  // 0
   zs.trouble // 1

The ZoneStatus object also exposes a number of query methods for getting a true/false for each attribute
value.  They are as follows:

===================== ===========================
Property Name         Query method
===================== ===========================
alarm1                 isAlarm1Set()
alarm2                 isAlarm2Set()
tamper                 isTamperSet()
battery                isBatterySet()
supervisionReports     isSupervisionReportsSet()
restoreReports         isRestoreReportsSet()
trouble                isTroubleSet()
ac                     isAcSet()
test                   isTestSet()
batteryDefect          isBatteryDefectSet()
===================== ===========================

**Example of DTH parseIasMessage for a motion sensor:**

.. code-block:: groovy

   private Map parseIasMessage(String description) {
       ZoneStatus zs = zigbee.parseZoneStatus(description)
       Map resultMap = [:]

       resultMap.name = 'motion'
       resultMap.value = zs.isAlarm1Set() ? 'active' : 'inactive'

       return resultMap
   }

DataType
^^^^^^^^

The ``DataType`` class contains information and some utility methods for ZCL data types.

DataType constants
>>>>>>>>>>>>>>>>>>

The list of types and their ``DataType`` constant name are as follows:

===================== ======================== ===================
ZCL Data Type          DataType constant name   ZCL numeric value
===================== ======================== ===================
No Data                NO_DATA                  0x00
8-bit data             DATA8                    0x08
16-bit data            DATA16                   0x09
24-bit data            DATA24                   0x0a
32-bit data            DATA32                   0x0b
40-bit data            DATA40                   0x0c
48-bit data            DATA48                   0x0d
56-bit data            DATA56                   0x0e
64-bit data            DATA64                   0x0f
Boolean                BOOLEAN                  0x10
8-bit bitmap           BITMAP8                  0x18
16-bit bitmap          BITMAP16                 0x19
24-bit bitmap          BITMAP24                 0x1a
32-bit bitmap          BITMAP32                 0x1b
40-bit bitmap          BITMAP40                 0x1c
48-bit bitmap          BITMAP48                 0x1d
56-bit bitmap          BITMAP56                 0x1e
64-bit bitmap          BITMAP64                 0x1f
Unsigned 8-bit int     UINT8                    0x20
Unsigned 16-bit int    UINT16                   0x21
Unsigned 24-bit int    UINT24                   0x22
Unsigned 32-bit int    UINT32                   0x23
Unsigned 40-bit int    UINT40                   0x24
Unsigned 48-bit int    UINT48                   0x25
Unsigned 56-bit int    UINT56                   0x26
Unsigned 64-bit int    UINT64                   0x27
Signed 8-bit int       INT8                     0x28
Signed 16-bit int      INT16                    0x29
Signed 24-bit int      INT24                    0x2a
Signed 32-bit int      INT32                    0x2b
Signed 40-bit int      INT40                    0x2c
Signed 48-bit int      INT48                    0x2d
Signed 56-bit int      INT56                    0x2e
Signed 64-bit int      INT64                    0x2f
8-bit enumeration      ENUM8                    0x30
16-bit enumeration     ENUM16                   0x31
Semi-precision         FLOAT2                   0x38
Single precision       FLOAT4                   0x39
Double precision       FLOAT8                   0x3a
Octet String           STRING_OCTET             0x41
Character String       STRING_CHAR              0x42
Long Octet String      STRING_LONG_OCTET        0x43
Long Character String  STRING_LONG_CHAR         0x44
Array                  ARRAY                    0x48
Structure              STRUCTURE                0x4c
Set                    SET                      0x50
Bag                    BAG                      0x51
Time of day            TIME_OF_DAY              0xe0
Date                   DATE                     0xe1
UTCTime                UTCTIME                  0xe2
Cluster ID             CLUSTER_ID               0xe8
Attribute ID           ATTRIBUTE_ID             0xe9
BACnet OID             BACNET_OID               0xea
IEEE address           IEEE_ADDRESS             0xf0
128-bit security key   SECKEY128                0xf1
Unknown                UNKNOWN                  0xff
===================== ======================== ===================

See the `ZigBee Cluster Library (ZCL) <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__ specification for more information on the different data types and how they are used.

DataType.getLength()
>>>>>>>>>>>>>>>>>>>>>>>>

This method is used to get the length of a variable of the given type.  This length is in number of bytes.  For variable length or unknown types it will return ``null``

**Signature:**

.. code-block:: groovy

    DataType.isVariableLength(type)

**Parameters:**
    - **type**: The type to check.  This should be one of the `DataType Constants`_ defined above.

**Example**

.. code-block:: groovy

    DataType.getLength(DataType.UINT8)       // returns 1
    DataType.getLength(DataType.CLUSTER_ID)  // returns 2
    DataType.getLength(DataType.STRING_CHAR) // returns null


DataType.isVariableLength()
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

This method is used to test if a given type is variable length or not.

**Signature:**

.. code-block:: groovy

    DataType.isVariableLength(type)

**Parameters:**
    - **type**: The type to check.  This should be one of the `DataType Constants`_ defined above.

**Example**

.. code-block:: groovy

    DataType.isVariableLength(DataType.UINT8)       // returns false
    DataType.isVariableLength(DataType.STRING_CHAR) // returns true


DataType.isDiscrete()
>>>>>>>>>>>>>>>>>>>>>>>>>

This method is used to test if a given type is discrete.

**Signature:**

.. code-block:: groovy

    DataType.isDiscrete(type)

**Parameters:**
    - **type**: The type to check.  This should be one of the `DataType Constants`_ defined above.

**Example**

.. code-block:: groovy

    DataType.isDiscrete(DataType.UINT8)  // returns true
    DataType.isDiscrete(DataType.FLOAT2) // returns false


DataType.pack()
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>


This method is used to pack data of a given type into hex string form.  Currently not all DataTypes are supported by this method.  All discrete data types are supported, but only ``STRING_CHAR`` is supported for variable length data types.  If the type passed in is ``null``, an empty string, or ``DataType.UNKNOWN`` the value of data will be returned unmodified.

**Signature:**

.. code-block:: groovy

    DataType.pack(data, type, littleEndian=false)

**Parameters:**
    - **data**: The data to pack, the type of this should be appropriate for the type argument
    - **type**: The type of the data being packed.  This should be one of the `DataType Constants`_ defined above.
    - **littleEndian**: If true it will pack it with the least significant bits first.

**Example**

.. code-block:: groovy

    DataType.pack(0x01, DataType.UINT8)        // returns "01"
    DataType.pack(0x01, DataType.UINT64)       // returns "0000000000000001"
    DataType.pack(0x01, DataType.UINT32, true) // returns "01000000"
