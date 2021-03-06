# RatAttack #

This profile facilitates [Fuel Ratting](https://www.fuelrats.com). It aims to 
eliminate as much of the required manual task and attention switching as 
possible via automation and voice commands.

If you don’t know what the Fuel Rats are, come hang out and ask :)

## Requirements ##

Only vanilla VoiceAttack is absolutely required to use this profile. Optionally 
you can install EDDI and my elite scripts for advanced features.

* [EDDI](https://github.com/EDCD/EDDI) installed as a VoiceAttack plugin: This 
  will give you a better (IMO) way of using TTS. Be sure to set 
  `RatAttack.useEddiForVoice`. It will also enable you to have ingame chat be 
  transferred to IRC; see below.
* [elite-scripts](https://github.com/alterNERDtive/elite-scripts): Using the 
  Python scripts will give RatAttack a way to be aware of where your CMDRs are 
  and give you the nearest one to a rat case. That’s only really needed if you 
  actually _have_ multiple CMDRs, obviously. If you are using the profile 
  package from the release page, they will be installed automatically.

### EDDI speech responder ###

For the convenience of people that have not been using EDDI in the past, 
RatAttack will deactivate the speech responder automatically to not clutter them 
with unwanted TTS.

If you are already an EDDI user and want to keep the default speech responder 
functionality, you will have to disable the `EDDI.quietMode` setting by running 
the `customize settings disable eddi quiet mode` command.

## Settings ##

There are a lot of preferences you can set, including some you really want to 
concern yourself with before you start using the profile. Some of the more 
advanced features heavily rely on you giving it the correct things to work with.

See the [Configuration Variables](#Configuration-Variables) section.

## Including the Profile ##

When including the profile, be sure to

* Run the startup command. You will need to have a startup command in your 
  profile (= one that is run on profile loading) and call `RatAttack.startup` 
  from that one.
* Make sure all EDDI events that RatAttack needs are correctly handled. For all 
  events used in RatAttack that you already have handlers for in your profile, 
  you’ll have to include a call to `RatAttack.<event name>`. E.g.  for “EDDI 
  Message sent”, call `RatAttack.EDDI Message sent` by name from your `((EDDI 
  Message sent))` command.

## Usage ##

### Going On/Off Duty ###

When you are on duty, RatAttack will automatically announce cases coming in 
through IRC. When off duty, it won’t.

* `[enable;disable] rat duty`: puts you on/off duty.
* `open [rat;] dispatch board`: opens the web dispatch board.

### Handling a Case ###

#### Getting Case Data From IRC ####

You can setup your IRC client to pass incoming RATSIGNALS to VoiceAttack by 
writing them to a file (`%appdata%\Ratattack\ratsignal.pipe`), then calling the 
appropriate command (`RatAttack.announceCaseFromRatsignal` for notification, 
`RatAttack.getInfoFromRatsignal` for silently putting it into the case list).

This has two purposes:

1. announcing a new incoming case
1. storing case data and making it available to VoiceAttack, e.g. for copying 
   the case’s system into the clipboard

You need to make your IRC client

1. wait until the file disappears (for several cases coming in at once)
1. write the RATSIGNAL to the file
1. run the VoiceAttack command

In my case I am running AdiIRC and have the following script setup for handling 
this:

```
on *:TEXT:RATSIGNAL - CMDR*(??_SIGNAL):#fuelrats:{
	/mkdir C:\users\<user>\appdata\roaming\RatAttack\
	/handleratsignal $1-
}
alias handleratsignal {
	if ( $exists(C:\users\<user>\appdata\roaming\RatAttack\ratsignal.pipe) ) {
		/sleep 1 /handleratsignal $1-
	}
	else {
		/write C:\users\<user>\appdata\roaming\RatAttack\ratsignal.pipe $1-
		if ( $away ) {
			/run -h "X:\path\to\VoiceAttack\VoiceAttack.exe" -nofocus -command "RatAttack.getInfoFromRatSignal"
		}
		else {
			/run -h "X:\path\to\VoiceAttack\VoiceAttack.exe" -nofocus -command "RatAttack.announceCaseFromRatSignal"
	    }
	}
}
```

You get the gist; if not and you don’t know how to do the same thing for your 
IRC client or it doesn’t support copying the control characters in the ratsignal 
that the profile uses to split the information, either switch to AdiIRC or bribe 
me to include some other way to get case data into VoiceAttack.

**Note**: If you are running VoiceAttack as admin you need to run your IRC 
client as admin, too! Otherwise it can’t talk to VoiceAttack for security 
reasons. You really should _not_ run VoiceAttack with elevated privileges 
though. Or anything.

#### Internal Case List ####

If you have your IRC client setup properly, VoiceAttack will hold a list with 
all rat cases that have come in while you had it running. It will save the case 
number, CMDR name, system, O₂ status and platform. There are several commands 
you can run on this list, giving it a case number:

* `rat case number [0..19] details`: Will give you all stored info on a case.
* `[current;] rat case details`: Will give you all stored info on the currently 
  open case.
* `distance to current rat case`: Will give you the distance from your current 
  location to the currently opened rat case. Requires the use of my 
  `elite-scripts` Python scripts.
* `distance to rat case number [0..19]`: Will give you the distance from your 
  current system to a case’s system. Requires the use of my `elite-scripts` 
  Python scripts.
* `nearest commander to rat case number [0..19]`: Will give you the nearest of 
  your CMDRs with their distance to a case’s system. Requires some setup and the 
  use of my `elite-scripts` Python scripts.
* `nearest commander to [the;] rat case`: Will give you the nearest of your 
  CMDRs with their distance to the current case’s system. Requires some setup 
  and the use of my `elite-scripts` Python scripts.

#### Opening a Case ####

* `open rat case number [0..19]`: Opens rat case with the given number. If there 
  is no case data for that case (e.g. because you don’t have your IRC client set 
  up properly), it will still open it, just not have any data on it.
* `open [latest;] rat case`: Opens the latest rat case that has come in through 
  IRC. Will probably error out in creative ways if you don’t have your IRC 
  client set up properly. Too tired right now to have proper error handling so 
  just open an issue if you run into problems (it’s 7am, I haven’t slept and 
  want to finish this doc to get the release out (yes, you are allowed to laugh 
  at this section)).

#### Closing a Case ####

* `[close;clear] rat case`: Closes the currently open rat case.

### Making Calls ###

There are a bunch of calls you can make for a case, the most common are modelled 
through VoiceAttack commands. The descriptive commands (e.g. “system confirmed”) 
will be shortened to the usual IRC short hands (e.g. “sysconf”). If you need 
something more unusual you can either still manually type it into your IRC 
client or use the “General IRC Integration”, see below.

* `call [1..20] jumps [and login;and takeoff;left;]`: Calls jump for the 
  currently open case. You can optionally include that you will still have to 
  login to the game or have to take off from your current 
  station/port/outpost/planet.
* `call friend [positive;negative] [in pg;in private group;in solo;in main 
  meu;sysconf;system confirmed]`: Friend request confirmations, with all the 
  things you might want to / should call with it.
* `call [beacon;fuel;instance;pos;position;prep;sys;system;wing] 
  [positive;negative]`: All the stuff you usually need for ratting after you 
  have received the friend request.
* `call wing pending`: Calls “wr pending” for when it takes 30s again to 
  actually show up.
* `call client in [exclusion zone;main menu;open;open sysconf;pg;private 
  group;solo;super cruise]`: Callouts for all the various things a client could 
  get themselves into.
* `call [client destroyed;client offline;sysconf;system confirmed]`: This is the 
  command you don’t want to use. Include sysconf in your “friend+” or “in open” 
  calls, and make sure you will never have to call “client destroyed”, would 
  you?

### General IRC Interaction ###

(requires EDDI)

Using EDDI to read the game’s journal, you can send messages to IRC from Elite’s 
ingame chat.

**Be aware that the chat message will still appear in the ingame chat channel 
you send it to!**

I recommend using local chat and limiting the use to instances that will 
probably not have other players in it (e.g. instanced in normal space with the 
client or in SC in some remote system out in the black on a long range rescue).

* #fuelrats: Use “.fr \<message\>” to have VoiceAttack send “#\<caseNumber\> 
  \<message\>” to the #fuelrats channel – or yell at you when you are not on 
  a case.
* #ratchat: Use “.rc \<message\>” to have VoiceAttack send “\<message\>” to 
  #ratchat.

These commands send their text to windows with “#fuelrats” and “#ratchat” in 
their title, respectively. If your IRC client does not do that, you will have to 
change the “target” window of the `RatAttack.sendToFuelrats` and 
`RatAttack.sendToRatchat` commands to reflect the actual window titles on your 
system. I will look into making this more elegant to change in the future.

## Logging ##

The profile supports logging a bunch of stuff to the VoiceAttack event log. By 
default, logging is concise and constrained to basically error messages.

If you need more logging (usually for debugging purposes), say `enable logging`. 
If you want to enable verbose logging _by default_, call the 
`Logging.enableLogging` command from your custom profile’s `startup` command.

## Exposed Variables ##

The following Variables are _global_ and thus readable (and writeable! Please 
don’t unless it’s a config variable …) from other profiles.

### Configuration Variables ###

There are a bunch of configuration variables. You should not overwrite those 
manually, instead use the provided commands in the `_configuration` section!

Basically all the settings are available using the `customize settings` prefix, 
then saying `[enable;disable] <setting>` for on/off switches and `set <setting>` 
for text variables.

* `EDDI.quietMode` (boolean): whether or not to set EDDI to quite mode. Default: 
  true.
* `EDDI.useEddiForVoice` (boolean): whether to use the EDDI plugin to handle 
  text-to-speech over VoiceAttacks built-in speech function. Default: false.
* `Elite.pasteKey` (string): the key used for pasting into Elite. On QWERTY this 
  is `v`. Default: `v`.
* `RatAttack.announceNearestCMDR` (boolean): whether or not to automatically 
  announce your nearest CMDR to a case.  Requires the `elite-scripts` Python 
  scripts. Will probably break in creative ways if you don’t have them and turn 
  it on anyway.  Default: false.
* `RatAttack.announcePlatform` (boolean): whether or not to announce the case’s 
  platform by default. Useful to set if you are active on more than one 
  platform. Even with this off, you will still be warned when you open a case 
  that is _not_ on one of your platforms.  Default: false.
* `RatAttack.CMDRs` (string): list of your CMDR names, delimited by spaces. If 
  your names include spaces, you will have to put them in quotes. Default: “"J 
  Jora Jameson" NameWithNoSpace”.
* `RatAttack.confirmCalls` (boolean): whether VoiceAttack should ask you before 
  posting to #fuelrats to make sure there hasn’t been an error in voice 
  recognition and you accidentally post the wrong thing. Default: true.
* `RatAttack.autoCloseCase` (boolean): whether or not to automatically close an 
  open rat case on calling “fuel+”. Default: false.
* `RatAttack.onDuty` (boolean): whether or not you are currently on rat duty. 
  Default: true.
* `RatAttack.platforms` (string): the platforms you want to be informed of 
  incoming cases for. If you are on console, you can still have VoiceAttack 
  running on the PC that you are using for IRC and handle calls and stuff using 
  voice!  Delimited by whatever you want. Can include “PC”, “XB”, “PS4”. 
  Default: “PC”.
* `python.scriptPath` (string): the path you put the Python scripts in. Default: 
  “{VA_DIR}\Sounds\scripts”.

### Other Variables ###

Current case data:

* `RatAttack.caseNumber` (int): the number of the case you are currently on. 
  Will be `Not Set` if you are not on a case.
* `RatAttack.onCase` (boolean): whether or not you are currently on a case.

Case list:

* `RatAttack.caseList.<case#>.cmdr` (string)
* `RatAttack.caseList.<case#>.system` (string)
* `RatAttack.caseList.<case#>.platform` (string)
* `RatAttack.caseList.<case#>.codeRed` (boolean)

… with `<case#>` being a number between 0 and 19.
