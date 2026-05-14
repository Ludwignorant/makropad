A Custom ZMK Macropad Build with XIAO nRF52840
I designed a wireless macropad using the XIAO nRF52840 module by Seeed Studios. Let’s cover how to set it up with the open source keyboard firmware ZMK.

Final picture of the macropad

The design  is quite simple since our XIAO module has a lot of features out of the box. All we really need the PCB to do is to wire up the encoder and switches to GPIO pins on the module. The XIAO modules takes care of a USB-C port and battery management.

PCB schematic

PCB layout

The parts list:

Part	Quantity Used
Seeed Studio XIAO nRF52840 	1
EC11 Rotary Encoder 	1
Rotary Encoder Knob 	1
Cherry MX Switches 	3
Keycaps 	3
Self tapping screws 	4
Rubber feet 	4
I am going to name my macropad “Gigapad” (Bluetooth’s frequency is in the Gigahertz, clever right?). You will see a lot of references to Gigapad in the steps below. Know that you can safely replace any instance of Gigapad with your own name. But, be aware that many steps are strictly case sensitive, so make sure to match the case throughout.

To build a custom ZMK image, you’ll need a Github  account as the firmware compiles using Github’s build server service (it’s free). Log in to Github and create a new repository. Name it something like: zmk-config-gigapad (replace gigapad with the name of your macropad).

Keeping the webpage with your new Github repository open, open a terminal (or command prompt) on your local machine. Change directory to where you want to store your ZMK project locally. An example command might be:

cd ~/Documents/projects
Once there, execute the following command:

bash -c "$(curl -fsSL https://zmk.dev/setup.sh)"
You will be greeted with a list of pre-configured keyboard options. Since we’re creating our own we’ll be re-writing all of these files. Choose any of the options. I will choose option 1 for this example. It doesn’t matter.

Terminal output from the setup script

Next is the important part. You be prompted to pick the MCU board your keyboard uses. Make sure to select “Seeeduino XIAO BLE”.

When it prompts you to copy in the stock keymap, you can choose “Y” or yes. Again, this will be overwritten with our own configuration later.

Provide your Github username and the name of the repository you just created. For me this would be: zmk-config-gigapad.

There will be a prompt that asks for Github Repo and will display a link with your username and the name of your repository. Confirm that this matches what you see on the webpage for your repository.

Github repository URL match

If it does match, you can just press enter at this prompt to accept the default. Otherwise, paste in the correct URL.

Allow the script to continue by typing “Y” for yes and enter. It will push some files to your new repository.

If you refresh your repository’s webpage, you should now see a couple of new files. That means everything had worked.

Back in the same terminal window from before, change directory into your new project’s folder. That command might look something like this:

cd zmk-config-gigapad
Open this folder in Visual Studio Code (or any other text editor) with the following command. You can also simply navigate to this folder and open it with a file explorer.

code .
The first file we need to edit is the build.yaml file. Open this up. Replace the shield entry with the name of your macropad. This is “gigapad” for me.

This file will also have the name of the board you originally picked in the configuration script. Feel free to delete some of the comments in this file.

Update build.yaml

In the config folder, delete everything except the west.yml file. Create a new folder in config named boards. In boards create another folder named shields, and in shields, create a final folder with the name of your macropad. For me this is gigapad. Your new config folder structure should look like this:

config
|_ boards
   |_ shields
      |_ gigapad
In config/boards/shields/gigapad create a new file named Kconfig.shield and paste in the following (replacing gigapad with your board name, case sensitive!).

config SHIELD_GIGAPAD
    def_bool $(shields_list_contains,gigapad)
It is very important that there is no space in this section: shields_list_contains,gigapad

In this same directory, create another file named Kconfig.defconfig. Add the following to this file (replacing gigapad with your board name).

if SHIELD_GIGAPAD
 
config ZMK_KEYBOARD_NAME
    default "gigapad"
 
endif
Remaining in this directory, create another file named gigapad.conf (replace the name). Here, you’ll add some configuration options for your firmware. This depends on your build.

More info can be found in the ZMK documentation , but our configuration is fairly self explanatory. We’re setting the Bluetooth transmit power  and enabling the EC11 rotary encoder  driver.

CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
In the same directory create another file named gigapad.overlay and paste the following:

/ {
    chosen {
        zmk,kscan = &default_kscan;
    };
 
    default_kscan: kscan {
        compatible = "zmk,kscan-gpio-direct";
        label = "default_kscan";
 
        input-gpios =
            <&xiao_d 0 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>,
            <&xiao_d 3 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>,
            <&xiao_d 4 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>,
            <&xiao_d 5 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
    };
 
    left_encoder: encoder_left {
        compatible = "alps,ec11";
        label = "encoder";
        a-gpios = <&xiao_d 1 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
        b-gpios = <&xiao_d 2 (GPIO_ACTIVE_HIGH | GPIO_PULL_UP)>;
        steps = <80>;
        status = "okay";
    };
 
    sensors {
        compatible = "zmk,keymap-sensors";
        sensors = <&left_encoder>;
    };
};
The above code is highly dependent on your macropad’s PCB design. We’re specifying which pins we’ve wired our switches and encoder to.

You’ll notice on the gigapad’s schematic , you can see we’ve connected the XIAO’s GPIO pins D0, D3, D4, and D5 to our various switches. In our gigapad.overlay we’ve configured these as active high inputs with internal pull-up resistors enabled.

Annotated picture of gigapad schematic

The pin numbers for the XIAO can be found on Seeed’s wiki  for this device. They’re labeled D(n):

Seeed wiki pinout

A special note about the ROT_SW net. The EC11 has a switch that triggers if you push on the encoder knob. This is treated just like any other keyboard switch and wired to D0 in our design.

Next, you’ll see the encoder configuration. We configure the EC11 encoder and assign it pins D1 and D2. This can be seen in the schematic.

We give it a steps property  with a value of 80. This is standard for the EC11, but you might want to play around with this value to adjust the sensitivity.

It’s worth noting that the encoder is named “left_encoder” because in ZMK it is standard to name the first device starting from the left of the keyboard (or keyboard halves). For our macropad, this doesn’t really matter since we just have one.

Finally, we assign the sensors node which connects our left_encoder to the “zmk,keymap-sensors”  device tree definition.

At last, we configure the macros for our macropad. Create a file named gigapad.keymap in the same directory we’ve been working in.

Copy and paste the following into your keymap file:

#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/outputs.h>
 
/ {
    macros {
        finder_home: finder_home {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <40>;
            tap-ms = <40>;
            bindings =
                <&macro_press &kp LEFT_COMMAND>,
                <&macro_press &kp LEFT_SHIFT>,
                <&macro_tap &kp H>,
                <&macro_release &kp LEFT_COMMAND>,
                <&macro_release &kp LEFT_SHIFT>;
        };
        finder_desktop: finder_desktop {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <40>;
            tap-ms = <40>;
            bindings =
                <&macro_press &kp LEFT_COMMAND>,
                <&macro_press &kp LEFT_SHIFT>,
                <&macro_tap &kp D>,
                <&macro_release &kp LEFT_COMMAND>,
                <&macro_release &kp LEFT_SHIFT>;
        };
        finder_downloads: finder_downloads {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <40>;
            tap-ms = <40>;
            bindings =
                <&macro_press &kp LEFT_COMMAND>,
                <&macro_press &kp LEFT_ALT>,
                <&macro_tap &kp L>,
                <&macro_release &kp LEFT_COMMAND>,
                <&macro_release &kp LEFT_ALT>;
        };
    };
    keymap {
        compatible = "zmk,keymap";
 
        default_layer {
            bindings = <
                &bt BT_CLR  &finder_downloads  &finder_desktop  &finder_home
            >;
            sensor-bindings = <&inc_dec_kp UP DOWN>;
        };
    };
};
There’s a lot to unpack here, but look to the macros { } node to find the macros I’m creating. For this configuration I’ve built 3 macros, all for use in the MacOS file explorer.

Let’s look at how the finder_downloads macro is built. It simulates the following shortcut: CMD + SHIFT + L.

Notice the difference between macro_press, macro_tap and macro_release. This replicates holding down CMD and SHIFT, tapping L then releasing CMD and SHIFT.

There are a lot more behaviors you can use to build your macros. .

In the keymap { } node, I define a default_layer and bind my action to my keys. You can read more about layers in the documentation .

Important note: these assignments are made in the order that you defined your pins in your gigapad.overlay file.

For example, the finder_downloads macro is the second entry in the default_layer bindings entry. finder_downloads what I’ve decided to name this macro. It can safely be renamed for your use case.

Keymap screenshot showing finder_downloads

Therefore, by looking at gigapad.overlay we can see that the second entry in the input-gpios definition is D3.

Overlay screenshot showing the D3 definition

By looking at our schematic, we see that D3 is wired up to switch 2. (Counter intuitive design on my part, I should fix this)

Schematic screenshot showing SW2

A note about &bt BT_CLR. As you can see by its placement in the default_layer bindings definition it is wired to D0 which is connected to the rotary encoder’s switch.

Pressing this switch will clear the Bluetooth bonding information and disconnect your macropad from your computer. The macropad will not be able to reconnect to your computer until you “forget” the connection in your Bluetooth settings.

Forget Bluetooth connection

You can read about all of the Bluetooth actions in the ZMK documentation .

I put this in for debugging, but haven’t really found a good macro to assign to that switch so I kept it in. It’s not very useful since I plan to keep this macropad connected to just one computer at all times.

Note the sensor-bindings entry. This defines the behavior of the rotary encoder. Turning the encoder simulates an “UP ARROW” or “DOWN ARROW” keypress depending on the direction.

The same principle applies here, the first assignment “UP” corresponds to the first pin definition in your overlay file.

And that’s it! When you’re happy with your macros, go back to a terminal in your zmk-config-gigapad directory and type:

git add .
git commit -m "my new commit"
git push
Navigate back to the webpage for your Github repository and refresh. Click the “actions” tab and find your current build process. It will have the same name as the commit message you just used.

Github actions screenshot

Wait for this action to complete as shown by the indicator next to the action. It will take a bit. Mine usually take around 5 minutes to build.

Once complete, click into the recently run job and find the “artifacts” section. Here you’ll see a file named “firmware”. Click on it to download. It’s a zip folder containing a .uf2 file, this is what we’ll need to upload to our XIAO device.

Firmware download screenshot

Upload it by plugging your XIAO into your computer and double pressing the reset button on the module. You should see a removable disk show up in your file explorer. Drag the .uf2 file into the removable disk. Give it a minute to transfer and reset the device.

Pinout diagram showing resetFile upload screenshot

Be aware that you might get some error indicating that the disk was not properly ejected. These errors can mostly be ignored.

File upload error

That’s it! Now in your Bluetooth settings, you should see a new device available with the name of your macropad

Bluetooth discovery screenshot

Sometimes you will need to choose “forget this device” on your computer after re-uploading new firmware. Try this if you can’t connect for any reason.

Forget this device
