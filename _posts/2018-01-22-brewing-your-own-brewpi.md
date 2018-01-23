---
layout: post
section-type: post
title: "Brewing your own BrewPi"
category: homebrewing
---

Like most people, I found out how important controlling fermentation temperature is only after making a huge mistake and letting a batch of beer ferment about 10 degrees Fahrenheit over the yeast strains max temperature.  And it was so horrible I had to dump it all, it wasn't even close to drinkable.  And I think my next actions where the same as most others as well, got my hands on an old cheap fridge (stand up freezer in my case), purchased an [STC-1000](https://www.amazon.com/Inkbird-All-Purpose-Temperature-Controller-Fahrenheit/dp/B00OXPE8U6/ref=sr_1_3?ie=UTF8&qid=1516664787&sr=8-3&keywords=stc-1000) relay with temp probe, and built something I could control fermentation temperature with.  ![STC-1000]({{ "/img/2018-01-22-brewing-your-own-brewpi/stc-1000.jpg" | absolute_url }})This is a quick, and cheap solution, and it does work, but the biggest issue I was running into, was temperature swing.  I would set my target temperature, and when it would cool, it would sometimes go 2-3 degrees Celsius (yes, I bought a Centigrade controller by accident, and they are one or the other) past the target.  Then the heater would turn on and go 1-3 degrees over.  And, especially as fermentation wound down, and the beer was creating less of its own heat, this would get even worse.  And I'm sure this doesn't affect the beer tremendously, but I still didn't like it.  Also it seemed like it was making my freezer and heater work much more then necessary as they where fighting each other.  There has to be a better way!

I started my research around the interwebs looking for other ways to build (or buy, but we all know building is more fun) a temperature controller, seems most everyone does it with the STC-1000 or some variation of it, but I kept seeing talk about building a [BrewPi](https://www.brewpi.com/).  And the more I looked into it, the more interested I became.  And at the time, I did have an extra [Raspberry Pi](https://www.raspberrypi.org/) laying around, so that made it even more appealing being that is one of the more expensive pieces of the project.  The concept is pretty close to the same as the STC-1000, the only real difference is you adding a second temperature probe (or more), and microcontroller (an Arduino Uno in this case) that runs a PID.  Basically, the system will _learn_ how your fermentation chamber heats and cools, and it can compensate for the temperature swings.  I know what your thinking, how much better can it be then the STC-1000.  Well, my first batch I fermented with this held the center of the beer to .01 degree Fahrenheit for the entire primary fermentation process.  Now, even if a few degrees doesn't affect it that much, I'm sure that holding the temperature that steady is going to produce a better final product.  And, it gets even better, after learning how the profiles work with a BrewPi, you can set it up do all sorts of different things.  For example, with my last beer, when I dry hopped it, I set it to slowly bring up the temp to max in the yeast range over a 24 hour period, then hold that for 24 hours for a diacetyl rest, then as quickly as possible, bring it down to 34f for a cold crash, and hold that for 48 hours.  And I never have to touch anything for that.

If you've made it this far, im guessing that means your still interested in building one of these bad boys.  And that is one of the main reasons of this writing, when I built mine, the info to get it working was spread all over the web, I had a bunch of dependency issues, and one obvious thing, was a case to hold it all, none of the articles or how to's, or blogs, did any one really ever tell how they have this thing in production.  And there is more then a few parts, and well, 110v running in it, so I thought that was a pretty important part, so I'll show what I used for a case to house this controller.  

Also, I want to make this clear, the developer of the BrewPi software has moved on and made his own proprietary controller, that is called the BrewPi Spark.  Its basically what we are building here, however, we will use a legacy version of the BrewPi software, because the BrewPi Spark does not use an Ardunio microcontroller, and Ardunio is not supported with the latest version.  So you will not get any new features or updates for this.  Which could be considered good or bad, it is stable, but it is what it is.  Anyway this is my journey and hopefully it will answer all your questions, and help you build a bad ass fermentation temperature controller!

---

### You will be working with electricity while building this project, if your not comfortable with that or don't know what you're doing, do not attempt this build.  Anytime you need to work on the hardware of this system always, always, always unplug it completely.  There will be 110v running inside this circuit.  PROCEED AT YOUR OWN RISK!

---

### Shopping list

Lets start with a shopping list shall we.  Now, you don't need to buy these via the links I am providing, they are merely for context.  Also, look in your junk drawer, things like the RaspberryPi and Arduino power supply, I found around the house, one was an old phone charger, and the other was to and old light that I hadn't seen in years.  The power cable is from an old computer.  I used some Romex wire that I already had as well.

* [RaspberryPi 3](https://www.amazon.com/Raspberry-Pi-RASPBERRYPI3-MODB-1GB-Model-Motherboard/dp/B01CD5VC92/ref=sr_1_3?s=pc&ie=UTF8&qid=1516666046&sr=1-3&keywords=raspberry+pi+3)
* [Arduino Uno](https://www.amazon.com/Compatible-Electronic-ATmega328P-Microcontroller-RoboGets/dp/B01N4LP86I/ref=sr_1_8?ie=UTF8&qid=1516666086&sr=8-8&keywords=arduino+uno)
* [Two Channel Relay Module](https://www.amazon.com/gp/product/B01MUATVXX/ref=oh_aui_detailpage_o02_s01?ie=UTF8&psc=1)
* [8G Micro SD Card](https://www.amazon.com/gp/product/B00M55C0VU/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
* [Three Position Screw Terminal](https://www.amazon.com/gp/product/B01M0W1X85/ref=oh_aui_detailpage_o02_s01?ie=UTF8&psc=1)
* [Jumper Wires](https://www.amazon.com/gp/product/B072L1XMJR/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1)
* [Outdoor Sprinkler System Case](https://www.amazon.com/gp/product/B000VYGMF2/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)
* [Raspberry Pi 3 Power Supply 5V 2a minimum](https://www.amazon.com/CanaKit-Raspberry-Supply-Adapter-Charger/dp/B00MARDJZ4/ref=sr_1_2?ie=UTF8&qid=1516671431&sr=8-2&keywords=rpi+power+supply)
* [Arduino Uno Power Supply 9v 1a minimum](https://www.amazon.com/Adapter-Arduino-Tbuymax-Listed-Positive/dp/B06Y1LF8T5/ref=sr_1_1_sspa?s=electronics&ie=UTF8&qid=1516671512&sr=1-1-spons&keywords=arduino+power+supply&psc=1)
* [Temperature Probes](https://www.amazon.com/gp/product/B00EU70ZL8/ref=oh_aui_detailpage_o00_s01?ie=UTF8&psc=1)
* [4.7k Ohm Resistor](https://www.amazon.com/Projects-10EP5124K70-4-7k-Resistors-Pack/dp/B0185FKBG4/ref=sr_1_6?ie=UTF8&qid=1516671702&sr=8-6&keywords=4.7k+resistor)
* [Power Cable](https://www.amazon.com/Tripp-Lite-Computer-IEC-320-C13-P007-003/dp/B00JT0DG94/ref=sr_1_3?s=industrial&ie=UTF8&qid=1516671775&sr=1-3&keywords=power+cable)
* [Outlets for the Heat/Cool](https://www.amazon.com/gp/product/B01M3URWIT/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1)
* [16 AWG Wire](https://www.amazon.com/Ancor-Marine-Grade-Primary-Battery/dp/B000NUYGDO/ref=sr_1_2?ie=UTF8&qid=1516713652&sr=8-2&keywords=16g%2Bwire&th=1&psc=1)

---

### Configuring the Raspberry Pi and Arduino Uno

I think the best place to start with this project is setting up the Raspberry Pi and the BrewPi software.  The easiest way to do this is to use the NOOBS installation of Raspbian from [raspberrypi.org](https://raspberrypi.org) (you can install any flavor of Linux you like, but Raspbian is what I have built my BrewPi on).  Here is what you will need to configure the SD card using Windows, sorry if your using Linux to copy the image to the SD card, you'll have to figure out those steps yourself (Same goes for flashing the hex file to the Arduino Uno):

* [NOOBS](https://downloads.raspberrypi.org/NOOBS_latest)
* [SD Formatter](https://www.sdcard.org/downloads/formatter_4/index.html)

Installing Raspbian on your Raspberry Pi is pretty simple:

1. Plug the SD card in to your machine.
2. Run the SD Formatter program and format the SD card.
3. Extract the NOOBS zip folder.
4. Copy all the contents from the extracted folder to the formated SD card.
5. Insert the SD card into the Raspberry Pi after the file copy completes.
6. Power up the Raspberry Pi
7. Install the Raspbian Linux Distribution.

Now, the install will take about 10 minutes, while it is going, go ahead and plug the Arduino Uno into your computer via a USB cable so you can flash the BrewPi firmware onto it.  You will need to download the following items for this:

* [BrewPi 2.10 Arduino Uno Rev C Firmware](https://github.com/BrewPi/firmware/releases/download/0.2.10/brewpi-arduino-uno-revC-0_2_10.hex)
* [Arduino Sketch Uploader 3.1.0 (latest version as of this writing)](https://github.com/christophediericx/ArduinoSketchUploader/releases/download/v3.1.0/ArduinoSketchUploader-3.1.0.zip)

After extracting the Arduino Sketch Uplaoder, flash the hex file to the Arduino Uno via the Windows command line replacing <DownloadPath> with the file path to the location of the hex file:
* `ArduinoSketchUploader.exe --file=<DownloadPath>brewpi-arduino-uno-revC-0_2_10.hex --model=Uno`

After the completion dialog box for the Raspbian install on the Raspberry Pi comes up, go ahead and reboot it.  After the Raspberry Pi comes back up, it will auto login to your new install of Raspbian.  Now there are a few basic configuration you are going to want to change to the system before we get started:

* Open the Raspberry Pi Configuration Menu:

![Raspberry Pi Configuration System Tab]({{ "img/2018-01-22-brewing-your-own-brewpi/rpi-configuration.jpg" | absolute_url }})
* On the _System_ tab:
1. CREATE A PASSWORD, by default there is no password, do not leave it this way, there is no reason to.
2. Give it a hostname, I called mine brewpi (no surprise there).

![Raspberry Pi Configuration Interfaces Tab]({{ "img/2018-01-22-brewing-your-own-brewpi/rpi-configuration-system.jpg" | absolute_url }})
* On the _Interfaces_ tab:
1. I enable ssh here, this will allow you to interface with the terminal remotely, I don't keep a keyboard and mouse hooked to this after the initial configuration, so I would suggest enabling this, and this is also why a password is good, so not just anybody can ssh to your BrewPi (you may see many more interfaces on you system, this is because I'm emulating this on a virtual machine so I can take screen shots).

![Raspberry Pi Configuration Locale Tab]({{ "img/2018-01-22-brewing-your-own-brewpi/rpi-configuration-interfaces.jpg" | absolute_url }})
* On the _Locale_ tab:
1. At least set the timezone.
2. Set the location if you'd like, the rest you can leave defaults

* It may ask you to reboot after these settings, select the option to reboot later, and connect to your WIFI before rebooting the Raspberry Pi.
1. Just click on the icon in the top right (again, I'm running on a VM, so it doesn't see the bound adapter as a wireless adapter) select your network, and authenticate

![Raspberry Pi  Wireless]({{ "img/2018-01-22-brewing-your-own-brewpi/rpi-wireless.jpg" | absolute_url }})
* Now reboot (the reboot option is under the Raspberry Menu, same place as step 1).

For the rest of the Raspberry Pi configuration, I'm going to be using a remote connection via SSH from my actual workstation, you can leave your keyboard and mouse plugged in, and use the bash terminal and run all the same commands.  I'm doing this because if your like me, you have an old rickety keyboard, some crappy monitor, and no mouse hooked up, so SSH is just easier.  Im not going to go into detail about SSH from a Windows machine, you can search that out if you'd like, there are more then a few ways to use SSH from Windows out there (none enabled by default unfortunately).  I am using the SSH client that comes with MINGW64, which I have installed because I use Jekyll and Ruby, which is what this blog is written with.

* Connect to the Raspberry Pi via SSH, the default user name is _pi_:
1. `ssh pi@brewpi`
2. Type `yes` to confirm the host certificate fingerprint.
3. Authenticate with the password you set earlier.

![Bash Prompt]({{ "img/2018-01-22-brewing-your-own-brewpi/ssh-connection.jpg" | absolute_url }})
* Update your Raspberry Pi (Packages and Firmware)
1. `sudo apt-get update`
2. `sudo apt-get upgrade`
3. `sudo apt-get install rpi-update`
4. `sudo rpi-update`
5. Reboot after all of these commands have completed `sudo reboot`.
* Clone the BrewPi source code repository from my GitHub, I have fixed some dependency issues that have not been merged into the parent repository.
1. `git clone https://github.com/dotps1/brewpi-tools.git ~/brewpi-tools`
2. `sudo ~/brewpi-tools/install.sh`
* You will see a bunch of things going accross the screen, just wait until it prompts you to answer some questions.  Follow the on screen prompts.  Default values or perfectly fine (just hit enter with the exception of over writing the `/var/www/html` directory, you need to put a `y` in for that one).
* After it is complete, you can browse to the system and check out the dashboard by going to `http://brewpi` (replace _brewpi_ with what ever you called yours.)

![Dashboard]({{ "img/2018-01-22-brewing-your-own-brewpi/rpi-dashboard.jpg" | absolute_url }})
* Thats it for setting up the Raspberry Pi.

---

### Wiring and Construction

Next, lets take a look at the general power schematic, The only thing missing from this sketch, is the Raspberry Pi.  This part just hooks up via USB to the Arduino.  The Raspberry Pi is the web server, its how you will interface with the BrewPi, setting temperatures and building profiles.  [Credit to FuzzeWuzze from HomebrewTalk for the schematic](https://www.homebrewtalk.com/forum/threads/howto-make-a-brewpi-fermentation-controller-for-cheap.466106/).  ![Schematic]({{ "img/2018-01-22-brewing-your-own-brewpi/schematic.jpg" | absolute_url }})  One other thing I did, was if you go look at the case I suggested you purchase from Amazon, it has a built in outlet.  This has many benefits, basically if you go power into that outlet, and then piggy back off it as the source in, you can then keep the power supplies for the Raspberry Pi and the Ardunio Uno inside the case, else you are going to have two temperature probes, and three power cables hanging out of this thing.  So you don't _have to_ go with that case, but I suggest you do.  One other perks, is it has a compartment below the panel, this gives you a place to hide wires.  So, I just played around with the layout of all my pieces inside my _sprinkler case_, until I got everything to kind of have a place, then I used small dabs of hot glue to hold everything in place.  Here is what it looked like before I glued everything in place: ![Case Layout]({{ "img/2018-01-22-brewing-your-own-brewpi/case-layout.jpg" | absolute_url }}).  Next before I glued everything, I cut holes for the specific outlets I purchased (link in _shopping list section_) in the lower right hand side, below the removable panel (sorry, the best picture of this I had was from the finished product).  ![Case Outlets]({{ "img/2018-01-22-brewing-your-own-brewpi/case-outlets.jpg" | absolute_url }}) after I got all the outlets in and secured, I then ran wires long enough for them to reach the relay, then I glued all the components and put the panel back in.  I did some other little customization, such as the usb cable from the Raspberry Pi to the Arduino Uno, I filed a little notch and ran that behind the panel, because it was a 5ft cord that needed to go 10 inches.  You can make yours as nice or as neat as you'd like.  Here is the finished product: ![Case Finished]({{ "img/2018-01-22-brewing-your-own-brewpi/case-finished.jpg" | absolute_url }})

---

### Configuring the BrewPi and Arduino

If you'd made it this far, congratulations, you're almost done, and its almost time for a beer.  The only thing left to do is to actually configure your BrewPi to talk to the Arduino Uno.
* First you need to roll back to the legacy version that supports the Arduino Uno
1. `sudo ~/brewpi-tools/updater.py`
