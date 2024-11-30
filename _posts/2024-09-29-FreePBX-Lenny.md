---
title: "FreePBX (Asterisk) Lenny"
categories: [software, freepbx, voip, asterisk]
tags: [software]     # TAG names should always be lowercase
image:
  path: https://www.freepbx.org/wp-content/uploads/Sangoma_FreePBX_Logo_RGB_hori-pos-e1588854523908.png{:width="50%"}
---

I thought I'd post a clever routine I've been running for years and years now.
It's called "It's Lenny"

It's basically a set of recordings that plays one after another in-between bursts of talking from the caller.

In-essence, it's something that you can blacklist telemarketers to and they can talk to him all day long if they like (Well, a good 40mins...ish)

It used to be hosted at a SIP server online but it went offline and people sarted hosting it themselves and the audio files went into distribution. It's not clear as to any copyright or use terms.

This is NOT my work on the sound files and no copyright is meant to be infringed.

I have, however, changed the dialplan quite significantly.

It's not perfect and I'm not the worlds-best dialplan creator but it's a place for you to start.

I created the Lenny extension on ext 7000.
Then I added the below code to my /etc/asterisk/extensions_override_freepbx.conf

## Asterisk Dialplan

```
; Lenny Extension - Used in blacklist dialplan

[from-internal-custom]
include => newlenny

[newlenny]

exten => 7000,1,Set(SOUND=1)
exten => 7000,2,Set(HELLOLOOP=0)
exten => 7000,3,Set(CALLFILENAME=ItsLenny-${CALLERID(num)}-${STRFTIME(${EPOCH},,%Y-%m-%d_%H-%M-%S)})
exten => 7000,4,MixMonitor(/var/spool/asterisk/monitor/itslenny/${CALLFILENAME}.wav)
exten => 7000,5,Ringing()
exten => 7000,6,Wait(4)
exten => 7000,7,Answer
exten => 7000,8,System(/var/lib/asterisk/bin/sendmail-lenny.sh youremail@example.com "${STRFTIME(,,%c)}" "${CALLERID(num)}" "${EXTEN}" "${EPOCH}")
exten => 7000,9,Playback(itslenny/Lenny${SOUND})
exten => 7000,10,Set(SOUND=$[${SOUND} + 1])
exten => 7000,11,Goto(loop,7000,1)
exten => talk,1,Goto(loop,7000,1)
  
[loop]

exten => 7000,1,BackgroundDetect(silence/10,1600,200,110000)
exten => 7000,2,Goto(helloloop,7000,1)
exten => talk,1,Set(HELLOLOOP=0)
exten => talk,2,Playback(itslenny/Lenny${SOUND})
exten => talk,3,Set(SOUND=$[${SOUND} + 1])
exten => talk,4,NoOp(Sound PLAYING is ${SOUND})
exten => talk,5,GoToIf($["${SOUND}" = "16"]?playcrap-and-hangup,7000,1:loop,7000,1)
  exten => 7000,3,Hangup()


[helloloop]
; When no talking is detected, Lenny says hello 3 times and then hangs up.

; Below line reserved for future use with different Hello files - just to vary things a bit - Last hello could be "Ahh well, no-one there" or similar

;exten => 7000,1,Playback(hellos/${HELLOLOOP})

exten => 7000,1,Playback(itslenny/Lenny-Hello)

exten => 7000,2,Set(HELLOLOOP=$[${HELLOLOOP} + 1])

exten => 7000,3,GoToIf($["${HELLOLOOP}" = "3"]?dohangup,7000,1:loop,7000,1)

  

[playcrap-and-hangup]
# These files are usually found in /var/lib/asterisk/sounds/en so no need to define unless using a different language.

exten => 7000,1,Wait(1)
exten => 7000,2,Playback(spam)
exten => 7000,3,Wait(1)
exten => 7000,4,Playback(tt-monty-knights)
exten => 7000,5,Wait(1)
exten => 7000,6,Playback(the-monkeys-twice)
exten => 7000,7,Playback(tt-monkeys)
exten => 7000,8,Playback(why-no-answer-mystery)
exten => 7000,9,Playback(lots-o-monkeys)
exten => 7000,10,Wait(1)
exten => 7000,11,Playback(not-taking-your-call)
exten => 7000,12,Hangup

  

[dohangup]

exten => 7000,1,Hangup

```


## Send notification email to user

I built this file which I store at: /var/lib/asterisk/bin/sendmail-lenny.sh

```
#!/bin/sh

#$1 email address
#$2 time
#$3 CallerID
#$4 CallLine
#$5 EPOCH

TMPFILE=/var/spool/asterisk/tmp/$5

echo "TO: "$1 >> $TMPFILE
echo "FROM: asterisk@yourdomain.com" >> $TMPFILE
echo "Subject: New Lenny call recieved " >> $TMPFILE
echo ""
echo "Just wanted to let you know, you received a new call for Lenny." >> $TMPFILE

echo "Time: "$2 >> $TMPFILE
echo "Calling number: "$3 >> $TMPFILE
echo "" >> $TMPFILE
echo "You Can Listen At: smb://freepbx/recordings-lenny" >> $TMPFILE # Set this to your share

echo "" >> $TMPFILE
echo "Enjoy" >> $TMPFILE
echo "" >> $TMPFILE
echo "." >> $TMPFILE
/usr/sbin/sendmail $1 < $TMPFILE
rm $TMPFILE
```

## Sound Files

These can be found here:
[Lenny Sound Files](https://github.com/lloydbayley/lloydbayley.github.io/blob/main/files/lenny.tgz)

I have them in /var/lib/asterisk/sounds/itslenny
Make sure they are owned by asterisk and have the appropriate permissions. (Probably 664)

Have fun. It's been MANY years since I fiddled with this, so documenting it was a bit tricky but I think it will make sense.
