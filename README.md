## EEPROM Layout

The layout presented below is retrived from the one suggested by Robert Joerdens in Quartiq deployment tools. The layout is backwards compatible and it will be abided by future ARTIQ tools.

Board and vendor IDs are their positions from the corresponding lists (see [sinara.py](sinara.py)). Important notes are placed in [HardwareLog.md](HardwareLog.md) as well.

| Bytes | Field | Description |
| ----: | ----: | ----------- |
| 4 | CRC32 |control sum|
| 2 | magic | 0x391E (control value) |
| 10 | name | human readable board name |
| 2 | board | board ID |
| 1 | data_rev | EEPROM data format rev. |
| 1 | major | board major rev. |
| 1 | minor | board minor rev. |
| 1 | variant | board variant (e.g. Urukul variant) |
| 1 | port | port number the board is connected to (beware: it's FTDI's port number and it's rather "whole-setup-specific")|
| 1 | vendor | vendor ID |
| 8 | vendor_data | vendor-specific |
| 16 | project_data | reserved for SINARA-HW; currently unused|
| 16 | user_data | up to each user |
| 64 | board_data | board-specific (e.g. in Urukul to store calibration seeds) |
| 122 | padding | default value: b'\xff' each byte |
| 6 | EUI-48 | EEPROM EUI-48 |

## Using
  * Make sure the I2C lines work on Kasli board and connected modules correctly first. 
  * Make sure all the required packages are installed. To do so, run Python virtualenv and install packages:

    ```bash
    $ pip3 install -r requirements.txt
    ```
    Please note that PyFTDI has undergone some code changes recently, so installing another package version may cause everything not to work properly, or even not work at all.

## Flashing EEPROM
To flash board's EEPROM with given contents, one may use _flash_ee.py_ module. However, it may need some adjustment, because it uses FTDI's serial number to open connection with approperiate device. 

The easiest modification for the module to work is to modify url. Using ``url="ftdi://ftdi:4232h:/2"`` will cause the module to auto-detect connected FTDI and open connection with it. For more URL formats supported by PyFTDI see module's [docs](https://eblot.github.io/pyftdi/urlscheme.html).

To write values to approperiate fields modify _ee_data_ accordingly to your needs (for all possible fields, refer to SinaraTuple in [sinara.py](sinara.py))

Additionally, modify _bus.enable()_ argument accordingly to the EEM connector Sinara board is connected. Default value (``"LOC0"``) refers to the EEPROM on the Kasli board.

## Example
To flash Zotino v2.1 connected to EEM1 one has to:
  * modify URL to FTDI, as described above
  * modify _ee_data_ variable, e.g.:
  ```python
    serial = 1234
    ee_data = Sinara(
        name="Zotino",
        board=Sinara.boards.index("Zotino"),
        data_rev=0, major=2, minor=1, variant=0, port=0,
        vendor=Sinara.vendors.index("invalid"),
        vendor_data=serial.to_bytes(8, "big"))
  ```
  * change _bus.enable()_ argument:
  ```python
    bus.enable("EEM1")
  ```
  * activate venv and run:
  ```bash
    $ python3 flash_ee.py
  ```
  If something went wrong, the script will let you know.