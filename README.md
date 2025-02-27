# Jetson_bargraph

***Adafruit 0.96' OLED SSD1306 via I2C on a Nvidia Xavier AGX***

Well, this should also work with other Nvidia Jetson devices.

Main driver for this initiative was a performance test of a Nvidia Xavier running CompfyUI. I did struggle with the Python version and could not get ComfyUI working on this wonderful **Nvidia 512 GPU cores beast**.

Finally I ended up with Stable Diffusion and now it became interesting, what is the Xavier doing right now? I didn't attach a monitor and was working with SHH. Sure there is a JTOP installed, as I am doing (almost) everything what Jim suggests!
However, the I2C bus could have been used for the BOSCH BME Sensor, so why not quickly adopting the Adafruit Lib for the super nice 0.96 OLED display?

Well, it is a Nvidia Tegra processor, thus trouble with the busio driver.
Finally I found a detour via ***smbus2*** and now I could permanently monitor CPU and GPU performance.

<img width="1025" alt="image" src="https://github.com/user-attachments/assets/b736886c-76a0-447f-b40b-1ef2ab7dc13f" />

The leftmost bar is GPU, remaining bars to the right are 8 times CPU.

About the super easy wiring, Jim from **[Jetsonhacks.com](https://jetsonhacks.com/2018/10/23/i2c-nvidia-jetson-agx-xavier-developer-kit/)** told us:

| Function | Value |
|----------|----------|
| SCL   | 3   |
| SDA   | 5   |
| Vcc   | 1   |
| GND   | 6   |

I2C Bus #8


***Running the GPU/CPU Performance Display Python program***

```Test$>python3 bargraph.py```

**Please note:** 

[1] I was running in full power mode with all 6 CPU cores ~30 Watt max.
In the Python program you find some ```time.sleep(0.003)``` statements.
There is risk, that in low power mode the performance display becomes unstable as timing doesn't match anymore.

[2] I could just run in (slow) command write mode. Data write mode caused I2C bus crashes with necessary power cycling. This could be due to a cheap OLED display or lousy smbus2 implementation. - I got stuck here for several hours.
