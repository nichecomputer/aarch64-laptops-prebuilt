## AArch64 Laptop Prebuilt Images

### Contributing

The current outstanding [Jobs List](#Jobs-list) can be seen below.

### Images

This repos offers the latest prebuilt images for your consumption.  There are currently 2 images available:

 * Complete prebuilt (configured) image
   * This image is **almost** (see below) ready for booting

 * Clean (not configured) Ubuntu Bionic image
   * This image is designed for the [Image Builder](https://github.com/aarch64-laptops/build) to consume

#### Complete prebuilt (configured) image

This image is ready to be flashed onto an SD card to boot any one of the 3 devices currently supported by this project:

 * Asus NovaGo TP370QL
 * HP Envy x2
 * Lenovo Miix 630

**IMPORTANT:** This image must be flashed using the [Flash Tool](https://github.com/aarch64-laptops/prebuilt/blob/master/flash-prebuilt.sh) provided by this repo.

If the [Flash Tool](https://github.com/aarch64-laptops/prebuilt/blob/master/flash-prebuilt.sh) is not used, the image will not boot.  If you find yourself in this situation you can do the following:

```
$ sudo mount /dev/<SD_CARD>2 <SDCARD_MOUNT_POINT>
$ ln -s /boot/<LAPTOP_DTB> <SDCARD_MOUNT_POINT>/boot/laptop.dtb
```

Where <LAPTOP_DTB> is either `laptop-asus-tp370ql.dtb` for the Asus NovaGo TP370QL or `laptop-hp-envy-x2.dtb` for HP Envy x2 and Lenovo Miix 630

**Note:** Currently there are no known differences between the later 2 devices, so they use the same DTB.

Example:

```
$ sudo mount /dev/sda2 /mnt
$ ln -s /boot/laptop-asus-tp370ql.dtb /mnt/boot/laptop.dtb
```

**Note:** Obviously this will only work if your machine enumerated the SD card at `/dev/sda` and you wish to boot the Asus NovaGO TP370QL.

#### Flash Tool

To use the Flash Tool, do the following:

 1. Download the latest [Prebuilt Image](https://github.com/aarch64-laptops/prebuilt/raw/master/aarch64-laptops-bionic-prebuilt.img.xz) and make it executable
 2. Download the [Flash Tool](https://github.com/aarch64-laptops/prebuilt/blob/master/flash-prebuilt.sh)
 3. Use the [Flash Tool](https://github.com/aarch64-laptops/prebuilt/blob/master/flash-prebuilt.sh) to flash the latest [Prebuilt Image](https://github.com/aarch64-laptops/prebuilt/raw/master/aarch64-laptops-bionic-prebuilt.img.xz) onto the SD card

Example:

```
$ wget https://github.com/aarch64-laptops/prebuilt/raw/master/aarch64-laptops-bionic-prebuilt.img.xz
$ wget https://raw.githubusercontent.com/aarch64-laptops/prebuilt/master/flash-prebuilt.sh
$ chmod +x flash-prebuilt.sh
$ ./flash-prebuilt.sh asus-tp370ql /dev/sda
```

Once complete, place the SD card into your machine and boot.

If you need help with that see the [Booting into Ubuntu](https://github.com/aarch64-laptops/build#booting-into-ubuntu) section of the [Image Builder](https://github.com/aarch64-laptops/build) README.
 
### Clean (not configured) Ubuntu Bionic image

This image is consumed by the CI loop and by the [Image Builder](https://github.com/aarch64-laptops/build) when the [Quick Start Script](https://github.com/aarch64-laptops/build/blob/master/scripts/quick-start.sh) is used.

When used, it provides the following benefits:

1. The build can be completely unattended
2. Saves a great deal of time (2 hours) in the Ubuntu installer

**Note:** This image is not bootable on its own.

## Jobs List

The code for each of these can be found in the [downstream tree](https://source.codeaurora.org/quic/la/kernel/msm-4.4/tree/?h=msm-4.4).

### MultiMedia Clock Controller (MMCC)

Status: Patches on [the list](https://patchwork.kernel.org/cover/10788967/) [Jeff Hugo]

### MultiMedia SubSystem (MMSS)

Status: Depends on [MultiMedia Clock Controller (MMCC)](MultiMedia-Clock-Controller-(MMCC)) [Jeff Hugo]

### Display

Status: Depends on [MultiMedia SubSystem (MMSS)](MultiMedia-SubSystem-(MMSS)) [Jeff Hugo]

Comments:

 * Once the [MultiMedia Clock Controller (MMCC)](MultiMedia-Clock-Controller-(MMCC)) is ready, work needs to be undertaken on the [Mobile Display SubSystem (MDSS)](https://www.kernel.org/doc/Documentation/devicetree/bindings/display/msm/mdp5.txt).
 * The output from the MDSS is fed into the DSI/eDP bridge, which already has full support upstream.
 * eDP is the display panel on the laptops.
 * Specific panels can then be described (e.g. Samsung 123).
 * If devices support more than one panel it is the F/W's responsibility to edit the DT and advertise the default at runtime.
 * Generic eDP bindings might be an option, the [upstream discussion](https://www.spinics.net/lists/devicetree/msg272412.html) continues.
 * It is thought that the 3 laptops have 3 different types of panel which all need to be supported.
 * Once complete, the laptops will still be software rendered, but we're bypassing UEFI and we're en route to being accelerated.

### Graphics Processing Unit (GPU)

Status: Depends on [Display](Display) [Jeff Hugo]

Comments:

 * Once Display is ready, the GPU should, in theory at least, be plug and play.
 * This will provide us with a a fully hardware accelerated system.
 * Once hardware rendering is enabled the application CPU will be relinquished of its CPU intensive software rendering duties ensuing a faster/smoother user experience.
 * There is a small possibility that a few GPU driver tweaks may be required - but we'll cross that bridge when we come to it.

### Camera

Status: Depends on [MultiMedia SubSystem (MMSS)](MultiMedia-SubSystem-(MMSS)) [Not yet allocated]

### USB

Status: USB is working on all of the platforms, but needs work [Not yet allocated]

Comments:

 * The USB controller currently hard coded to Host Mode.
   * Peripheral (device) mode can be supported too.
   * Device type should be automatically detected by DWC3-QCom using an Extcon.
   * VBus, PD and all the other pieces are managed by the EC (USCI compliant).
     * UCSI code available in this [kernel directory](https://elixir.bootlin.com/linux/v5.0-rc6/source/drivers/usb/typec/ucsi).
       * Controlled via userspace.
   * Once fully enabled we can remove `dr_mode=host` from Device Tree.
     * Might be needed for for Displayport (HDMI) over USB, etc.

### PCIe

Status: In active development [Marc Gonzalez]

Comments:

 * PCIe is used on the Asus NovaGO TP370QL for an additional USB hub
 * Involves porting QMP Phy support to bring up and use PCIe
 * This might have a nice side-effect of making the USB ports faster
   * Linux would have to be able to discover and fully control the device/PCIe bus

### Capacitive Touchpanel (+ stylus)

Status: Not started [Not yet allocated]

Comments:

 * Possibly limited to the HP Envy x2 and Lenovo Mixx 630
 * The touchpanel and stylus appear to be attached to a different I2C bus as an ELAN device
 * Would need to check the ACPI tables for more details

### Other interesting topics

 * ACPI enablement      [Not yet allocated]
 * CPU Freq             [Not yet allocated]
 * CPU Scaling          [Not yet allocated]
 * Thermal Limiting     [Not yet allocated]
 * Keyboard Hotplug     [Not yet allocated]
   * EC on I2C7
