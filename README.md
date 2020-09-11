# IOB-I2C-MCP23017
## License: CERN Open Hardware Licence v1.2

IOB I2C MCP23017 based IO Expander

The MCP23017 expander is an inexpensive and reliable device that supports up to 8x instances on a single I2C chain.

### Specifications

Each board provides a latched set of 16x IO points, with each point
being software selectable to be either an input or an output. The
individual points are reasonably protected from the environment, and can
source or sink 20 mA each (25mA absolute max).

### Communication Protocol

```C++
    enum MCP23017Registers {
        MCP23017_IODIRA   = 0x00,
        MCP23017_IODIRB   = 0x01,
        MCP23017_IPOLA    = 0x02,
        MCP23017_IPOLB    = 0x03,
        MCP23017_GPINTENA = 0x04,
        MCP23017_GPINTENB = 0x05,
        MCP23017_DEFVALA  = 0x06,
        MCP23017_DEFVALB  = 0x07,
        MCP23017_INTCONA  = 0x08,
        MCP23017_INTCONB  = 0x09,
        MCP23017_IOCONA   = 0x0A,
        MCP23017_IOCONB   = 0x0B,
        MCP23017_GPPUA    = 0x0C,
        MCP23017_GPPUB    = 0x0D,
        MCP23017_INTFA    = 0x0E,
        MCP23017_INTFB    = 0x0F,
        MCP23017_INTCAPA  = 0x10,
        MCP23017_INTCAPB  = 0x11,
        MCP23017_GPIOA    = 0x12,
        MCP23017_GPIOB    = 0x13,
        MCP23017_OLATA    = 0x14,
        MCP23017_OLATB    = 0x15,
    };

void I2Cexpander::init23017(uint8_t i2caddr, uint16_t dir) {
    if (i2caddr < base23017) {
        _i2c_address = base23017 + i2caddr;
    } else {
        _i2c_address = i2caddr;
    }
    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_IODIRA);
    Wire.write(0xff & dir);         // Low byte
    Wire.endTransmission();

    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_IODIRB);
    Wire.write(0xff & (dir >> 8));  // High byte
    Wire.endTransmission();

    // enable 100k pullups on all inputs...
    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_GPPUA);
    Wire.write(0xff & dir);         // Low byte
    Wire.endTransmission();

    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_GPPUB);
    Wire.write(0xff & (dir >> 8));  // High byte
    Wire.endTransmission();
}


uint32_t I2Cexpander::read23017() {
    uint32_t data = 0;
    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_GPIOA);
    Wire.endTransmission();

    Wire.requestFrom(_i2c_address, (uint8_t)2, (uint8_t)1);
    data = Wire.read();
    data |= (Wire.read() << 8);
    return data;
}

void I2Cexpander::write23017(uint32_t data) {
    data = data | _config;
    Wire.beginTransmission(_i2c_address);
    Wire.write(MCP23017_GPIOA);
    Wire.write(0xff & data);  //  low byte
    Wire.write(data >> 8);    //  high byte
    Wire.endTransmission();
}
```

See [I2Cexpander-lib](/pages/I2Cexpander "wikilink") for a more complete interface
library.


### Addressing
The addres selection for the MCP23017 is simple - set the desired address (in binary) on the 3x address selection switches.

**AD2**|**AD1**|**AD0**|**A6**|**A5**|**A4**|**A3**|**A2**|**A1**|**A0**|**ADDRESS (HEX)**
:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:
GND|GND|GND|0|1|0|0|0|0|0|0x20
GND|GND|V+|0|1|0|0|0|0|1|0x21
GND|V+|GND|0|1|0|0|0|1|0|0x22
GND|V+|V+|0|1|0|0|0|1|1|0x23
V+|GND|GND|0|1|0|0|1|0|0|0x24
V+|GND|V+|0|1|0|0|1|0|1|0x25
V+|V+|GND|0|1|0|0|1|1|0|0x26
V+|V+|V+|0|1|0|0|1|1|1|0x27


