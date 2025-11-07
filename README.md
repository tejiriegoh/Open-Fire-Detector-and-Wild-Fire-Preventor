# Open-Fire-Detector-and-Wild-Fire-Preventor
# By using a Raspberry Pi kit and Thonny I programmed a prototype with sensors to detect when an open fire may progress to a more hazardous wild fire

from machine import ADC, Pin, PWM
import time
import dht
from lcd1602 import LCD
import uasyncio as asyncio

# MQ135 Sensor
mq135 = ADC(Pin(26))
mq135_value = mq135.read_u16()
vout = (mq135_value/66535)*3.3
Rs = 10000 * (3.3 - vout)/vout
ppm = 470 *((Rs/20000)**-0.42)
percent = ppm/10000
           
           
           
           
buzzer = PWM(Pin(17))
buzzer.duty_u16(0)
buzzer.freq(1000)

# DHT11 sensor
sensor = dht.DHT11(Pin(16))

# LCD
lcd = LCD()
lcd.clear()

# LEDs
led_pins = []
for x in range(6, 16):
    led = PWM(Pin(x))
    led.freq(1000)
    led_pins.append(led)

# Globals
hum = 0.0
temp = 0.0

def set_humidity():
    global hum
    hum = sensor.humidity
    hum = max(75, min(sensor.humidity, 85))
    return hum

def set_temperature():
    global temp
    temp = sensor.temperature
    temp = max(25, min(temp, 35))
    return temp

# Async scrolling text
async def scrolling_text(text, delay=0.3):
    start = 0
    stop = 16
    while stop <= len(text):
        lcd.write(0, 0, text[start:stop])
        await asyncio.sleep(delay)
        start += 1
        stop += 1

# Buzzer task
async def beep():
    while True:
        sensor.measure()
        temp = sensor.temperature
        if percent >= 0.2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                :
            buzzer.duty_u16(50000)
            await asyncio.sleep(0.1)
            buzzer.duty_u16(0)
            await asyncio.sleep(0.1)
        else:
            buzzer.duty_u16(0)
            await asyncio.sleep(0.5)

# Main task: LEDs, LCD, sensor
async def otherstuff():
    while True:
        sensor.measure()
        temperature = set_temperature()
        humidity = set_humidity()

        # LED brightness based on temp and humidity
   
               
        now = time.ticks_ms()
        sensor.measure()
        temps = (int(set_temperature()))
        temps1 = int(((temps))*6553.4-163834)
        hum = round(set_humidity()*0.9 -66.5)
       

        for x in range(10):
            if x < hum:
                led_pins[x].duty_u16(int(0))
            else:
                led_pins[x].duty_u16(temps1)

        # Display scrolling text
        lcd.clear()
        message = f"                -Temp is {temperature}C and humidity is {humidity}%   "
        await scrolling_text(message)
        lcd.clear()

        # Debug output
        print("temp:", temperature, "C")
        print("hum:", humidity, "%")
        await asyncio.sleep(0.1)
        print(mq135_value)

# Entry point
async def main():
    await asyncio.gather(otherstuff(), beep())

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
