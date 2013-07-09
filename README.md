launchd-keepalive
=================

Mac OS X plist files for use with launchd demonstrating how to use KeepAlive

* [at.obdev.LittleSnitchUIAgent.plist](at.obdev.LittleSnitchUIAgent.plist) is a basic 'KeepAlive' plist.

Note the 

		<key>KeepAlive</key>
		<true/>

And

		<key>RunAtLoad</key>
		<true/>

Together, those tell `launchd` to run the program listed in `ProgramArguments` as soon as the user logs in, and keep it running as long as they are logged in, no matter what:

		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
		<plist version="1.0">
		<dict>
			<key>KeepAlive</key>
			<true/>
			<key>Label</key>
			<string>at.obdev.LittleSnitchUIAgent</string>
			<key>ProgramArguments</key>
			<array>
				<string>/Library/Little Snitch/Little Snitch Agent.app/Contents/MacOS/Little Snitch Agent</string>
			</array>
			<key>RunAtLoad</key>
			<true/>
		</dict>
		</plist>

Now if `Little Snitch Agent` is terminated for any reason (i.e. crash or `kill`) it will automatically restart.

At first you might be tempted to create several plists like this to keep all of your favorite apps running all of the time. 

***Don't do that.***

Think about what happens if an app you have designated as  `KeepAlive` needs to be updated. What happens? The app tells you it has an update, you download it, the app quits and then -- uh, it restarted. Did it have a chance to finish updating? Are you sure? 

Occasionally, `KeepAlive` apps can interfere with rebooting or logging out.

In most cases, a better solution is to use `KeepAlive` together with `SuccessfulExit`.

### KeepAlive *and* SuccessfulExit

For example [com.tjluoma.keeprunning.moveaddict.plist](com.tjluoma.keeprunning.moveaddict.plist) shows how to create a plist to keep an app running all of the time *unless* it exits cleanly (i.e. the user told it to quit). It uses `KeepAlive` but adds a check called `SuccessfulExit.` If an app exits "successfully" (technically, with an exit code = 0) then the app will not be automatically restarted (kept alive). However, if the app crashes (exit code not equal to 0) then it will be automatically restarted. Here is the relevant bit of `launchd` code:

		<key>KeepAlive</key>
		<dict>
			<key>SuccessfulExit</key>
			<false/>
		</dict>

This is handy if you have an app which occasionally crashes, especially if that app is one which runs in the background or the menu bar where you might not notice immediately.

### `KeepAlive` and `NetworkState`

Another option is `KeepAlive` and `NetworkState` as shown in  [com.tjluoma.keeprunning.mail.plist](com.tjluoma.keeprunning.mail.plist) 
which tells Mail.app to always keep running as long as we have a network connection.

Here is the portion of the plist which specifically deals with the "is the network up?" part:


		<key>KeepAlive</key>
		<dict>
			<key>NetworkState</key>
			<true/>
		</dict>

**Important note #1:** `launchd` considers the "network" to be "up" if you have an IP address. However, it is possible that your local network could be up but your connection to the Internet is down. For example, right now my ISP is offline, but I am connected to my local Wi-Fi network, so as far as `launchd` is concerned, the "network" is up. Just remember that "Network is up" does not necessarily mean "Internet is up/accessible."

**Important note #2:** It is also important to remember that `launchd` will not *quit* an app just because the network connection goes down. The only time the `KeepAlive` would be used is if the app quits, at which point `launchd` will check and say "Is the network up?" and if the answer is yes, it will relaunch the app. If the answer is no, it will not relaunch the app.

### Installation ###

Unless otherwise noted, plists should be placed in the "$HOME/Library/LaunchAgents" (where "$HOME" represents the path to your home directory, for example: **/Users/sjobs/Library/LaunchAgents/**

### How to start a new launchd plist  

By default, `launchd` will load the '.plist' files from "$HOME/Library/LaunchAgents" when you log in. If you want to start a new .plist, you will have to tell `launchd` to load it. To do this, launch Terminal.app and type:

		cd "$HOME/Library/LaunchAgents"

And then:

		launchctl load com.tjluoma.keeprunning.mail.plist

(change 'com.tjluoma.keeprunning.mail.plist' to the filename of whichever plist you want to load)

### How to tell `launchd` that you have changed an existing plist

If you have changed an existing plist and want the changes to be recognized right away, you have to unload it, and then reload it:

		cd "$HOME/Library/LaunchAgents"

		launchctl unload com.tjluoma.keeprunning.mail.plist

		launchctl load com.tjluoma.keeprunning.mail.plist


### How to uninstall / remove a plist

Again, in Terminal.app, do:

		cd "$HOME/Library/LaunchAgents"

		launchctl unload com.tjluoma.keeprunning.mail.plist

And then either move it to the trash:

		mv com.tjluoma.keeprunning.mail.plist ~/.Trash/

Or delete it:

		rm -i com.tjluoma.keeprunning.mail.plist

