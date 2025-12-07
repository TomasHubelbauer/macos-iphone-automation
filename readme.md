# macOS iPhone automation

I am trying to find a way to fully control an iPhone from my Mac Mini running
macOS.

There solutions won't work for me:

- Apple's XCUITest testing framework

  Won't allow to automate the entire operating system.
  Meant to support development and testing of own apps.
  Requires interacting with X Code.

- 3rd party testing tools

  Require a supporting application installation on the target device, a
  developer certificate and/or a provisioning profile.
  Not meant for general automation.

- iPhone Mirroring feature on macOS

  Doesn't provide programmatic control.
  Also, not available in Europe.

- Switch Control feature on iOS

  Doesn't provide programmatic control.
  Could be used in conjunction with programmatic HID.

- Jail-breaking the device

  It is a PITA and I don't want to do it.
  Also not immediately available following new iOS version releases.

This is the solution I am working towards building at the moment:

## The "read" side

Use `libimobiledevice` and its `idevicescreenshot` utility to monitor the iPhone
screen.
Use an LLM or SAM to recognize UI landmarks and elements for coordinates.

The landmark and element recognition could be improved with the use of Switch
Control potentially because AFAIK it renders an outline over the active element.

The virtual pointer displayed when a mouse is connected could also help anchor
the UI element recognition logic.

## The "write" side

The only feasible solution seems to be to part a HID to the iPhone and have it
emit keyboard and mouse events as appropriate.

I was considering Handheld Scientific BT-600, but it doesn't seem to have a
programmatic access option, the serial interface is only for configuration.

Now I'm thinking of using a Raspberry Pi Zero W and have it be powered off the
controlling Mac Mini with a firmware that uses wi-fi to expose a programmatic
interface and accept commands to translate to HID events for keyboard and mouse
movement to send to the iPhone paired over BLE.

Switch Control could play some sort of a role here to make the navigation work
more reliably by automatically switching among landmarks etc.
