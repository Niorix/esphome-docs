Modbus Binary Sensor
====================

.. seo::
    :description: Instructions for setting up a modbus_controller device binary sensor.
    :image: modbus.png

The ``modbus_controller`` binary sensor platform creates a binary sensor from a modbus_controller component
and requires :doc:`/components/modbus_controller` to be configured.


Configuration variables:
------------------------

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID used for code generation.
- **name** (**Required**, string): The name of the sensor.
- **modbus_functioncode** (**Required**): type of the modbus register.
    - "read_coils": Function 01 (01hex) Read Coils - Reads the ON/OFF status of discrete coils in the device.
    - "read_discrete_inputs": Function 02(02hex) - Reads the ON/OFF status of discrete inputs in the device.
    - "read_holding_registers": Function 03 (03hex) Read Holding Registers - Read the binary contents of holding registers in the device.
    - "read_input_registers": Function 04 (04hex) Read Input Registers - Read the binary contents of input registers in the device.

- **address**: (**Required**, int): start address of the first register in a range
- **bitmask** : some values are packed in a response. The bitmask is used to determined if the result is true or false
- **skip_updates**: (*Optional*, int): By default all sensors of of a modbus_controller are updated together. For data points that don't change very frequently updates can be skipped. A value of 5 would only update this sensor range in every 5th update cycle
- **register_count**: (*Optional*, int): only required for uncommon response encodings or to :ref:`optimize modbus communications<modbus_register_count>`
  The number of registers this data point spans. Overrides the defaults determined by ``value_type``.
  If no value for ``register_count`` is provided, it is calculated based on the register type.

  The default size for 1 register is 16 bits (1 Word). Some devices are not adhering to this convention and have registers larger than 16 bits.  In this case ``register_count`` and  ``response_size`` must be set. For example, if your modbus device uses 1 registers for a FP32 value instead the default of two set ``register_count: 1`` and ``response_size: 4``.
- **response_size**:  (*Optional*, int): Size of the response for the register in bytes. Defaults to register_count*2.
- **force_new_range**: (*Optional*, boolean): If possible sensors with sequential addresses are grouped together and requested in one range. Setting ``force_new_range: true`` enforces the start of a new range at that address.
- **custom_data** (*Optional*, list of bytes): raw bytes for modbus command. This allows using non-standard commands. If ``custom_data`` is used ``address`` and ``register_type`` can't be used.
  custom data must contain all required bytes including the modbus device address. The crc is automatically calculated and appended to the command.
  See :ref:`modbus_custom_data` how to use ``custom_command``
- **lambda** (*Optional*, :ref:`lambda <config-lambda>`):
  Lambda to be evaluated every update interval to get the new value of the sensor
- **offset**: (*Optional*, int): not required for most cases
  offset from start address in bytes. If more than one register is read a modbus read registers command this value is used to find the start of this datapoint relative to start address. The component calculates the size of the range based on offset and size of the value type
  The value for offset depends on the register type. If a binary_sensor is created from an input register the offset is in bytes. For coil and discrete input resisters the LSB of the first data byte contains the coil addressed in the request. The other coils follow toward the high-order end of this byte and from low order to high order in subsequent bytes. For the registers  offset is the position of the relevant bit.
  To get the value of the coil register 2 can be retrived using address: 2 / offset: 0 or address: 0 / offset 2


Example

.. code-block:: yaml

    binary_sensor:
    - platform: modbus_controller
      modbus_controller_id: epever
      id: battery_internal_resistance_abnormal
      name: "Battery internal resistance abnormal"
      register_type: read
      address: 0x3200
      bitmask: 0x80 #(bit 8)


Parameters passed into the lambda

- **x** (bool): The parsed float value of the modbus data
- **data** (std::vector<uint8_t): vector containing the complete raw modbus response bytes for this sensor
- **item** (const pointer to a ModbusBinarySensor object):  The sensor object itself.

Possible return values for the lambda:

 - ``return true/false;`` the new value for the sensor.


See Also
--------
- :apiclass:`:modbus_controller::ModbusBinarySensor`
- :doc:`/components/modbus_controller`
- :doc:`/components/switch/modbus_controller`
- :doc:`/components/output/modbus_controller`
- :doc:`/components/sensor/modbus_controller`
- :doc:`/components/text_sensor/modbus_controller`
- https://www.modbustools.com/modbus.html
- :ghedit:`Edit`
