/*
Loetluemmel Pit Förster (c) & (r) 2025


Python code for a SSD1306 OLED Display connected to a Nvidia Xavier AGX

SDA pin5
SCL pin7
PWR
GND

The Display shows 9 bar graphs proportional to the GPU and 8 x CPU performannce 

As this is a Nvidia Tegra specific implementation, the Adafruit driver based on busio is
not supported. That's why I did implement the bargraph display manually with smbus2 
*/


import time
import subprocess
import re
import smbus2
import numpy as np

# **I2C Setup**
bus_number = 8
address = 0x3C
bus = smbus2.SMBus(bus_number)

# **Display Constants**
OLED_WIDTH = 128
OLED_HEIGHT = 64
GPU_BAR_WIDTH = 10  # GPU bar width
CPU_BAR_WIDTH = 5   # CPU bar width
GPU_BAR_SPACING = 5  # Dedicated space after the GPU bar
BAR_SPACING = 4
NUM_BARS = 9
MAX_BAR_HEIGHT = 64

# **Framebuffer (Ensure It’s Defined)**
oled_buffer = np.zeros((8, 128), dtype=np.uint8)

# **TegraStats Setup**
def start_tegrastats():
    return subprocess.Popen(["tegrastats"], stdout=subprocess.PIPE, stderr=subprocess.DEVNULL, text=True)

def get_performance_data(tegrastats_process):
    try:
        line = tegrastats_process.stdout.readline().strip()

        gpu_match = re.search(r"GR3D_FREQ (\d+)%", line)
        gpu_load = int(gpu_match.group(1)) if gpu_match else 0

        cpu_match = re.findall(r"CPU \[([^\]]+)\]", line)
        cpu_loads = [int(m.split('%')[0]) for m in cpu_match[0].split(',')] if cpu_match else [0] * 8

        return gpu_load, cpu_loads
    except Exception as e:
        print(f"Error parsing tegrastats: {e}")
        return 0, [0] * 8

# **OLED Functions**
def send_command(cmd):
    """Send an I2C command to the OLED with retry logic."""
    for attempt in range(3):
        try:
            time.sleep(0.002)
            bus.write_byte_data(address, 0x00, cmd)
            time.sleep(0.003)
            return
        except OSError as e:
            print(f"⚠ I2C Error sending command 0x{cmd:02X} on attempt {attempt+1}: {e}")
            time.sleep(0.01)  # Wait before retrying

def send_data_chunk(data):
    """Send a chunk of data (multiple bytes) to the OLED one byte at a time."""
    for byte in data:
        try:
            time.sleep(0.002)
            bus.write_byte_data(address, 0x40, byte)  # Send one byte at a time
            time.sleep(0.003)  # Small delay between each byte to avoid overloading the display
        except OSError as e:
            print(f"Error sending data byte: {e}")
            time.sleep(0.01)  # Retry with a delay if an error occurs

def reset_oled():
    """Force an OLED reset after an I2C bus reset."""
    print("🔄 Resetting OLED...")
    send_command(0xAE)  # Turn display off
    time.sleep(0.1)
    send_command(0xE3)  # NOP - wakes up some controllers
    time.sleep(0.1)
    send_command(0xAF)  # Turn display on
    time.sleep(0.1)

def configure_display():
    """Initialize OLED with stability improvements."""
    reset_oled()  # Ensure display is reset

    commands = [
        0x20, 0x00, 0xA8, 0x3F, 0xD3, 0x00,
        0x40, 0xA1, 0xC8, 0xDA, 0x12, 0x81, 0x7F,
        0xD9, 0xF1, 0xDB, 0x40, 0xA4, 0xA6, 0x8D, 0x14
    ]
    for cmd in commands:
        send_command(cmd)

def draw_bargraph(gpu_load, cpu_loads):
    """Draws bar graphs efficiently by updating only changed areas."""
    global oled_buffer  # Ensure we use the global framebuffer
    new_buffer = np.zeros((8, 128), dtype=np.uint8)

    def set_pixel(x, y):
        page = y // 8
        bit_position = y % 8
        new_buffer[page][x] |= (1 << bit_position)

    # The GPU bar uses 10 pixels, the CPU bars use 5 pixels
    bar_widths = [GPU_BAR_WIDTH] + [CPU_BAR_WIDTH] * len(cpu_loads)
    loads = [gpu_load] + cpu_loads

    # Calculate the remaining width after GPU bar + GPU bar spacing
    remaining_width_for_cpu_bars = OLED_WIDTH - (GPU_BAR_WIDTH + GPU_BAR_SPACING)  # GPU bar width + space after it
    num_cpu_bars = len(cpu_loads)
    total_cpu_width = num_cpu_bars * CPU_BAR_WIDTH
    space_left_for_cpu_bars = remaining_width_for_cpu_bars - total_cpu_width

    # Distribute the remaining space evenly between the CPU bars
    extra_space_per_bar = space_left_for_cpu_bars // (num_cpu_bars - 1) if num_cpu_bars > 1 else 0
    cpu_starting_pos = GPU_BAR_WIDTH + GPU_BAR_SPACING  # Start CPU bars after GPU bar and its space

    # Draw bars
    for i, (load, bar_width) in enumerate(zip(loads, bar_widths)):
        if i == 0:  # GPU bar
            x_pos = 0
        else:  # CPU bars
            x_pos = cpu_starting_pos
            if i > 1:  # Add extra spacing for CPU bars, except the first one
                x_pos += (i - 1) * (CPU_BAR_WIDTH + extra_space_per_bar)

        bar_height = int((load / 100) * MAX_BAR_HEIGHT)

        for y in range(MAX_BAR_HEIGHT - bar_height, MAX_BAR_HEIGHT):
            for w in range(bar_width):
                set_pixel(x_pos + w, y)

    # Only update the display if something has changed
    for page in range(8):
        if not np.array_equal(oled_buffer[page], new_buffer[page]):
            oled_buffer[page] = new_buffer[page]

            send_command(0xB0 + page)
            send_command(0x21)
            send_command(0x00)
            send_command(0x7F)
            
            # Send all data for this page at once
            send_data_chunk(oled_buffer[page])

def clear_display():
    """Clears the OLED screen pixel by pixel to avoid I2C overload."""
    for page in range(8):  # 8 pages (each 8 pixels tall)
        send_command(0xB0 + page)  # Set page address
        send_command(0x21)  # Set column range
        send_command(0x00)  # Start column
        send_command(0x7F)  # End column

        # Send empty data in one chunk
        send_data_chunk([0x00] * 128)

    time.sleep(0.02)  # Small delay before turning ON





# **Run the Sequence**
configure_display()
clear_display()
tegrastats_process = start_tegrastats()

try:
    while True:
        gpu_load, cpu_loads = get_performance_data(tegrastats_process)
        draw_bargraph(gpu_load, cpu_loads) 
        # time.sleep(1.5)
except KeyboardInterrupt:
    print("\n🔴 Stopping...")
    tegrastats_process.terminate()
