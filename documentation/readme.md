This activity uses BLE beacons & Raspberry Pis to create a high-tech treasurehunt. The following features have been implemented out of the box:

The treasurehunt script starts at boot nigating the need for familiaritty with Raspbian.
Completely audible to enable headless operation.
The first clue is sounded automatically. Subsiquent clues are triggered when a given beacon is in range.
Automatic shutdown when a given beacon is encountered to prevent file system corruption.

Checklist:

A BLE beacon for each location minus the starting one. Beacons should be compatible with Apple's iBeacon standard. We have had good results using Estimote (http://www.estimote.com) beacons in iBeacon mode.
Enough Raspberry Pi 3's, speakers & power sources (E.G. power bars) for everyone. You may be able to use other SBC's so long as you are able to use Bluetooth & run Python but some of the code is Pi-specific. NB: we had relatively good success in splitting partisipants into groups & giving each group a Pi, but if headphones are being used watch out for trailing wires!
A modified treasurehunt.py (it is very unlikely that the supplied code will work out of the box for you).
To ensure the smooth running of the activity it would be preferable if you are able to familiarise yourself with the location before hand (see the planning a root section below).

Understanding how iBeacons work:

Genericly speaking, beacons emit a packet every NMS for clients to consume & act upon. The contents of said packets vary between standards and are largely beyond the scope of this document but it is however important to take a moment to develop a basic understanding of how iBeacon packets are structured in preperation for the activity.

The parts of an iBeacon "advertising packet" that we are concerned about are as follows:

Universally unique identifier or UUID: 32 hexadecimal didgits split into 5 groups.
Major: an integer between 0 & 65535 inclusive.
Minor: an integer between 0 & 65535 inclusive.

NB: there are other potentially useful pieces of information in a packet although due to time constraints we have not taken these into account as of yet. We are however always open to pull requests.

A worked example:

Suppose you are a supermarket chain who wishes to provide an app-based indoor navigation experience for your customers. Your chain owns multiple supermarkets & within each supermarket there are multiple iles of items. In this scenario, the above values would most likely be mapped similarly to the following:

UUID represents your chain.
Major represents a given supermarket.
Minor represents an ile within a supermarket.
Note that in this instance minor values would not have to be exclusive accross supermarkets so long as the combination of the UUID, major & minor values are able to uniquely identify a beacon.

As you may have sermized, the iBeacon standard is well-suited to larger roleouts and is rather overkill for this activity. for our implementation we chose consistant UUID & major values, but if your location has multiple floors for example, there is no reason why major could not denote a floor & minor a specific location on said floor.

Beacon setup:

The following steps are based on our experiences with Estimote beacons but the steps should largely be the same regardless of the manifacturer:

1. Change the name of all of your beacons to something sensible. We chose Beacon1, Beacon2, ... Beacon11, Beacon12. Note that these names are only used internally & aren't transmitted in advertising packets. Estimote lets you do this via their app or the cloud control panel.
2. Take each beacon & write its name on it for easy identification later. We found the best way to do this was to move a beacon away from the rest of the beacons then use the Estimote app to show nearby beacons.
3. For each beacon you should additionally:
decrease its range. Estimote lets you specify the broadcast power as a percentage or in feet - we found that 0 percent (3 feet) was a sensible starting range.
Change the major & minor values as per your requirements.

Setting up a Pi:

These steps assume that you have a working Raspbian install & you are reasonably familiar with how it operates. Steps 1, 2 and 3 should be completed before you carry out the steps in the "planning a root" section. Steps 5 & 6 should only be performed when you are happy with your root & you have modified hunt.py to work with your setup.

1. Copy the 3 .py files to a sensible location.
2. For debugging during the planning a root section we highly recommend you connect the Pi to a wireless network.
3. Install dependencies (need to figure out what dependencies we installed)
4. Copy all of the sounds into the directory in step 1.
We now need to make hunt.py run when the Pi boots. To do this:
5. Run
raspi-config
choose option 3 then b4.
6. Edit
/home/pi/.config/lxsession/LXDE-pi/autostart 
where pi is the name of your user account. You will need to sudo to modify this file.
add the following line:
xxxx (need to confirm this)
The setup on the Pi is now complete.

Planning a root:

The effectiveness of wireless transmition varies drasticly depending on the devices in use, other nearby devices & your location. It is there for important to resist the urge to assume that once you have everything working in one location it will perform even remotely identicly at the location you wish to run the activity in.
We found that a treasurehunt that contained 11 clues took just over 5 hours of setup for 2 people with very few interuptions. Our experiences suggest that more people would not necessarely decrease the setup time but familiarity with the location may do. We recommend that a dry run is carried out the day before the event & that the beacons are left in situ overnite once everything is setup if at all possible.

1. Place the beacons in the desired locations.
2. We now need to make sure that the range of the beacons is appropriate & that none of them overlap; this is probably the longest & most frustrating part of the setup process but stick with it! You will need a way of receiving output from a Pi whilst walking about; we used a laptop connected via SSH via a phone that was broadcasting a wireless network. Once everything's working, run
python debug.py
and consult the output.
This script is a minimal version of the treasurehunt which, as the name implies, is designed for debugging your root. When a beacon is in range its major & minor values are printed, although if multiple packets are received in a ro from one beacon only the first is displayed. If you have modified a given beacon and wish to test its range again, you will need to walk to the sight of another beacon then walk back to the original one. The Shutdown functionality has not been implemented in this script.
3. Walk your root taking care to pass the clues in the same order that the partisipants will do whilst checking that you are happy with the performance of the beacons. Pay paticular attention to scenarios where 2 beacons are close to each other, consulting the output from the Pi to make sure that it is not possible for both of them to be detected at the same time. If this does happen, experiment with repositioning one of the beacons slightly - E.G. high up, low down, inside a cupboard, behind a door etc.  We found the Estimote companion app useful for modifying the range of individual beacons on the go for scenarios where "3 feet" was actually considerably less due to the environment. Please also be mindful of scenarios where partisipants may pass a clue on root to another one - E.G. try to avoid clue 5 sounding when partisipants are on there way to clue 2.
4. Once you're happy with everything you will have most likely walked the root a large number of times and will have started to question why you committed to do the activity in the firstplace. Never the less, it is important to carry out one last check by starting at the begining & walking it through to confirm that no more adjustments need to be made. Keep on doing this until you can walk the root without changing anything.

Supplied .py files:

3 .py files are supplied with this activity:

bluescan.py - methods to scan for nearby BLE devices. Partisipants need not concern themselfs with the content of this file.
debug.py - a minimal implementation of the treasure hunt activity for debuggin purposes.
hunt.py - the main treasure hunt file that partisipants should modify.

Understanding hunt.py:

Whilst the code is well commented, a breakdown of the various methods & sections of the file can be found below:

