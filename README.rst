.. This file is automatically generated by ./docs/script/make_doc.py Python script.
   Please, don't change. In case you need to make corrections or changes change
   source documentation in ./doc folder or script.

.. Generated from esphome-docs/climate/haier.rst

Haier Climate
=============

This is an implementation of the ESPHome component to control HVAC on the base of the SmartAir2 and hOn Haier protocols (AC that is controlled by the hOn or SmartAir2 application).

There are two versions of the Haier protocol: the older version uses an application called SmartAir2 while the newer version uses an application called hOn. Both protocols are compatible on the transport level but utilize different commands to control appliances.

Older Haier models controlled by the SmartAir2 application are using the KZW-W002 module. This module can't be reused, and you need to replace it with an ESP or RPI Pico W module. The USB connector on the board doesn't support the USB protocol. It is a UART port that just uses a USB connector. To connect the ESP board to your AC you can cut a USB type A cable and connect wires to the climate connector.

.. list-table:: Haier UART pinout
    :header-rows: 1

    * - Board
      - USB
      - Wire color
      - ESP8266
    * - 5V
      - VCC
      - red
      - 5V
    * - GND
      - GND
      - black
      - GND
    * - TX
      - DATA+
      - green
      - RX
    * - RX
      - DATA-
      - white
      - TX

.. raw:: HTML

  <p><a href="./docs/esphome-docs/climate/images/usb_pinout.png?raw=true"><img src="./docs/esphome-docs/climate/images/usb_pinout.png?raw=true" height="70%" width="70%"></a><br><i>&emsp;KZW-W002 module pinout</i></p>

Newer Haier models using a module called ESP32-for-Haier. It is an ESP32 single-core board with an ESP32-S0WD chip. In some cases, you can reuse this module and flash it with ESPHome, but some new modules don't support this. They look the same but have encryption enabled.

**Warning!** The new generation of ESP32-Haier devices has encryption enabled, so they can only be flashed with firmware that is signed with a private key. There is no way to make them work with ESPHome, so if you try to do it, the board will get into a boot loop with the error ``rst:0x10 (RTCWDT_RTC_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)`` The only way to recover this board is to flash it with the original image. So before starting your experiments make a backup image.

.. raw:: HTML

  <p><a href="./docs/esphome-docs/climate/images/haier_pinout.jpg?raw=true"><img src="./docs/esphome-docs/climate/images/haier_pinout.jpg?raw=true" height="70%" width="70%"></a><br><i>&emsp;ESP32-for-Haier UART0 pinout</i></p>

Also, you can use any other ESP32, ESP8266, or an RPI pico W board. In this case, you will need to cut the original wire or make a connector yourself (the board has a JST SM04B-GHS-TB connector)

This component requires a `UART Bus <https://esphome.io/components/uart#uart>`_ to be setup.

.. code-block:: yaml

    # Example configuration entry
    climate:
      - platform: haier
        id: haier_ac
        protocol: hon
        name: Haier AC
        uart_id: ac_port
        wifi_signal: true
        display: true
        visual:
          min_temperature: 16 °C
          max_temperature: 30 °C
          temperature_step: 1 °C
        supported_modes:
        - 'OFF'
        - HEAT_COOL
        - COOL
        - HEAT
        - DRY
        - FAN_ONLY
        supported_swing_modes:
        - 'OFF'
        - VERTICAL
        - HORIZONTAL
        - BOTH
      supported_presets:
        - AWAY
        - BOOST
        - SLEEP
      on_alarm_start:
        then:
          - logger.log:
              level: WARN
              format: "Alarm activated. Code: %d. Message: \"%s\""
              args: [ code, message]
      on_alarm_end:
        then:
          - logger.log:
              level: INFO
              format: "Alarm deactivated. Code: %d. Message: \"%s\""
              args: [ code, message]
      on_status_message:
        then:
          - logger.log:
              level: INFO
              format: "New status message received, size=%d, subcmd=%02X%02X"
              args: [ 'data_size', 'data[0]', 'data[1]' ]


Configuration variables:
------------------------

- **uart_id** (*Optional*, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): ID of the UART port to communicate with AC.
- **protocol** (*Optional*, string): Defines communication protocol with AC. Possible values: ``hon`` or ``smartair2``. The default value is ``smartair2``.
- **wifi_signal** (*Optional*, boolean): If ``true`` - send wifi signal level to AC.
- **answer_timeout** (*Optional*, `Time <https://esphome.io/guides/configuration-types.html#config-time>`_): Responce timeout. The default value is ``200ms``.
- **alternative_swing_control** (*Optional*, boolean): (supported by smartAir2 only) If ``true`` - use alternative values to control swing mode. Use only if the original control method is not working for your AC.
- **status_message_header_size** (*Optional*, int): (supported only by hOn) Define the header size of the status message. Can be used to handle some protocol variations. Use only if you are sure what you are doing. The default value: ``0``.
- **control_packet_size** (*Optional*, int): (supported only by hOn) Define the size of the control packet. Can help with some newer models of ACs that use bigger packets. The default value: ``10``.
- **sensors_packet_size** (*Optional*, int): (supported only by hOn) Define the size of the sensor packet of the status message. Can help with some models of ACs that have bigger sensor packet. The default value: ``22``, minimum value: ``18``.
- **control_method** (*Optional*, list): (supported only by hOn) Defines control method (should be supported by AC). Supported values: ``MONITOR_ONLY`` - no control, just monitor status, ``SET_GROUP_PARAMETERS`` - set all AC parameters with one command (default method), ``SET_SINGLE_PARAMETER`` - set each parameter individually (this method is supported by some new ceiling ACs like AD71S2SM3FA)
- **display** (*Optional*, boolean): Can be used to set the AC display off.
- **beeper** (*Optional*, boolean): Can be used to disable beeping on commands from AC. Supported only by hOn protocol.
- **supported_modes** (*Optional*, list): Can be used to disable some of AC modes. Possible values: ``'OFF'``, ``HEAT_COOL``, ``COOL``, ``HEAT``, ``DRY``, ``FAN_ONLY``.
- **supported_swing_modes** (*Optional*, list): Can be used to disable some swing modes if your AC does not support it. Possible values: ``'OFF'``, ``VERTICAL``, ``HORIZONTAL``, ``BOTH``.
- **supported_presets** (*Optional*, list): Can be used to disable some presets. Possible values for smartair2 are: ``AWAY``, ``BOOST``, ``COMFORT``. Possible values for hOn are: ``AWAY``, ``BOOST``, ``SLEEP``. ``AWAY`` preset can be enabled only in ``HEAT`` mode, it is disabled by default.
- **on_alarm_start** (*Optional*, `Automation <https://esphome.io/guides/automations#automation>`_): (supported only by hOn) Automation to perform when AC activates a new alarm. See `on_alarm_start Trigger`_.
- **on_alarm_end** (*Optional*, `Automation <https://esphome.io/guides/automations#automation>`_): (supported only by hOn) Automation to perform when AC deactivates a new alarm. See `on_alarm_end Trigger`_.
- **on_status_message** (*Optional*, `Automation <https://esphome.io/guides/automations#automation>`_): Automation to perform when status message received from AC. See `on_status_message Trigger`_.
- All other options from `Climate <https://esphome.io/components/climate/index.html#config-climate>`_.

Automations
-----------

.. _haier-on_alarm_start:

``on_alarm_start`` Trigger
**************************

This automation will be triggered when a new alarm is activated by AC. The error code of the alarm will be given in the variable ``code`` (``uint8_t``), error message in the variable ``message`` (``const char *``). Those variables can be used in `lambdas <https://esphome.io/guides/automations#config-lambda>`_.

.. code-block:: yaml

    climate:
      - protocol: hon
        on_alarm_start:
          then:
            - logger.log:
                level: WARN
                format: "Alarm activated. Code: %d. Message: \"%s\""
                args: [ 'code', 'message' ]

.. _haier-on_alarm_end:

``on_alarm_end`` Trigger
************************

This automation will be triggered when a previously activated alarm is deactivated by AC. The error code of the alarm will be given in the variable ``code`` (``uint8_t``), error message in the variable ``message`` (``const char *``). Those variables can be used in `lambdas <https://esphome.io/guides/automations#config-lambda>`_.

.. code-block:: yaml

    climate:
      - protocol: hon
        on_alarm_end:
          then:
            - logger.log:
                level: INFO
                format: "Alarm deactivated. Code: %d. Message: \"%s\""
                args: [ 'code', 'message' ]

.. _haier-on_status_message:

``on_status_message`` Trigger
*****************************

This automation will be triggered when component receives new status packet from AC. Raw message binary (without header and checksum) will be provided in the variable ``data`` (``const char *``), message length in the variable ``data_size`` (``uint8_t``). Those variables can be used in `lambdas <https://esphome.io/guides/automations#config-lambda>`_.
This trigger can be used to support some features that unique for the model and not supported by others.

.. code-block:: yaml

    climate:
      - protocol: hon
        on_status_message:
          then:
            - logger.log:
                level: INFO
                format: "New status message received, size=%d, subcmd=%02X%02X"
                args: [ 'data_size', 'data[0]', 'data[1]' ]

``climate.haier.power_on`` Action
*********************************

This action turns AC power on.

.. code-block:: yaml

    on_...:
      then:
        climate.haier.power_on: device_id

``climate.haier.power_off`` Action
**********************************

This action turns AC power off

.. code-block:: yaml

    on_...:
      then:
        climate.haier.power_off: device_id

``climate.haier.power_toggle`` Action
*************************************

This action toggles AC power

.. code-block:: yaml

    on_...:
      then:
        climate.haier.power_toggle: device_id

``climate.haier.display_on`` Action
***********************************

This action turns the AC display on.

.. code-block:: yaml

    on_...:
      then:
        climate.haier.display_on: device_id

``climate.haier.display_off`` Action
************************************

This action turns the AC display off.

.. code-block:: yaml

    on_...:
      then:
        climate.haier.display_off: device_id

``climate.haier.health_on`` Action
**********************************

Turn on health mode (`UV light sterilization <https://www.haierhvac.eu/en/node/1809>`__).

.. code-block:: yaml

    on_...:
      then:
        climate.haier.health_on: device_id

``climate.haier.health_off`` Action
***********************************

Turn off health mode.

.. code-block:: yaml

    on_...:
      then:
        climate.haier.health_off: device_id

``climate.haier.beeper_on`` Action
**********************************

(supported only by hOn) This action enables beep feedback on every command sent to AC.

.. code-block:: yaml

    on_...:
      then:
        climate.haier.beeper_on: device_id

``climate.haier.beeper_off`` Action
***********************************

(supported only by hOn) This action disables beep feedback on every command sent to AC (keep in mind that this will not work for IR remote commands).

.. code-block:: yaml

    on_...:
      then:
        climate.haier.beeper_off: device_id

``climate.haier.set_vertical_airflow`` Action
*********************************************

(supported only by hOn) Set direction for vertical airflow if the vertical swing is disabled. Possible values: Health_Up, Max_Up, Up, Center, Down, Health_Down.

.. code-block:: yaml

    on_...:
      then:
        - climate.haier.set_vertical_airflow:
          id: device_id
          vertical_airflow: Up

``climate.haier.set_horizontal_airflow`` Action
***********************************************

(supported only by hOn) Set direction for horizontal airflow if the horizontal swing is disabled. Possible values: ``Max_Left``, ``Left``, ``Center``, ``Right``, ``Max_Right``.

.. code-block:: yaml

    on_...:
      then:
        - climate.haier.set_horizontal_airflow:
          id: device_id
          vertical_airflow: Right

``climate.haier.start_self_cleaning`` Action
********************************************

(supported only by hOn) Start `self-cleaning <https://www.haier.com/in/blogs/beat-the-summer-heat-with-haier-self-cleaning-ac.shtml>`__.

.. code-block:: yaml

    on_...:
      then:
        - climate.haier.start_self_cleaning: device_id

``climate.haier.start_steri_cleaning`` Action
*********************************************

(supported only by hOn) Start 56°C steri-cleaning.

.. Generated from esphome-docs/sensor/haier.rst

Haier Climate Sensors
=====================

Additional sensors for Haier Climate device. **These sensors are supported only by the hOn protocol**.


.. raw:: HTML

  <p><a href="./docs/esphome-docs/sensor/images/haier-climate.jpg?raw=true"><img src="./docs/esphome-docs/sensor/images/haier-climate.jpg?raw=true" height="50%" width="50%"></a></p>

.. code-block:: yaml

    # Example configuration entry
    sensor:
      - platform: haier
        haier_id: haier_ac
        outdoor_temperature:
          name: Haier outdoor temperature
        humidity:
          name: Haier Indoor Humidity
        compressor_current:
          name: Haier Compressor Current
        compressor_frequency:
          name: Haier Compressor Frequency
        expansion_valve_open_degree:
          name: Haier Expansion Valve Open Degree
        indoor_coil_temperature:
          name: Haier Indoor Coil Temperature
        outdoor_coil_temperature:
          name: Haier Outdoor Coil Temperature
        outdoor_defrost_temperature:
          name: Haier Outdoor Defrost Temperature
        outdoor_in_air_temperature:
          name: Haier Outdoor In Air Temperature
        outdoor_out_air_temperature:
          name: Haier Outdoor Out Air Temperature
        power:
          name: Haier Power

Configuration variables:
------------------------

- **haier_id** (**Required**, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): The id of haier climate component
- **outdoor_temperature** (*Optional*): Temperature sensor for outdoor temperature.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **humidity** (*Optional*): Sensor for indoor humidity. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **compressor_current** (*Optional*): Sensor for climate compressor current. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **compressor_frequency** (*Optional*): Sensor for climate compressor frequency. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **expansion_valve_open_degree** (*Optional*): Sensor for climate's expansion valve open degree. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **indoor_coil_temperature** (*Optional*): Temperature sensor for indoor coil temperature. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **outdoor_coil_temperature** (*Optional*): Temperature sensor for outdoor coil temperature. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **outdoor_defrost_temperature** (*Optional*): Temperature sensor for outdoor defrost temperature. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **outdoor_in_air_temperature** (*Optional*): Temperature sensor incoming air temperature.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **outdoor_out_air_temperature** (*Optional*): Temperature sensor for outgoing air temperature.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.
- **power** (*Optional*): Sensor for climate power consumption. Make sure that your climate model supports this type of sensor.
  All options from `Sensor <https://esphome.io/components/sensor/index.html#config-sensor>`_.


.. Generated from esphome-docs/binary_sensor/haier.rst

Haier Climate Binary Sensors
============================

Additional sensors for Haier Climate device. **These sensors are supported only by the hOn protocol**.


.. raw:: HTML

  <p><a href="./docs/esphome-docs/binary_sensor/images/haier-climate.jpg?raw=true"><img src="./docs/esphome-docs/binary_sensor/images/haier-climate.jpg?raw=true" height="50%" width="50%"></a></p>

.. code-block:: yaml

    # Example configuration entry
    binary_sensor:
      - platform: haier
        haier_id: haier_ac
        compressor_status:
          name: Haier Outdoor Compressor Status
        defrost_status:
          name: Haier Defrost Status
        four_way_valve_status:
          name: Haier Four Way Valve Status
        indoor_electric_heating_status:
          name: Haier Indoor Electric Heating Status
        indoor_fan_status:
          name: Haier Indoor Fan Status
        outdoor_fan_status:
          name: Haier Outdoor Fan Status

Configuration variables:
------------------------

- **haier_id** (**Required**, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): The id of haier climate component
- **compressor_status** (*Optional*): A binary sensor that indicates Haier climate compressor activity.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.
- **defrost_status** (*Optional*): A binary sensor that indicates defrost procedure activity.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.
- **four_way_valve_status** (*Optional*): A binary sensor that indicates four way valve status.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.
- **indoor_electric_heating_status** (*Optional*): A binary sensor that indicates electrical heating system activity.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.
- **indoor_fan_status** (*Optional*): A binary sensor that indicates indoor fan activity.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.
- **outdoor_fan_status** (*Optional*): A binary sensor that indicates outdoor fan activity.
  All options from `Binary Sensor <https://esphome.io/components/binary_sensor/index.html#base-binary-sensor-configuration>`_.

.. Generated from esphome-docs/text_sensor/haier.rst

Haier Climate Text Sensors
==========================

Additional sensors for Haier Climate device. **These sensors are supported only by the hOn protocol**.

.. code-block:: yaml

    # Example configuration entry
    text_sensor:
      - platform: haier
        haier_id: haier_ac
        appliance_name:
          name: Haier appliance name
        cleaning_status:
          name: Haier cleaning status
        protocol_version:
          name: Haier protocol version

Configuration variables:
------------------------

- **haier_id** (**Required**, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): The id of haier climate component
- **appliance_name** (*Optional*): A text sensor that indicates Haier appliance name.
  All options from `Text Sensor <https://esphome.io/components/text_sensor/index.html#base-text-sensor-configuration>`_.
- **cleaning_status** (*Optional*): A text sensor that indicates cleaning status. Possible values "No cleaning", "Self clean", "56°C Steri-Clean".
  All options from `Text Sensor <https://esphome.io/components/text_sensor/index.html#base-text-sensor-configuration>`_.
- **protocol_version** (*Optional*): A text sensor that indicates Haier protocol version.
  All options from `Text Sensor <https://esphome.io/components/text_sensor/index.html#base-text-sensor-configuration>`_.

.. Generated from esphome-docs/button/haier.rst

Haier Climate Buttons
=====================

Additional buttons for Haier AC cleaning. **These buttons are supported only by the hOn protocol**.

.. code-block:: yaml

    # Example configuration entry
    button:
      - platform: haier
        haier_id: haier_ac
        self_cleaning:
          name: Haier start self cleaning
        steri_cleaning:
          name: Haier start 56°C steri-cleaning

Configuration variables:
------------------------

- **haier_id** (**Required**, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): The id of Haier climate component
- **self_cleaning** (*Optional*): A button that starts Haier climate self cleaning.
  All options from `Button <https://esphome.io/components/button/index.html#base-button-configuration>`_.
- **steri_cleaning** (*Optional*): A button that starts Haier climate 56°C Steri-Clean.
  All options from `Button <https://esphome.io/components/button/index.html#base-button-configuration>`_.

.. Generated from esphome-docs/switch/haier.rst

Haier Climate Switches
======================

Additional switches to support additional features for Haier AC.

.. code-block:: yaml

    # Example configuration entry
    switch:
      - platform: haier
        beeper:
          name: Haier beeper
        health_mode:
          name: Haier health mode
        display:
          name: Haier display
        quiet_mode:
          name: Haier quiet mode

Configuration variables:
------------------------

- **haier_id** (**Required**, `ID <https://esphome.io/guides/configuration-types.html#config-id>`_): The id of Haier climate component
- **beeper** (*Optional*): (supported only by hOn) A switch that enables or disables Haier climate sound feedback.
  All options from `Switch <https://esphome.io/components/switch/index.html#base-switch-configuration>`_.
- **health_mode** (*Optional*): A switch that enables or disables Haier climate health mode (`UV light sterilization <https://www.haierhvac.eu/en/node/1809>`__).
  All options from `Switch <https://esphome.io/components/switch/index.html#base-switch-configuration>`_.
- **display** (*Optional*): A switch that enables or disables Haier climate led display.
  All options from `Switch <https://esphome.io/components/switch/index.html#base-switch-configuration>`_.
- **quiet_mode** (*Optional*): (supported only by hOn) A switch that enables or disables Haier climate quiet mode. Quiet mode not supported in Fan only mode.
  All options from `Switch <https://esphome.io/components/switch/index.html#base-switch-configuration>`_.

.. Generated from esp32_backup.rst

How to backup the original image and flash ESPHome to the ESP32 Haier module
============================================================================

**It is strongly recommended to make a backup of the original flash
content before flashing ESPHome!**

To make a backup and to flash the new firmware you will need to use a
USB to TTL converter and solder wires to access UART0 on board (or use
something like this: `Pogo Pin Probe Clip 2x5p 2.54
mm <https://www.tinytronics.nl/shop/en/tools-and-mounting/measuring/accessories/test-probe-with-clamp-pogo-pin-2x5p>`__)

**UART0 pinout:**

.. raw:: HTML

  <p><a href="./docs/esphome-docs/climate/images/haier_pinout.jpg?raw=true"><img src="./docs/esphome-docs/climate/images/haier_pinout.jpg?raw=true" height="70%" width="70%"></a></p>

To put the device in the flash mode you will need to shortcut GPIO0 to
the ground before powering the device.

Once the device is in flash mode you can make a full backup of the
original firmware in case you would like to return the module to its
factory state. To make a backup you can use
`esptool <https://github.com/espressif/esptool>`__. Command to make a
full flash backup:

**python esptool.py -b 115200 –port read_flash 0x00000 0x400000
flash_4M.bin**

After this, you can flash firmware using ESPHome tools (dashboard,
website, esphome command, etc)
.. Generated from additional_information.rst

Additional information
======================

- `FAQ <./docs/faq.rst>`_
- `HaierProtocol <https://github.com/paveldn/HaierProtocol>`_
- `Haier smart modules <./docs/haier_modules.rst>`_
- `Haier protocol overview <./docs/protocol_overview.rst>`_
- `Example of climate configuration for smartair2 protocol <./docs/smartair2_example.rst>`_
- `Example of climate configuration for hOn protocol <./docs/hon_example.rst>`_
- `ESPHome Haier Climate <https://esphome.io/components/climate/haier.html>`_
- `ESPHome Haier Climate Sensors <https://esphome.io/components/sensor/haier.html>`_
- `ESPHome Haier Climate Binary Sensors <https://esphome.io/components/binary_sensor/haier.html>`_
- `Esptool.py Documentation <https://docs.espressif.com/projects/esptool/en/latest/esp32/>`_
- `Sniffing serial communication <./docs/sniffing_serial_communication.rst>`_