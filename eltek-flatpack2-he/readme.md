

## send 'login' to flatpack
cansend can0 05004804#1424710928870000

## 16 amps, 57.6 volts, 59.5 ovp
A000801680163E17

## 16 amps, 47.6v, 49.0v ovp
cansend can0 05FF4004#A000981298122413

## set permanent voltage 47.6v
cansend can0 05009C00#2915009812

## set permanent voltage 47.0v
cansend can0 05009C00#2915005C12

## protocol
https://github.com/the6p4c/Flatpack2/blob/master/Protocol.md

## how to read & decode can bus messages from flatpack
https://github.com/the6p4c/Flatpack2/blob/master/Arduino/fp2_control/fp2_control.ino

## main knowledge goes here
https://openinverter.org/forum/viewtopic.php?t=1351

## can bus in linux
https://wiki.rdu.im/_pages/Application-Notes/Software/can-bus-in-linux.html

## candleLight setup
https://pengutronix.de/en/blog/2022-02-17-first-steps-using-the-candlelight.html

## candle aliexpress link
https://www.aliexpress.com/item/1005005771323549.html



```python
#!/usr/bin/python
import os
import time
running = True

def follow(name):
    global running
    seek = True

    while running:
        f = open(name, "r")
        f_ino = os.stat(f.fileno()).st_ino

        if seek:
          f.seek(0,2)

        while running:
            try:
                line = f.readline()
            except UnicodeDecodeError:
                continue

            if not line:
                time.sleep(0.1)
                if f_ino != os.stat(name).st_ino:
                    seek = False
                    break
                continue
            yield line

        f.close()

filename = 'candump.txt'


def parse_line(line):
  if '05014004' in line or '0501400C' in line:
      b = bytearray()
      for val in line.split()[6:]:
        b.append(int(val, 16))
      print("\n\n")
      print("intake temp", b[0])
      print("current", (b[1] | (b[2] << 8))/10)
      print("output voltage", 0.01 * (b[3] | (b[4] << 8)))
      print("input voltage", (b[5] | (b[6] << 8)) + 6 )
      print("out temp", b[7])

for line in follow(filename):
    parse_line(line)
```
