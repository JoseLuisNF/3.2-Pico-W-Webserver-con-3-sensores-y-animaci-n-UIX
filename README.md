![image](https://user-images.githubusercontent.com/99285798/202308028-56dd1bf5-a8f9-47cb-94fa-1b35a05ef139.png)

INSTITUTO TECNOLÓGICO DE TIJUANA

SUBDIRECCIÓN ACADÉMICA

DEPARTAMENTO DE SISTEMAS Y COMPUTACIÓN

SEMESTRE    Agosto-Diciembre 2022

Ingeniería en sistemas computacionales

----

Sistemas Programables 

Docente: RENE SOLIS REYES

Alumnos: 

Cuevas Martinez Adrian de Jesus 19211623

Robledo Sanchez Damian 19211719

Nunez Felipe Jose Luis 18212233

----
```py
# Codigo el cual utiliza un fotoresistor, laser y buzzer para detectar un bloqueo en la conexion entre el laser
# y el fotoresistor, enviando esta informacion mediante Wi-Fi, dando la habilidad de conectarse a la Pico W
# Codigo adaptado de : https://github.com/pi3g/pico-w

import rp2
import network
import ubinascii
import machine
import urequests as requests
import time
import socket
import gc
import random


LED = machine.Pin(0, machine.Pin.OUT)
SIG = machine.ADC(27)
BUZ = machine.Pin(2, machine.Pin.OUT)
BUTTON = machine.Pin(15, machine.Pin.IN)
LED.high()
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
# If you need to disable powersaving mode
wlan.config(pm = 0xa11140)

# See the MAC address in the wireless chip OTP
mac = ubinascii.hexlify(network.WLAN().config('mac'),':').decode()
print('mac = ' + mac)

# Other things to query
# print(wlan.config('channel'))
# print(wlan.config('essid'))
# print(wlan.config('txpower'))

# Load login data from different file for safety reasons
ssid = "picow"
pw = "tec123tec123"

wlan.connect(ssid, pw)

# Wait for connection with 10 second timeout
timeout = 10
while timeout > 0:
    if wlan.status() < 0 or wlan.status() >= 3:
        break
    timeout -= 1
    print('Waiting for connection...')
    time.sleep(1)
    
# Handle connection error
# Error meanings
# 0  Link Down
# 1  Link Join
# 2  Link NoIp
# 3  Link Up
# -1 Link Fail
# -2 Link NoNet
# -3 Link BadAuth
if wlan.status() != 3:
    raise RuntimeError('Wi-Fi connection failed')
else:
    led = machine.Pin('LED', machine.Pin.OUT)
    for i in range(wlan.status()):
        led.on()
        time.sleep(0.2)
        led.off()
        time.sleep(0.2)
    print('Connected')
    status = wlan.ifconfig()
    print('ip = ' + status[0])
    
# Function to load in html page    
def get_html(html_name):
    with open(html_name, 'r') as file:
        html = file.read()
        
    return html

data = BUZ.value()
dice_val = 'Nothing'

# HTTP server with socket
addr = socket.getaddrinfo('0.0.0.0', 80)[0][-1]

s = socket.socket()
s.bind(addr)
s.listen(1)

print('Listening on', addr)

# Listen for connections
while True:
    try:
        cl, addr = s.accept()
        print('Client connected from', addr)
        cl_file = cl.makefile('rwb', 0)
        while True:
            line = cl_file.readline()
            if not line or line == b'\r\n':
                break
        time.sleep(0.1)
        print("B: ", BUTTON.value())
        if(BUTTON.value()):
            if(SIG.read_u16() > 8000):
                BUZ.high()
                data = True
            else:
                BUZ.low()
                data = False
                print("BUZ: ",BUZ.value())
                print("SIG: ", SIG.read_u16())
        else:
            BUZ.low()
            data = False
        response = get_html('index.html')
        response = response.replace('DETECTED', str(data))
        
        cl.send('HTTP/1.0 200 OK\r\nContent-type: text/html\r\n\r\n')
        cl.send(response)
        cl.close()
    except OSError as e:
        cl.close()
        print('Connection closed')



```

https://www.loom.com/share/92766c7f9ee840d6980f195f2764e10e

https://www.loom.com/share/dadf60a2f90a49ebad2fd3af25394f72

