### Overview
* SX1276 is a LoRa modem that can send or receive data over a long distance.
* Many ESP32 LoRa development boards are using this modem: Heltec WiFi LoRa 32 V2, TTGO T-Beam V1.1 
* Adafruit also created a standalone breakout: Adafruit RFM95W
* One key thing is they all use SPI as the control interface of LoRa modem.
* SPI pins on RFM95W are exposed so we can hook it up with Raspberry Pi Pico while pins are marked on other two ESP32 LoRa development boards.
### See the wiring

### How to use SX1276
* Enable the module if use Adafruit RFM95W
* Configure SPI communication to control the LoRa modem
* Choose LoRa Modem other than FSK/OOK Modem
* Set bandwidth, coding rate, header mode, spreading factor, preamble length, frequency, amplifier
* Set an interrupt routine service to read incoming message and to monitor modem's working status
* Write FIFO data buffer when transmit and read when receive.
### Jargon in [Datasheet](DS_SX1276-7-8-9_W_APP_V7.pdf)
* RF: Radio Frequency
* RFI: RF Input
* RFO: RF Output
* { HF: {Band 1: ~915MHz}, LF: {Band 2: ~433MHz, Band 3: ~150MHz} }
* PA: Power Amplifier
* Three amplifiers: RFO_LF, RFO_HF, PA_BOOST
* PA_HP: High Power
* PA_HF and PA_LF are high efficiency amplifiers
* AFC: automatic frequency correction
* RFOP: RF output power
### 4.1.2. LoRa ® Digital Interface
* The LoRa ® modem comprises three types of digital interface,
  * static configuration registers
  * status registers
  * a 256-byte user-defined FIFO data buffer
### Data Transmission Sequence (Figure 9)
* Change to Standby mode so the modem initiate everything
* Start Tx loop
  * Prepare payload to Tx
  * Fill FIFO data buffer with payload
  * Change to Tx mode
  * Wait for TxDone IRQ
    * In ISR, do something and clear IRQ Flags
  * Fall back to Standby mode automatically
### Continous Mode Data Reception Sequence (Figure 10)
* Change to Standby mode so the modem initiate everything
* Change to Rx Continuous mode
* Wait for IRQ (RxDone and ValidHeader/PayloadCrcError)
  * In ISR, Read FIFO data buffer to get payload
* Next IRQ
### Packet Structure
<img src="Packet_Structure.png"></img>
* Header (exists in explicit mode): Payload length, payload's coding rate (Tx tells Rx which CR it uses)
* Explicit header's coding rate is 4/8 and payload's could be different.
* SF is for whole packet
<img src="Packet_Structure_Waterfall.jpg"></img>
### FIFO Buffer
<img src="FIFO_Buffer.png"></img>
* In order to write packet data into FIFO user should:
  1. Set RegFifoAddrPtr to *RegFifoTxBaseAddr.
  2. Write *RegPayloadLength bytes to the FIFO (RegFifo)
* In order to read packet data from FIFO user should:
  1. Set RegFifoAddrPtr to *RegFifoRxCurrentAddr.
  2. Read RegRxNbBytes from RegFifo
### Follow code to learn SX1276
* These codes are commented extensively for learning
* [Tx](SX1276_Tx.py) [Rx](SX1276_Rx.py)
* Thanks [martynwheeler/u-lora](https://github.com/martynwheeler/u-lora)
