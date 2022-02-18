# p1-parasitic
Project with the goal to build a wifi and/or BLE enabled P1-gateway

## P1-gateway

I have a smart meter and want to be able to access readings from it in a (near) realtime fashion. 

## Parasitic

I currently don't have an additional power outlet near the meter so a parasitic device, running of the power supplied by the P1 port would be preferable.

The power supplied by the P1 port is limited, so that is something to take into account.

## ESP32 / M5stick

I have a M5stick laying around with which I've managed to read from the P1 port and post to an MQTT topic before. This also has the benefit of having a battery which might be used as a buffer to supply current for the peak power required to send data over WiFi, whilst charging off the P1 power supply when the M5stick is dormant.

## Smart meter

The meter in my home is a ISKRA AM550-TD2 3-phase meter.

### P1 port

I have confirmed that the P1 port has a 5V power supply on pin #1 and #6.

:question: will the P1 port be able to supply enough power to charge the battery of the M5stick
:question: how much current can the P1 supply?
:question: how much current is required to get the M5stick to charge it's internal battery?

As I don't have the equipment at hand, nor the knowledge to test the max current the P1 port can supply, I will have to find out empirically whether or not the P1 port and the M5stick are a good match.

#### Charging the M5stick

The M5stick has a 80mAh battery and is configured by default to charge at 100mA.

Even though elsewhere a max current draw of 100mA is mentioned [^1] for DSMR 4.x, and no voltage on pins #1 and #2 for the ISKRA AM550, [^1] also mentions that the ISKRA AM550 uses ESMR5.0, which is supposed to be able to output 250mA of current.
I haven't been able to find a spec for ESMR5.0, but [ref/DSMRv5.0.pdf] is a spec for "DSMR5.0 (Dutch Smart Meter Requirements)" which metions a max output of 250mA. The device itself has a label on it with "SMR 5.0", so I'm hoping ESMRv5.0 and DSMRv5.0 are kinda the same thing and the meter will just supply 250mA.

This might even be enough to not have to worry too much about deep sleep...

## Test setup
Let's see if this configuration will run the M5Stick-C

```
            5V< ───────────┐
            3V3            └──────────── #1 +5V Power supply
            BAT                          #2 Data request
 M5STICK-C  G06                          #3 Data GND         P1
            G36                          #4 -
            G26                          #5 Data line - Open collector
            5V>            ┌──────────── #6 Power GND
            GND ───────────┘
```

In order to test whether the M5stick can operate off of the M5stick, without worrying about reading the telegrams yet, I'll run the following test:

* Attempt to power the M5Stick-C through above cirquit
* Run the M5Stick-C with the following program:
    * On wake up:
        * Connect to WiFi
        * Connect to MQTT
        * publish battery stats to MQTT topic
        * Go to sleep for 10 seconds
* See how long the device lasts. Without power supply, this program can run roughly 3.5 hours.

**17 feb 2021, 11:55**

Just connecting the M5stick to the P1 port using above schema does not show any voltage reported on `v_5vin` or current on `i_5vin`.

:question: Perhaps there's a hardware or configuration adjustment needs to be done before this can be used? 
:question:Or the P1 port might not be able to supply the requested current? 

**17 feb 2021, 17:00**

Apparently I just had +5V and GND mixed up. Sorting this out runs the M5stick just fine.

## Notes

The M5stickC suffers from a bug where uploading times out. This can be remidied by either shorting `G0` to `GND` while programming using a dupont line, or by updating the firmware if you're brave enough [^2].
update: I've updated the firmware, ridding the device of said bug.


## References

[^1] Domoticx Knowledge Center - P1 poort slimme meter (hardware)
[^2] (M5StickC/ATOM on MacOS Catalina can't upload/ ESP32: Timed out waiting for packet header solution (Solved)

[^1]: http://domoticx.com/p1-poort-slimme-meter-hardware/
[^2]: https://forum.m5stack.com/topic/1591/m5stickc-atom-on-macos-catalina-can-t-upload-esp32-timed-out-waiting-for-packet-header-solution-solved