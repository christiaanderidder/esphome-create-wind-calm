# CREATE WIND CALM ceiling fan ESPHome mod
This repository contains a guide and configuration files for converting the Create Wind Calm fan from Tuya Smart to ESPHome.
For more information on this fan and some experiences using it, see [https://github.com/velzend/create_ikohs_fan](https://github.com/velzend/create_ikohs_fan).
Most of the information in this guide came from [an issue on velzend's git repo](https://github.com/velzend/create_ikohs_fan/issues/7) and my own experiences.

**I am not resposible for any damage you might do to your device, nor will I provide any support besides the information given below**

The [CREATE WIND CALM](https://www.create-store.com/nl/kopen-plafondventilatoren-zonder-lamp/82468-wind-calm-plafondventilator-40w-silent-o132-cm.html) (also known as IKOHS) ceiling fan is Tuya based device, which means that it requires cloud connectivity to be able to be controlled remotely.

For Tuya devices there are generally two ways to make the device local only:
- Using local integrations that speak the tuya protocol like [localtuya](https://github.com/rospogrigio/localtuya), [tuya-local](https://github.com/make-all/tuya-local), [tinytuya](https://github.com/jasonacox/tinytuya) etc.
- By modding the device to use ESPHome

## Tuya Datapoints
Tuya devices expose a number of data points which represent different features of a tuya device. These are relevant for either type of integration. The data points for the CREATE WIND CALM can be found in the [tuya](tuya) folder in this repository.

## Local integrations
While these generally work, they have a big downside. It requires registering the device with the Tuya Smart app, registering as a developer on the Tuya cloud platform and finally blocking internet access from the device. All in all it works, but it is not fully local only.
Initially this was the integration method I used, but it had a lot of intermittent connection issues, where then fan just started refusing connectings after a certain amount of time. All in all it will work in some cases, and might be the solution if you are not comfortable physicially modifying your device.

## Modding to ESPHome
The better solution, in my eyes, is to fully eliminate the Tuya Cloud connectivity and switch over to ESPHome instead.
For some devices this is as easy as flashing ESPHome directly to the tuya provided board using [libretiny](https://docs.libretiny.eu/).
In the case of this fan this is not possible as it uses the WBR3 board which is [not supported](https://docs.libretiny.eu/docs/status/supported/#unsupported-boards) by libretiny.
However, the [WBR3](https://developer.tuya.com/en/docs/iot/wbr3-module-datasheet?id=K9dujs2k5nriy) board is fully pin-compatible with an [ESP-12F](https://docs.ai-thinker.com/_media/esp8266/docs/esp-12f_product_specification_en.pdf), which makes replacing it relatively easy.

## Replacing the WBR3 with ESP-12F
First of all it's important to be aware that there are multiple versions of this fan. Some variants have a dimmable light, while my fan has not. The exact motor control unit and RF remote used also seem to differ between units. This is most likely due to CREATE just using any parts available off the shelve.

Make sure to flash your ESP-12F with ESPHome before continuing. In my case I used a [development board](https://www.tinytronics.nl/en/development-boards/accessories/adapter-boards/development-board-for-esp8266-wi-fi-module) which makes this very quick and easy without any soldering. 
The [esphome](esphome) folder in this repository contains the ESPHome yaml for the fan, carefully check this file and make sure you set up the correct secrets and IP addresses.

When opening the motor control unit, it should have a main board with 2 small daughter boards soldered on using through-hole headers. One of these boards has the WBR3 soldered to it, the other is the RF receiver for the remote.
The WBR3 daughter board can easily be desoldered to make working on replacing the WBR3 a little easier. This is also the perfect time to remove the speaker (or put a small amount of glue in the top hole) to get rid of the annoyingly loud beeping sound the fan makes.

<img src="https://github.com/user-attachments/assets/ce8caa76-2155-4edd-ba73-7b7db52eb295" width="250" />
<img src="https://github.com/user-attachments/assets/3d6e0a4e-d0f2-47d6-b297-649c8fe9524f" width="250" />
<img src="https://github.com/user-attachments/assets/be020b66-2fc3-4315-ab36-c84492b434d9" width="250" />
<img src="https://github.com/user-attachments/assets/8ddb6150-24a2-4d83-b156-de602178a3f9" width="250" />

1. The motor control unit with the removed daughter board.
2. Rear side of the daughter board, the removed WBR3 and ESP-12F
3. Front side of the daughter board, the removed WBR3 and ESP-12F
4. Wiring from the daughter board to the ESP-12F

When placing the ESP-12F the following connections need to be made:
| ESP-12F  | Daughter Board | Notes |
| -------- | -------------- | ----- |
| TXD | TXD | |
| RXD | RXD | |
| GPIO15 | GND | Set correct boot mode |
| EN | VCC | Always enable, don't use EN on daughter board |
| VCC | VCC | |
| GND | GND | |

As you can see, my first time soldering did not go down so well. I destroyed the pads on the daughter boards and had to use wires instead.
This is not neccesary if the WBR3 is removed a bit more carefully.

In the pictures a wire from GPIO12 can also be seen, but this one is not neccesary, I added it in case I would have to desolder the RF receiver directly to the ESP-12F to keep the remote working as expected.

However, I found out that this is not neccesary, the motor control unit already informs the ESP-12F when a change is made using the remote as long are you are running ESPHome with a small fix (see [below](#fan-speed-not-working)).

After reassembling everything, the fan should start and the ESPHome dashboard should be reachable over the network.

## Home Assitant
If the ESPHome integration is configured in home assistant the device should be auto discovered.

## Known Issues

### Fan speed not working
Multiple people ran into issues setting the fan speed, because the tuya device does not correctly report all data points to ESPHome on startup. I found out that this was caused by ESPHome not processing all information being sent by the tuya device and managed to fix this by making [a small change to ESPHome](https://github.com/esphome/esphome/pull/6980). This fix is available since ESPHome 2024.7, so make sure to use a recent version.

### Fan timer
Datapoint 64 can be used to set the countdown timer, the actual countdown is handled by the motor control unit.
Currently the fan turns off when this timer expires, but there is no feedback to ESPHome.
After the fan turns off, ESPHome can't issue any further commands. Currently, the only way to get the fan working again is to turn it on with the RF remote after the timer expires. After doing so ESPHome works as expected again.
Since this is a hardware issue with the motor control unit, it's probably best to not use this datapoint at all and build a custom timer script in ESPHome.

### Light cycles between 3 light temperatures
When the light is turned off and turned on again quickly, it will automatically cycle to the next of three different light temperatures. This can be extremely annoying, especially considering the intesity of the two brightest settings.
Sadly this is a feature that is baked directly into the LED board and not in any way controllable with this ESPHome conversion. However, it is possible to swap the LED panel for another one that only offers a single shade of white light.
I've personally used the following light as a replacement: [AliExpress](https://www.aliexpress.com/item/1005004103063784.html?spm=a2g0o.order_list.order_list_main.11.a91c79d2zs2K1w).
This light offers just a single shade of warm white light and just requires swapping the original lights magnetic legs, so there is enough space for the wires underneath.
Any comparable magnetic LED of around 11-12cm should do.
