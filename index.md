# Operator Control of Rubin Observatory NVR Camera Infrared Illuminators

```{abstract}
Proposed solution to provide web dashboard control of the infrared illuminators on the UniFi web cameras
deployed throughout the observatory, without needing to provide and admin-level UniFi account for each
operator.
```

## Introduction

Operators at the observatory occasionally need to control the illuminators on the UniFi web cameras deployed
throughout the observatory.  This has been problematic, since such control via the stock UniFi web interface
requires a UniFi account with administrator-level privileges and the system supports only a limited number of
such accounts. Broad distribution of a shared admin-level account with full control over the UniFi system to
the entire operator staff has been viewed as a non-starter from a security perspective.

A survey of available Python libraries which might be used to control the illuminators via UniFi-provided web
APIs was conducted.  Unfortunately, a compelling option was not immediately apparent; the UniFi APIs
themselves are rather poorly documented, and the off-the-shelf client libraries currently available are the
usual assortment of abandonware, partially implemented, or outdated solutions.

In the course of the survey, however, it was discovered that there is a fairly complete and up-to-date support
for UniFi NVR camera systems within the [Home Assistant](https://home-assistant.io) home automation platform.
Home Assistant is a popular and vigorous open-source project with hundreds-of-thousands of installations
world-wide.  In addition to providing an up to date, maintained, and complete interface to the UniFi camera
systems, the Home Assistant platform itself provides a tailorable web control UI, configuartion support,
authentication plugins, etc. all directly out of the box.  Home Assistant is also distributed as a pre-built
container that can be directly utilized on the observator Kubernetes clusters. It was decided to evaluate the
use of Home Asisstant and its [in-built UniFi
support](https://www.home-assistant.io/integrations/unifiprotect/) as a potential low-effort near-term
solution.

## Prototype

IT provided the following for the purposes of prototyping:

* Access to the kueyen dev cluster
* A DNS CNAME (`test-nvr.dev.lsst.org`) for the ingres controller
* A dedicated, local service account on the NVR appliance

A simple Kubernetes deployment using the latest official Home Assistant release container was hand-coded and
manually deployed with `kubectl`:

```{figure} kubernetes.svg
Prototype Kubernetes Deployment
```

Home Assistant was then configured via its built-in UI.  Once pointed at the NVR appliance and auth'd, all
on-site UniFi cameras were automatically introspected and made available in the interface.

A dedicated "kiosk mode" page which exposes illuminator controls only was built, using a kiosk plug-in
available in the Home Assistant community store.  This was set as the default view for a shared "operators"
local Home Assistant account, and we had a fully operable system up and running:

```{figure} interface.png
Prototype illuminator control page
```

## To Do

Use of Home Assistant for this purpose in the near term seems completely feasible from a technical
perspective.  Should we wish to do this, the following would still need to be done to close the gap between a
prototype and an operable deployment:

* If desired, integrate with Keycloak authentication for finer-grained access control.  This would obviate
  need for a shared Home Assistant "operator" account and password.  This is feasible, but perhaps not an
  issue since the dashboard is limited to illuminators only? <br><br>

* Decide which organization within the observatory would adopt and maintain the deployment.  This would
  seemingly be either Telescope and Site, or IT.  The deployment described here has no direct integration with
  the observatory control systems, and interacts only with IT owned/administrated systems. <br><br>


* "Devops-ify" the deployment.  This would mean wrapping up the existing prototype deployment
   yamls in either Rancher Fleet (if adopted by IT) or Phalanx (if adopted by Telescope and Site). <br><br>

* Arrange for backup of the persistent volume on which the Home Assistant configuration is stored. <br><br>