---
layout: page
title: "Emulated Hue Bridge"
description: "Instructions how to emulated Hue Bridge within Home Assistant."
date: 2016-08-26 08:00
sidebar: true
comments: false
sharing: true
footer: true
logo: home-assistant.png
ha_category: Hub
ha_release: 0.27
ha_iot_class: "Local Push"
---

The `emulated_hue` component provides a virtual Philips Hue bridge, written entirely in software, that allows services that work with the Hue API to interact with Home Assistant
entities. The driving use case behind this functionality is to allow Home Assistant to work with an Amazon Echo or Google Home with no set up cost outside of configuration changes.
The virtual bridge has the ability to turn entities on or off, or change the brightness of dimmable lights. The volume level of media players can be controlled as brightness.

<p class='note'>
A physical Hue Bridge is required for the lights to function - this virtual bridge will not replace a physical bridge.
</p>

<p class='note'>
It is recommended to assign a static IP address to the computer running Home Assistant. This is because the Amazon Echo discovers devices by IP addresses, and if the IP changes, the Echo won't be able to control it. This is easiest done from your router, see your router's manual for details.
</p>

### {% linkable_title Configuration %}

To enable the emulated Hue bridge, add one of the following configs to your `configuration.yaml` file:

```yaml
# Google Home example configuration.yaml entry
emulated_hue:
  listen_port: 80
  # Google Home does not work on different ports.
```

```yaml
# Amazon Echo example configuration.yaml entry
emulated_hue:
```

Configuration variables:

- **type** (*Optional*): The type of assistant who we are emulated for. Either `alexa` or `google_home`, defaults to `google_home`. **This configuration option is deprecated and will be removed in a future release. It is no longer necessary to define type.**
- **host_ip** (*Optional*): The IP address that your Home Assistant installation is running on. If you do not specify this option, the component will attempt to determine the IP address on its own.
- **listen_port** (*Optional*): The port the Hue bridge API web server will run on. If not specified, this defaults to 8300. This can be any free port on your system.

- **advertise_ip** (*Optional*): If you need to override the IP address used for UPNP discovery. (For example, using network isolation in Docker)
- **advertise_port** (*Optional*): If you need to specifically override the advertised UPNP port.

- **upnp_bind_multicast** (*Optional*): Whether or not to bind the UPNP (SSDP) listener to the multicast address (239.255.255.250) or instead to the (unicast) host_ip address specified above (or automatically determined). The default is true, which will work for most situations.  In special circumstances, like running in a FreeBSD or FreeNAS jail, you may need to disable this.

- **off_maps_to_on_domains** (*Optional*): The domains that maps an "off" command to an "on" command.

  For example, if `script` is included in the list, and you ask Alexa to "turn off the *water plants* script," the command will be handled as if you asked her to turn on the script.

  If not specified, this defaults to the following list:

  - `script`
  - `scene`

- **expose_by_default** (*Optional*): Whether or not entities should be exposed via the bridge by default instead of explicitly (see the 'emulated_hue' customization below). If not specified, this defaults to true. Warning: If you have a lot of devices (more than 49 total across all exposed domains), you should be careful with this option. Exposing more devices than Alexa supports can result in it not seeing any of them.  If you are having trouble getting any devices to show up, try disabling this, and explicitly exposing just a few devices at a time to see if that fixes it.

- **exposed_domains** (*Optional*): The domains that are exposed by default if `expose_by_default` is set to true. If not specified, this defaults to the following list:
  - `switch`
  - `light`
  - `group`
  - `input_boolean`
  - `media_player`
  - `fan`

A full configuration sample looks like the one below.

```yaml
# Example configuration.yaml entry
emulated_hue:
  host_ip: 192.168.1.186
  listen_port: 8300
  advertise_ip: 10.0.0.10
  advertise_port: 8080
  off_maps_to_on_domains:
    - script
    - scene
  expose_by_default: true
  exposed_domains:
    - light
```

With additional customization you will be able to specify the behavior of the existing entities.

```yaml
# Example customization
homeassistant:
  customize:
    light.bedroom_light:
      # Don't allow light.bedroom_light to be controlled by the emulated Hue bridge
      emulated_hue_hidden: false
    light.office_light:
      # Address light.office_light as "back office light"
      emulated_hue_name: "back office light"
```

The following are attributes that can be applied in the `customize` section:

- **emulated_hue_hidden** (*Optional*): Whether or not the entity should be exposed by the emulated Hue bridge. Adding `emulated_hue_hidden: false` will expose the entity to Alexa. The default value for this attribute is controlled by the `expose_by_default` option.
- **emulated_hue_name** (*Optional*): The name that the emulated Hue will use. The default for this is the entity's friendly name.

### {% linkable_title Troubleshooting %}

You can verify that the `emulated_hue` component has been loaded and is responding by pointing a local browser to the following URL:

 - `http://<HA IP Address>:8300/description.xml` - This URL should return a descriptor file in the form of an XML file.
 - `http://<HA IP Address>:8300/api/pi/lights` - This will return a list of devices, lights, scenes, groups, etc.. that `emulated_hue` is exposing to Alexa.

For Google Home, verify that the URLs above are using  port 80, rather than port 8300 (i.e. `http://<HA IP Address>:80/description.xml`).

An additional step is required to run Home Assistant as non-root user and use port 80 when using the AiO script.  Execute the following command to allow `emulated_hue` to use port 80 as non-root user.

```bash
sudo setcap 'cap_net_bind_service=+ep' /srv/homeassistant/homeassistant_venv/bin/python3
```
Please note that your path may be different depending on your installation method. For example, if you followed the [Virtualenv instructions](https://home-assistant.io/docs/installation/virtualenv/), your path will be `/srv/homeassistant/bin/python3`.

### {% linkable_title License %}

Much of this code is based on work done by Bruce Locke on his [ha-local-echo](https://github.com/blocke/ha-local-echo) project, originally released under the MIT License. The license is located [here](https://github.com/blocke/ha-local-echo/blob/b9bf5dcaae6d8e305e2283179ffba64bde9ed29e/LICENSE).
