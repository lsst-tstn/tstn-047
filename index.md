# Operator Control of Rubin Observatory NVR Camera Infrared Illuminators

```{abstract}
Interim solution to provide web dashboard control of the infrared illuminators on the UniFi web cameras
deployed throughout the observatory, without needing to provide and admin-level UniFi account for each
operator.
```

## Introduction

Operators at the observatory occasionally need to control the illuminators on the UniFi web cameras deployed
throughout the observatory.  This has been problematic, since such control via the stock UniFi web interface
requires a UniFi account with administrator-level privileges and the system supports only a limited number of
such accounts.

There is, however, fairly complete and up-to-date support for UniFi NVR camera systems within the [Home
Assistant](https://home-assistant.io) home automation platform, which also provides a tailorable web UI and
authentication plugins.

A Phalanx chart wrapping the official Home Assistant release container has been developed and deployed at the
base test stand and summit, which provides simplified control of the camera illuminators for anyone on the
summit network with valid KeyCloak credentials.  All accesses through the platform use a single backend UniFi
service account, the credentials for which need not be shared.


## Use and Operation

The NVR application can be reached on the summit network or summit VPN at <https://nvr.summit-lsp.lsst.codes>.
After authenticating with summit KeyCloak, users are redirected to a single page interface with a pull-down
menu for each known UniFi camera:

```{figure} interface.png
Illuminator control page
```

Supported illuminator modes for each camera are available on the individual menus.  Menu selections take
effect immediately.  Cameras which are currently completely offline are indicated as greyed out.


## Technical Details

The relevant phalanx chart is maintained at
<https://github.com/lsst-sqre/phalanx/tree/main/applications/nvr-control>.

After a first-time deployment, a Kubernetes port-forward is used to access Home Assistant on its default port
in order to perform one-time setup. The resulting configuration is stored automatically on an attached
persistent volume and used from that point forward.

Initial setup is as follows:

* Use the on-boarding wizard to create an ``admin`` account, using the password escrowed in the LSST IT
  [1password](https://lsstit.1password.com) "NVR Camera App" vault.

* Add the Unifi Protect integration, and configure it with the shared service account credentials also in
  the 1password vault.  All attached cameras are then automatically detected.

* Edit the default "Overview" dashboard to be a single page displaying NVR illuminator controls.  This
  can be done incrementally in the dashboard editor, or pick "Raw configuration editor" from the dots menu
  at the upper right in the editor and paste in the following yaml for example to do it all in one go:

  ```yaml
  views:
    - path: default_view
      title: NVR Camera IR Modes
      cards:
        - type: entities
          entities:
            - entity: select.aux_cam01_infrared_mode
            - entity: select.aux_cam02_infrared_mode
            - entity: select.aux_cam03_infrared_mode
            - entity: select.aux_cam05_infrared_mode
            - entity: select.besalco_cam01_infrared_mode
            - entity: select.comcam_cam01_infrared_mode
            - entity: select.dimm_cam01_infrared_mode
            - entity: select.dimm_cam02_infrared_mode
            - entity: select.dome_cam01_infrared_mode
            - entity: select.dome01_ptz_infrared_mode
            - entity: select.eie_cam01_infrared_mode
            - entity: select.gen_cam00_infrared_mode
            - entity: select.gen_cam01_infrared_mode
        - type: entities
          entities:
            - entity: select.highbay_cam01_infrared_mode
            - entity: select.highbay_cam02_infrared_mode
            - entity: select.highbay_cam03_infrared_mode
            - entity: select.highbay_cam04_infrared_mode
            - entity: select.highbay_cam06_infrared_mode
            - entity: select.m1m3_cam01_infrared_mode
            - entity: select.main7_cam01_infrared_mode
            - entity: select.main7_cam02_infrared_mode
            - entity: select.main7_cam03_infrared_mode
            - entity: select.main7_cam04_infrared_mode
            - entity: select.penon_cam01_infrared_mode
            - entity: select.penon01_ptz_infrared_mode
            - entity: select.pflow_cam01_infrared_mode
        - type: entities
          entities:
            - entity: select.pflow_cam02_infrared_mode
            - entity: select.pier5_cam01_infrared_mode
            - entity: select.pier5_dynalene_cam01_infrared_mode
            - entity: select.pier5_dynalene_cam02_infrared_mode
            - entity: select.pier6_cam01_infrared_mode
            - entity: select.pier7_cam01_infrared_mode
            - entity: select.pier7_cam02_infrared_mode
            - entity: select.pier7_cam03_infrared_mode
            - entity: select.pier7_refrig_cam01_infrared_mode
            - entity: select.sdc_cam01_infrared_mode
            - entity: select.sdc_cam02_infrared_mode
            - entity: select.sdc_cam03_infrared_mode
            - entity: select.shutter_motor01_infrared_mode
      type: masonry
      icon: mdi:webcam
  ```

* Download and install the Home Assistant Community Store integration ("HACS"), per directions at
  <https://www.hacs.xyz/docs/use/>.

* Install the "Kiosk Mode" extension in HACS. This allows hiding extraneous/distracting menus and UI elements.
  Configure kiosk mode for the "Overview" dashboard by returning to the "Raw configuration editor" and pasting
  in the following yaml at the top above the ``views:`` section:

  ```yaml
  kiosk_mode:
    hide_sidebar: true
    hide_menubutton: true
    hide_search: true
    hide_assistant: true
  ```

  ```{note}
  To temporarily restore the sidebar and menus at any point after activating kiosk mode, append
  ``?disable_km`` in the browser URL bar.
  ```

* Create a new non-administrator ``operator`` user in the "People" configuration section, using the password
  escrowed in 1password. This will be the single shared Home Assistant user used downstream of KeyCloak
  authentication.

* Install the "Header Authentication" extension in HACS.  This will enable Home Assistant to auth as
  ``operator`` automatically via a header passed from upstream after KeyCloak authentication.

* Get a shell in the Home Assistant pod via Kubernetes, and enable authenticated external access by adding the
  following sections to ``/config/configuration.yaml``, immediately before the ``!include`` sections at the
  end:

  ```yaml
  http:
    use_x_forwarded_for: true
    trusted_proxies:
      - 10.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16

  auth_header:
    username_header: X-Rubin-NVR-Proxied-User
  ```

* Return to the Home Assistant web UI and reload the edited configuration via the "Developer tools" view.  The
  application should now be accessible without port forward via its auth'd "front door" at
  <https://nvr.summit-lsp.lsst.codes>.


## To Do

* Consider obtaining a surplus UniFi NVR for integration into the BTS.  Right now, the BTS deployment accesses
  the NVR appliance at the summit.

* Integrate PDU control for full power-cuts to a subset of cameras in the dome once the (currently misplaced)
  PDU hardware is located.
