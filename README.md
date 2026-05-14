Keymap settings:
Folder: C:\Hobby\Projekte\macropad\makropad\config\boards\shields\macropad

//includes
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>
#include <dt-bindings/zmk/outputs.h>

//Macros can be defines as is or can be deleted 

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
	 
//Keymap is setting the key in order undr bindings. The order is the same as in macropad.overlay file

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
