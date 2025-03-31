![snapshot](https://github.com/eman8388/ETHlink8ADC/blob/main/snapshot_PCB/3D_ETHlink8xADC_PCB_3D_1.png)

# ETHlink8ADC

ETHlink8ADC is an open-source, open-hardware 8-input Ethernet audio interface, remotely controlled, and part of the ETHlink audio interface family.

## Main Features

- **No Driver Required**: The ETH8ADCV2 VST3 plugin enables direct audio streaming to your DAW. Project: https://github.com/eman8388/ETH8ADCV2 .
- **Scalability**: Supports up to 32 devices on a local network, depending on network quality.
- **Ultra-Low Latency**: Only 8 samples per buffer.
- **Unique Identification**: Each device is identified on the network via a unique serial number and MAC address.
- **Minimalist Design**: No controls, no display—just a dual-color status LED and two phantom power LEDs!
- **Software-Controlled Gain, Mute, Phantom power**
- **Powerful Hardware**:
  - STM32H743 high-performance single-core Arm Cortex-M7 MCU
  - Ethernet PHY LAN8742 in 100-BaseTX mode
  - ADC6140 ADC featuring:
    - 32-bit resolution with support for 44.1kHz, 48kHz, 88.2kHz, and 96kHz sample rates
    - integrated PGA input fully balaced for reduced PCB footprint and enhanced precision
  - High-precision clock generation using the Si5351 arbitrary clock synthesizer
  - Dual stage Microphone preamplifier for each channel:
    - Full digital controlled mic preamp design based on pga + instrumentation amplifier with I2C controlled digipot
    - High input impedence design for high sensibility ( SM58, 0dB gain set, peak at -15dbFS)
    - tasty saturation when overcome -6dbFS
    - **Ridiculus +80dB MAX GAIN** selectable with 1 db steps (from 0 to +80db)
    - Clip at +6dBv
    - -136dbFS noise floor at 0db  
  - High current phantom power selectable for channel 1-4 5-8
- **Flexible Power Options**: PoE-powered or 12V-15V DC jack (<1A).
- **Optimized for Efficiency**: Rail-to-rail power supply, dual 3.3V LDO regulators for analog and digital rails, improving efficiency and reducing noise, phantom power made from input voltage with LT8330 boost converter
- **Compact Design**: Everything fits on a dual-layer 120x90 mm PCB.

## Hardware

The PCB is cost-optimized with a simplified architecture to improve manufacturability.

- **Schematic:**\
  [View Schematic](https://github.com/eman8388/ETHlink8ADC/blob/main/PCB_Schematic/SCH_ETHlink8xADC_Schematic.pdf)

- **EasyEDA Project:**\
  [View Project](https://github.com/eman8388/ETHlink8ADC/tree/main/PCB_EasyEDAPro_Project)

- **Bill of Materials (BOM):**\
  [View BOM](https://github.com/eman8388/ETHlink8ADC/tree/main/PCB_BOM)

- **Gerber Files:**\
  [Download Gerber Files](https://github.com/eman8388/ETHlink8ADC/blob/main/PCB_fabrication_file/Gerber_ETHlink8xADC_PCB.zip)

### Known Hardware Issues

- Depending on the PCB layout, TDM signals (BCK, FS, DATA\_OUT) may require impedance compensation. In my case, i don't have to add anything (R150 R151 R152 not populated).
- The resistor values for the status LED (R148, R149) may need to be adjusted based on the LED type.

### Protorype Application Device

![View Example](https://github.com/eman8388/ETHlink8ADC/blob/main/snapshot_device_example/Image%202025-03-31%20at%2012.25.09.jpeg)

## Firmware

Developed in **C** using **STM32IDE**, based on **FreeRTOS**.

- **Supported MCUs:** STM32H743VIT6 or the cost-effective STM32H750VBT6 (128KB ROM, requires project regeneration).
- **MAC Address Assignment:**
  - `00:E8:AD:C0:00:00` → Serial number 0
  - `00:E8:AD:C0:00:01` → Serial number 1
  - The last two bytes of the MAC address represent the device's serial number (16-bit value).

### Communication Protocol

ETHlink8DAC communicates with the computer using **Raw Ethernet Frames (Layer-2)** with a custom protocol (`0x0802`). The only network identifier is the **MAC address**.

- The device receives a packet containing:
  - **64-byte header** (containing commands)
- The device:
  1. Processes the command.
  2. Sends audio data recorded via the TDM interface.
  3. If no command packet is received within one second, communication stops.

**Packet Structure:**

```
[ETHERNET LAYER 2 HEADER (14 BYTES)] - [APPLICATION HEADER (50 BYTES)] - [AUDIO DATA (8 CHANNELS × 8 SAMPLES × 4 BYTES PER SAMPLE)]
```

For details, refer to **main.c** and **ethernetif.c**.

## Flashing Instructions

1. **Download and extract** `ETH8ADC_STM32IDE_FILES.zip`.
2. **Open STM32IDE** and create a new project using `ETH8ADC.ioc` configuration.
3. **Replace project files** with those from the `ETH8ADC_STM32IDE_FILES` folder.
4. **Modify MAC Address**:
   - Open `ethernetif.c` (under `LWIP->TARGET`).
   - Locate `static void low_level_init(struct netif *netif);`.
   - Update the `MACAddr[]` array with your desired address.
5. **Optimize the build**:
   - Right-click on the project → Properties → `C/C++ Build` → Settings → `MCU/MPU GCC Compiler` → Optimization → Set to `-Ofast`.
6. **Connect ST-Link V2**:
   - Pin header at the **top-left of the board**.
   - Connect **GND, SWDIO, SWCLK** (Pins 4, 1, 2).
   - Build and flash the project.

For testing, a **precompiled ELF binary** is available in the `TEST` folder with serial numebr 0, ready to be flashed with **STM32CubeProgrammer**.

---

Eugenio Mancini\
 [mancini97email@gmail.com](mailto\:mancini97email@gmail.com)


