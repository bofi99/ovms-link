# README

[A Better Routeplanner (ABRP)](https://abetterrouteplanner.com) is an electric
vehicle (EV) focussed route planner that incorporates planning of EV charging
stops.

This OVMS plugin sends live telemetry data from the vehicle to ABRP to be used
for the route planning process and appropriate updates to the plan along the way
based on live information.

This version is for the Kia Soul EV 30kWh.
The SOH calculation is not correct in OVMS and leads to the wrong calculation of the reference consumption in ABRP.
The SOH is replaced with a manually entered value. Currently only the KIA workshop and the SoulSpy app can read the SOH for the 30 kWh version.

## Installation
Extra step for this version:
Enter the correct SOH value in line 196 of the abrp.js file

### Obtain Live Data Token

1. Register with [A Better Routeplanner (ABRP)](https://abetterrouteplanner.com)
   and login
2. Setup your vehicle, starting with **Select car model**
3. In the settings for the new vehicle, click on the **Live data** button to
   generate a generic token. Keep a record of this token

### Install the abrp.js Plugin in OVMS

1. Login to the
   [OVMS web console](https://docs.openvehicles.com/en/latest/userguide/installation.html#initial-connection-wifi-and-browser)
2. Navigate to the **Tools** -> **Editor** menu item
3. Create a new `lib` directory in `/store/scripts` if it does not exist
4. Create a new `abrp.js` file in the `/store/scripts/lib` directory
5. Copy the content of the `lib/abrp.js` file in this repository to that new
   file and save
6. Create a new `ovmsmain.js` file in `/store/scripts` if it does not exist
7. Copy the content of the `ovmsmain.js` file in this repository to that new
   file and **Save**

### Install the Google Subordinate CA Certificate in OVMS

A recent change to the APRB API means a new CA certificate is required. This 
requires a Google subordinate CA certificate  ([GTS CA 1P5](https://pki.goog/repo/certs/gts1p5.pem)) 
to be installed into OVMS as per the instructions at the [OVMS SSL/TLS UserGuide](https://docs.openvehicles.com/en/latest/userguide/ssltls.html)

The [GTS CA 1P5](https://pki.goog/repo/certs/gts1p5.pem) can be downloaded from the [Google Trust Services site](https://pki.goog/repository/).

1. Login to the
   [OVMS web console](https://docs.openvehicles.com/en/latest/userguide/installation.html#initial-connection-wifi-and-browser)
2. Navigate to the **Tools** -> **Editor** menu item
3. Create a new `trustedca` directory in `/store/` if it does not exist
4. Create a new `gts1p5.pem` file in the `/store/trustedca` directory
5. Copy the contents of the `gts1p5.pem` file into that file
6. Navigate to the **Tools** -> **Shell** menu item
7. Execute the following message: `tls trust reload`
8. Execute the following message: `tls trust list` and confirm that
   `gts1p5.pem` shows up in the list

### Configure Plugin

1. Navigate to **Tools** -> **Shell** in the OVMS web console
2. In the OVMS shell issue the following command substituting `<token>` with the
   live data generic token that was set up for the vehicle in ABRP

   ```text
   config set usr abrp.user_token <token>
   ```

### Reload the JS Engine

1. Navigate to **Tools** -> **Editor** in the OVMS web console and press the
   **Reload JS Engine** button. This should result in an `ABRP::started`
   notification if the plugin is installed and configured correctly.

## Usage

With the configuration described above the ABRP plugin will automatically send
live telemetry from the vehicle to ABRP on a periodic basis. When data is sent
depends on what is happening for the vehicle. For example, when driving,
information will be sent more frequently than when charging, and even less often
when the car is off.

### OVMS Shell Commands

- `script eval abrp.info()` - display vehicle telemetry that would be sent to
  ABRP
- `script eval abrp.onetime()` - send current telemetry to ABRP once only
- `script eval abrp.send(1)` - start periodically sending telemetry to ABRP
  (when necessary)
- `script eval abrp.send(0)` - stop sending telemetry
- `script eval abrp.resetConfig()` - reset configuration
