linino-to-serial
=====

About
----
A wrapper for `pyserial` on the Arduino Yun, which enables users to send messages from Linino to ATmega. It is modified from the Python code sample posted by user [wayoda](http://forum.arduino.cc/index.php?action=profile;u=2322) on the [Arduino forums](http://forum.arduino.cc/index.php?topic=191820.msg1422155#msg1422155).

Installation
----
Copy `linino-to-serial` to a directory in your `PATH`. Use it like this:

    linino-to-serial PORT BAUDRATE MSG

For reference, the port `/dev/ttyATH0` will connect you to the ATmega side.

If you don't have `pyserial`, install it using `pip install pyserial`. (If you don't have `pip`, [follow these instructions](http://samjbrenner.com/notes/using-pip-to-install-python-packages-on-the-arduino-yun/).)

Cookbook
----

On the ATmega (Arduino sketch) side of the Yun, listen for messages like this:

```c
char lininoChars[4] = "000";

void setup() {
    Serial1.begin(250000);
}

void loop() {
    while (Serial1.available() > 0) {
        char c = (char)Serial1.read();
        bufferLininoCommunication(c);
    }
    
    if (hasLininoSaid("SAM")) {
        // shift the buffer so this doesn't get called over and over
        bufferLininoCommunication('0');
        
        // now do something
    } else if (hasLininoSaid("BEN")) {
        bufferLininoCommunication('0');
        // do something else
    }
}

void bufferLininoCommunication(char nextChar) {
  // store the last three chars we've received over serial from linino
  lininoChars[0] = lininoChars[1];
  lininoChars[1] = lininoChars[2];
  lininoChars[2] = nextChar;
}

// http://stackoverflow.com/questions/20169474/char-array-comparison-in-c
short hasLininoSaid(char sequence[]) {
  int i;
  
  for(i = 0; i < strlen(lininoChars); i++) {
    if(lininoChars[i] != sequence[i]) {
      return 0;
    }
  }
  
  return 1;
}
```

This assumes a "message" is a three-character string that is known in advance to both sides of the Yun. Now you can use `linino-to-serial` for almost anything! For example:

#### Knowing when the Yun has booted up

On Linino, add the following line to `/usr/bin/boot-complete-notify`:

    linino-to-serial /dev/ttyATH0 230400 "%%%"

In the `loop()` method you can now check `hasLininoSaid("%%%")`. Pair this with a `boolean contactWithLinino = false;` that gets updated to `true` when contact is made.

#### Cron Jobs

Follow the instructions [here](http://martybugs.net/wireless/openwrt/cron.cgi) to get cron running on your Yun. Use `0 * * * * linino-to-serial /dev/ttyATH0 230400 "ABC"` to send the message "ABC" to your ATmega on the hour, every hour. Adjust to taste!

Known Issues
----
For reasons I don't entirely understand, the only way I've been able to translate clean messages (without garbling) is by listening on `Serial1` with a baud rate of 250000, but sending messages from Linino with a baud rate of 230400. It works just fine like this so it's not really an "issue," but unexpected behaviour nonetheless. Leave a comment if you know what's going on!

More Reading
----
http://forum.arduino.cc/index.php?topic=191820.msg1422155#msg1422155
http://pyserial.sourceforge.net/
