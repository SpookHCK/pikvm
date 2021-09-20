What works:

GUI integrated channel control
MSD - USB CD pass through etc... 

Parts list:

PiKVM - assuming you have it working... I have one with a HDMI Loop device, because I have devices with strange BIOS' that I want to control

PWAY 16-Port KVM Includes 16xHDMI and 16xUSB () cables and the little green RS232 block needed to wire everything up
https://www.alibaba.com/product-detail/PWAY-SH1601A-16-Port-Support-4Kx2K_62474691127.html
It took 3 days for shipping to the US after about 10 days waiting for dispatch.
PWAY support seems to try and be helpful, so extra points for that, but I got it working before they answered.

USB RS232 adapter - any RS232 that presents as a ttyUSB device in linux should work...
https://www.amazon.com/gp/product/B00ZHP2NN0/

RS232 DB9 breakout - I'm feeling lazy and didn't want to solder a DB9 Port one just for this...
https://www.amazon.com/gp/product/B073RGRQ43/



Wiring the RS232

so on the KVM the port is labeled TGR (no not for Tiger) Transmit, Ground, Receive. These are all RS232 voltages so don't think TTL and fry your toys...

Serial.jpg

so wire T to Pin 2 on the DB9
        R to Pin 3 on the DB9
        G to Pin 5 on the DB9
        

Now plug the USB to RS232 adapter into your PiKVM and connect the male USB RS232 adapter to the female TGR DB9 port.

This concludes the hardware build.


Software:

Ssh into your PiKVM.
Enable read-write mode on the sd card via `rw`
1st lets test everything.
Install the screen tool `pacman -S screen` (needs root and internet access)
Now run `sudo dmesg | grep ttyUSB`
You should see something with "converter now attached to ttyUSB0" This means that your serial port is ttyUSB0

Lets test the KVM; `screen /dev/ttyUSB0 19200` The console should go blank
Now power cycle the KVM.
You should see:

AX6800x bootloader[0] v1.0.3

Runtime checksum check.


AX68002 KVM S7F01H_9187, SDK:2.0.6.2 - Version:1.0.2,Date:2021-06-28


[Virtual HUB] VID = 0b95, PID = 6802

> DC_Port:0, VBUS Attached

> DC_Port:0, Suspend

VGA_SWITCH_CONTROL=0

> DC_Port:0, Reset

> DC_Port:0, Suspend

ch452 version: 0x20.
->state:1, set led0  0x8. 

If you see this then you have wired up the receive side of the RS232 port correctly
Now type in `PS 2` and press enter. You should see the characters as you type; then the KVM should beep and you have switched to port 2
exit the screen command with [control]+[a] then press [\] then confirm with [y]


If that does not work test the serial port by disconnecting the TGR port from the KVM and short out pin T with R (use a metal paperclip or some wire)
Then use the screen session and type you should see your letter print on the screen. if that doesn't work either the RS232 port is bad or the cable is bad...

Assuming success...

Now add the `pway.py` file to `/usr/lib/python3.9/site-packages/kvmd/plugins/ugpio/` This is the driver file that has been modified to make the 16-port KVM work; Thanks Max; you made this super easy!
I found SCP worked well, but there are lots of options.

Next modify the `/etc/kvmd/override.yaml` file and include the following. Note the assumption that the RS232 adapter is present on /dev/ttyUSB0:

kvmd:
    gpio:
        drivers:
            pw:
                type: pway
                device: /dev/ttyUSB0
        scheme:
            ch1_led:
                driver: pw
                pin: 1
                mode: input
            ch2_led:
                driver: pw
                pin: 2
                mode: input
            ch3_led:
                driver: pw
                pin: 3
                mode: input
            ch4_led:
                driver: pw
                pin: 4
                mode: input
            ch5_led:
                driver: pw
                pin: 5
                mode: input                                
            ch6_led:
                driver: pw
                pin: 6
                mode: input
            ch7_led:
                driver: pw
                pin: 7
                mode: input
            ch8_led:
                driver: pw
                pin: 8
                mode: input
            ch9_led:
                driver: pw
                pin: 9
                mode: input
            ch10_led:
                driver: pw
                pin: 10
                mode: input
            ch11_led:
                driver: pw
                pin: 11
                mode: input
            ch12_led:
                driver: pw
                pin: 12
                mode: input
            ch13_led:
                driver: pw
                pin: 13
                mode: input
            ch14_led:
                driver: pw
                pin: 14
                mode: input
            ch15_led:
                driver: pw
                pin: 15
                mode: input
            ch16_led:
                driver: pw
                pin: 16
                mode: input                                
            ch1_button:
                driver: pw
                pin: 1
                mode: output
                switch: false
            ch2_button:
                driver: pw
                pin: 2
                mode: output
                switch: false
            ch3_button:
                driver: pw
                pin: 3
                mode: output
                switch: false
            ch4_button:
                driver: pw
                pin: 4
                mode: output
                switch: false
            ch5_button:
                driver: pw
                pin: 5
                mode: output
                switch: false
            ch6_button:
                driver: pw
                pin: 6
                mode: output
                switch: false
            ch7_button:
                driver: pw
                pin: 7
                mode: output
                switch: false
            ch8_button:
                driver: pw
                pin: 8
                mode: output
                switch: false
            ch9_button:
                driver: pw
                pin: 9
                mode: output
                switch: false
            ch10_button:
                driver: pw
                pin: 10
                mode: output
                switch: false
            ch11_button:
                driver: pw
                pin: 11
                mode: output
                switch: false
            ch12_button:
                driver: pw
                pin: 12
                mode: output
                switch: false
            ch13_button:
                driver: pw
                pin: 13
                mode: output
                switch: false
            ch14_button:
                driver: pw
                pin: 14
                mode: output
                switch: false
            ch15_button:
                driver: pw
                pin: 15
                mode: output
                switch: false                
            ch16_button:
                driver: pw
                pin: 16
                mode: output
                switch: false
        view:
            table:
                - ["#Input 1", ch1_led, ch1_button]
                - ["#Input 2", ch2_led, ch2_button]
                - ["#Input 3", ch3_led, ch3_button]
                - ["#Input 4", ch4_led, ch4_button]
                - ["#Input 5", ch5_led, ch5_button]
                - ["#Input 6", ch6_led, ch6_button]
                - ["#Input 7", ch7_led, ch7_button]
                - ["#Input 8", ch8_led, ch8_button]
                - ["#Input 9", ch9_led, ch9_button]
                - ["#Input 10", ch10_led, ch10_button]
                - ["#Input 11", ch11_led, ch11_button]
                - ["#Input 12", ch12_led, ch12_button]
                - ["#Input 13", ch13_led, ch13_button]
                - ["#Input 14", ch14_led, ch14_button]
                - ["#Input 15", ch15_led, ch15_button]
                - ["#Input 16", ch16_led, ch16_button]
                
Return to read-only mode for the sd card via `ro`

Switching between hosts in the UI

To switch between hosts, enter the KVM UI and click the "GPIO" menu. You should see 16 inputs, one of which will have a green circle indicating it is currently selected. Click the other inputs to change the selected host.
