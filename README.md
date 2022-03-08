Sailfish Button Monitor

| Device            | Status       |
| ----------------- | ------------ |
| Xperia X          | Working OOTB |
| Xperia X Compact  | Working OOTB |
| Xperia XA2        | Working OOTB |
| Xperia XZ2 Compact| Working OOTB |
| FxTec Pro1        | Manul Tweak  |
| (everything else) | ???          |

----------
Copyright (c) 2020 Elliot Wolk

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

You should have received a copy of the GNU General Public License
along with this program.  
If not, see <http://www.gnu.org/licenses/>.

----------

```
Perform actions when Sailfish OS hardware buttons are pressed.

Supports volume up/down and camera half/full keys,
  when screen is locked or unlocked.

Can run arbitrary shell commands, or a few built-in commands.

Works best with automatic display-on disabled (requires mcetool).
  Install mcetool and run the following,
  to prevent unlocking when camera/volume is pressed:
    mcetool --set-exception-length-volume=0
    mcetool --set-exception-length-camera=0

./sf-button-monitor -h | --help
  display this message

./sf-button-monitor
  -listen to /dev/input/by-path/platform-gpio_keys.105-event
  -parse bytes into button press/release events
  -split sequences of events into patterns
  -read actions and patterns from /home/wolke/.config/sf-button-monitor.conf
  -perform actions that match event patterns and system conditions


actions are recorded in /home/wolke/.config/sf-button-monitor.conf
  one per line, formatted like:
  action=ACTION,PTRN,CONDITION

ACTION    = cmd(<CMD>) | torch | screenshot
            | reboot | shutdown | openCamera | selfie
            | newAlarm | newNote | writeEmail
              action to perform
                cmd:         run shell command <CMD>
                torch:       toggle flashlight
                screenshot:  take a screenshot in ~/screenshot-<MILLIS>.png
                reboot:      reboot phone
                shutdown:    shutdown phone
                openCamera:  open the camera app (in previous mode)
                selfie:      open the camera app in selfie mode
                newAlarm:    open app to add alarm
                newNote:     open app to add note
                writeEmail:  open app to write email
CMD       = <any-string>
              any shell command: EXEC ARG ARG...
CONDITION = always | screenLocked | screenUnlocked
            | app(CMD_REGEX) | home | noapp | anyapp | android
              when to allow the command to run
                always:         always
                screenLocked:   when display is off and screen is locked
                screenUnlocked: when display is on and screen is unlocked
                app:            when cmd of topmost window process matches CMD_REGEX
                home:           when screenUnlocked and there is no topmost window
                noapp:          when there is no topmost window
                                  same as: 'home' or 'screenLocked'
                anyapp:         when there is any topmost window
                                  same as: 'app()'
                android:        when topmost window is aliendalvik
                                  same as: 'app(^system_server$)'
CMD_REGEX = <any-string>
              regular expression used to match process cmd returned by ps

BTN_PTRN  = <EVENT> | <EVENT> <BTN_PTRN>
              a full pattern string containing button press/releases
EVENT     = <PRESS> | <RELEASE> | <CLICK> | <GROUP>
              a button press and/or release
PRESS     = <BTN>-press
              pressing button named <BTN>
RELEASE   = <BTN>-press
              releasing button named <BTN>
CLICK     = <BTN>
              same as: <BTN>-press <BTN>-release
GROUP     = <BTN>() | <BTN>( <BTN_PTRN> )
              same as: <BTN>-press <BTN_PTRN> <BTN>-release
BTN       = vu | vd | ch | cf
              the name of a physical hardware button
                vu: volume up
                vd: volume down
                ch: camera half
                cf: camera full

BTN_PTRN is a sequence of button presses/releases, with optional short synonyms
e.g.:
   vd-press vd-release =>
       vd-press vd-release
   vd =>
       vd-press vd-release
   vd() =>
       vd-press vd-release
   vd vd =>
       vd-press vd-release vd-press vd-release
   vd(vu) =>
       vd-press vu-press vu-release vd-release
   ch(vd vd(vu vu()) cf) =>
       ch-press vd-press vd-release vd-press
       vu-press vu-release vu-press vu-release
       vd-release cf-press cf-release ch-release
```
