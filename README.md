linino-to-serial
=====

What?
----
A wrapper for `pyserial` on the Arduino Yun, which enables users to send messages from Linino to ATmega. It is modified from the Python code sample posted by user [wayoda](http://forum.arduino.cc/index.php?action=profile;u=2322) on the [Arduino forums](http://forum.arduino.cc/index.php?topic=191820.msg1422155#msg1422155).

How?
----
Copy `linino-to-serial` to a directory in your `PATH`. Use it like this:

    linino-to-serial PORT BAUDRATE MSG

For reference, the port `/dev/ttyATH0` will connect you to the ATmega side.

If you don't have `pyserial`, install it using `pip install pyserial`. (If you don't have `pip`, [follow these instructions](http://samjbrenner.com/notes/using-pip-to-install-python-packages-on-the-arduino-yun/).)

On the ATmega (Arduino sketch) side of the Yun, listen for messages like this:

```
void setup() {
    Serial1.begin(250000);
}

void loop() {
    if (Serial1.available()) {
        char c = (char)Serial1.read();
        // handle the serial messages as you wish.
    }
}
```

For reasons I don't entirely understand, the only way I've been able to translate clean messages (without garbling) is by listening on `Serial1` with a baud rate of 250000, but sending messages from Linino with a baud rate of 230400. Leave a comment if you know what's going on!

Why?
----
This is my solution to the "how will my Arduino code know when linino has booted up?" problem. To `/usr/bin/boot-complete-notify` on the Linino side of the Yun, I added:

    linino-to-serial /dev/ttyATH0 230400 "%%%"

Then I wait until I have recieved three `%` signs over `Serial1` on the ATmega side. This probably isn't the most reliable method, but it hasn't failed me yet (the Linino boot process doesn't include a `%` character, at least on my Yun).

More Reading
----
http://forum.arduino.cc/index.php?topic=191820.msg1422155#msg1422155
http://pyserial.sourceforge.net/
