# CREATE WIND CALM ceiling fan ESPHome mod
This repository contains a guide and configuration files for converting the Create Wind Calm fan from Tuya Smart to ESPHome.
Most of the information in this guide came from [an issue on velzend's git repo](https://github.com/velzend/create_ikohs_fan/issues/7) and my own experiences.

**I am not resposible for any damage you might do to your device, nor will I provide any support besides the information given below**

The [CREATE WIND CALM](https://www.create-store.com/nl/kopen-plafondventilatoren-zonder-lamp/82468-wind-calm-plafondventilator-40w-silent-o132-cm.html) (also known as IKOHS) ceiling fan is Tuya based device, which means that it requires cloud connectivity to be able to be controlled remotely.

For Tuya devices there are generally two ways to make the device local only:
- Using local integrations that speak the tuya protocol like [tuya-local](https://github.com/rospogrigio/localtuya), [local-tuya](https://github.com/make-all/tuya-local), [tinytuya](https://github.com/jasonacox/tinytuya) etc.
- By modding the device to use ESPHome

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

When opening the motor control unit, it should have a main board with 2 small daughter boards soldered on using through-hole headers. One of these boards has the WBR3 soldered to it, the other is the RF receiver for the remote.

<img src="https://github.com/user-attachments/assets/ce8caa76-2155-4edd-ba73-7b7db52eb295" width="250" />
<img src="https://github.com/user-attachments/assets/3d6e0a4e-d0f2-47d6-b297-649c8fe9524f" width="250" />

The first image shows the motor control unit with the removed daughter board.

The second image shows the daughter board, the removed WBR3 and pin-compatible ESP-12F.
