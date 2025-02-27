# Jetson_bargraph

***Adafruit 0.96' OLED SSD1306 via I2C on a Nvidia Xavier AGX***

Well, this should also work with other Nvidia Jetson devices.

Main driver for this initiative was a performance test of a Nvidia Xavier running CompfyUI. I did struggle with the Python version and could not get ComfyUI working.

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

```Test$>Python3 bargraph.py```

**Please note:** I was running in full power mode ~30 Watt max.
In the program you find some system.wait(0.002) statements.
There is risk, that in low power mode the performance display becomes unstable.
