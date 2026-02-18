RF Radar System - Technical Briefing
Overview

This project creates a 2.4GHz RF Spectrum Analyzer using an ESP32-C3 and nRF24L01 module, visualizing radio frequency activity as a real-time radar display on a PC using Processing.
System Architecture

┌─────────────────┐    Serial    ┌─────────────────┐
│   ESP32-C3      │   ────────►   │   Processing    │
│  + nRF24L01     │    COM8       │      PC App     │
└─────────────────┘   115200 baud └─────────────────┘

Hardware Components
ESP32-C3 (ESP-C3-32S-Kit)

    Microcontroller: 32-bit RISC-V @ 160MHz

    Role: Master controller, data processing, serial communication

nRF24L01+ Transceiver

    Frequency: 2.4GHz ISM band

    Channels: 126 (2400MHz - 2525MHz)

    Role: RF detector, signal strength measurement

    Pin Connections:
    text

    nRF24L01    →    ESP32-C3
    VCC         →    3.3V
    GND         →    GND
    CE          →    GPIO4
    CSN         →    GPIO5
    SCK         →    GPIO6
    MOSI        →    GPIO7
    MISO        →    GPIO2

Software Components
1. Arduino Code (ESP32-C3)

The firmware that runs on the ESP32:
Key Functions:


setup() 
  ├─ Initializes SPI communication
  ├─ Configures nRF24L01 for maximum sensitivity
  └─ Sets up serial communication

loop()
  ├─ scanChannels() - Sweeps all 126 channels
  ├─ measureSignal() - Samples RF activity 8x per channel
  └─ sendData() - Transmits data to PC via serial

Signal Detection Algorithm:

for each channel (0-125):
  set radio to channel
  enable receiver
  wait 250μs
  take 8 samples of testRPD()
  signal strength = number of positive samples (0-8)
  disable receiver
  store value

Data Format:

RF_DATA:0,0,0,5,3,0,0,8,0,... (126 comma-separated values)

2. Processing Sketch (PC Display)

The visualization software running on PC:
Display Components:

┌─────────────────────────────────────┐
│  RADAR VIEW           SPECTRUM VIEW │
│  ┌────────────┐      ┌───────────┐  │
│  │   ●        │      │  ███      │  │
│  │  ● ● ●     │      │  ███      │  │
│  │   ●        │      │  ███ ███  │  │
│  └────────────┘      └───────────┘  │
│  INFO PANEL                          │
│  Status: ONLINE                      │
│  Active Channels: 12                 │
└─────────────────────────────────────┘

Signal Processing:

    Serial Reception: Reads "RF_DATA:" lines at 115200 baud

    Data Parsing: Splits comma-separated values into array

    Visual Mapping:

        Radar view: Channel → Angle (0-360°), Strength → Radius

        Spectrum view: Channel → X position, Strength → Bar height

    Peak Detection: Maintains maximum values with decay

    Animation: Rotating sweep line at 0.02 rad/frame

How It Works - Step by Step
1. RF Detection

nRF24L01 Antenna
    ↓
Receives 2.4GHz RF energy
    ↓
Internal RPD (Received Power Detector)
    ↓
Digital output: 1 if signal > -64dBm
    ↓
ESP32 reads this 8x per channel

2. Channel Scanning

Channel 0: ████████ (8/8 samples)
Channel 1: ██······ (2/8 samples)
Channel 2: ········ (0/8 samples)
...
Channel 125: ██████·· (6/8 samples)

3. Data Transmission

ESP32 → [RF_DATA:0,2,0,8,5,0,...] → PC via USB

4. Visualization

Signal Strength → Visual Element
    0-1    →  Faint dot
    2-4    →  Medium dot
    5-8    →  Bright dot + peak marker

Key Technical Concepts
Received Power Detector (RPD)

    Built into nRF24L01

    Triggers when signal > -64dBm

    Fast response (~40μs)

    No decoding needed - just energy detection

Multiple Sampling

    8 samples per channel reduces false positives

    Accounts for bursty RF signals (WiFi, Bluetooth)

    Provides crude signal strength indication

Channel Mapping

Channel 0   = 2400 MHz
Channel 1   = 2401 MHz
...
Channel 125 = 2525 MHz

WiFi Channels (reference):
- Channel 1  = 2412 MHz (channel 12)
- Channel 6  = 2437 MHz (channel 37)
- Channel 11 = 2462 MHz (channel 62)

Performance Characteristics
Timing

    Channel scan: ~300μs per channel

    Full sweep: 126 × 300μs = 38ms

    Data transmission: ~2ms

    Update rate: ~20-25 frames/second

Sensitivity

    Detection threshold: -64dBm

    Dynamic range: ~30dB (using multi-sampling)

    Frequency resolution: 1MHz

Accuracy Considerations

    No absolute power measurement (only threshold detection)

    Relative signal strength via sampling count

    Antenna orientation affects detection

    Nearby objects can reflect signals

Applications

    WiFi Network Analysis

        Detect router activity

        Find crowded channels

        Identify interference sources

    RFI Hunting

        Locate microwave oven leaks

        Find Bluetooth devices

        Detect cordless phones

    Educational Tool

        Visualize invisible RF spectrum

        Understand radio communications

        Experiment with antennas
