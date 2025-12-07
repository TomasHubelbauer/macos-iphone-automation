# macOS iPhone automation

I am trying to find a way to fully control an iPhone from my Mac Mini running
macOS.

There solutions won't work for me:

- Apple's XCUITest testing framework

  Won't allow to automate the entire operating system.
  Meant to support development and testing of own apps.
  Requires interacting with XCode.

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

1. Install `libimobiledevice` with `brew install libimobiledevice`
2. Connect the iPhone with a USB C cable
3. Allow the iPhone accessory to connect via the macOS dialog that appears
4. Allow the iPhone to trust the macOS computer via the iOS trust dialog
5. Run `idevicepair pair` and ensure the success confirmation is printed

Turns out for modern iOS versions the "Developer Disk Image" thing (which is
something that runs alongside iOS on the phone as long as it is connected to the
macOS computer and provides the macOS<>iOS interaction services),
`libimobiledevice` can't be used anymore?

1. Install `pymobiledevice3`: `pip3 install pymobiledevice3 --break-system-packages`
2. Run `sudo pymobiledevice3 lockdown start-tunnel` and note the `--rsd` line
3. Confirm the pairing request on the iPhone

Next up looks like what's needed is:

1. Enable developer mode: `pymobiledevice3 amfi enable-developer-mode`

   Requires disabling the screen lock (presumably can be re-enabled after).
   This can also be done via Settings > Privacy & Security, but the option will
   only appear when connected to XCode.

2. Mount the developer disk image: `pymobiledevice3 mounter auto-mount`

   This will install the developer tools including the `screenshot` service.
   Without this, this error shows up: `ERROR Failed to start service.`

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
