# raspberry-pi-setup
Raspberry Pi Zero W v1.1 set up

Launch Rasberry pi manager
Select Raspberry Pi Zero option

1. Device: Raspberry Pi Zero
2. OS: Choose Raspberry Pi OS Lite (32-bit) for a headless setup.
Customisation: Set your hostname, username, password, Wi-Fi SSID and password, and Wi-Fi country to GB.
Enable SSH, preferably using password authentication for simplicity.
Write the image to the card.

https://github.com/Prospect-Development-Team/pos80



```


cat print_reciept.py
import json
import subprocess
from decimal import Decimal

PRINTER = "POS80"
JSON_FILE = "test.json"


def escpos_text(text):
    return str(text).encode("cp437", errors="replace")


with open(JSON_FILE, encoding="utf-8") as f:
    data = json.load(f)

payload = bytearray()
payload.extend(b"\x1b@")  # Initialise printer

# Store heading: centred, bold, double-size
payload.extend(b"\x1b\x61\x01")  # Centre
payload.extend(b"\x1b\x45\x01")  # Bold ON
payload.extend(b"\x1d\x21\x33")  # Double width and height
payload.extend(escpos_text(data["place"]))
payload.extend(b"\n")

# Receipt details: normal-size, left-aligned
payload.extend(b"\x1d\x21\x00")
payload.extend(b"\x1b\x45\x00")
payload.extend(b"\x1b\x61\x00")
payload.extend(escpos_text(f'Receipt: {data["orderNo"]}\n'))
payload.extend(b"-" * 32 + b"\n")

# Items: bold, double-width names; normal-size prices
for item in data["items"]:
    name = str(item["name"])
    price = Decimal(str(item["price"]))
    price_text = f"{price:.2f}"

    # Reserve room for the normal-size price and a gap.
    # Double-width names use approximately two columns per character.
    name_width = max(1, (32 - len(price_text) - 1) // 2)
    name_text = name[:name_width]

    # Print item name: bold, double-width, normal-height
    payload.extend(b"\x1b\x45\x01")  # Bold ON
    payload.extend(b"\x1d\x21\x01")  # Double width
    payload.extend(escpos_text(name_text))

    # Print price: normal-size, normal-weight, same line
    payload.extend(b"\x1d\x21\x00")
    payload.extend(b"\x1b\x45\x00")

    spaces = max(1, 32 - (len(name_text) * 2) - len(price_text))
    payload.extend(b" " * spaces)
    payload.extend(escpos_text(price_text + "\n"))

payload.extend(b"-" * 32 + b"\n")

# Total: centred, bold, double-size
total = Decimal(str(data["total"]))
payload.extend(b"\x1b\x61\x01")
payload.extend(b"\x1b\x45\x01")
payload.extend(b"\x1d\x21\x11")
payload.extend(escpos_text(f"TOTAL: {total:.2f}"))
payload.extend(b"\n")

# Restore normal formatting, feed paper, and cut once
payload.extend(b"\x1d\x21\x00")
payload.extend(b"\x1b\x45\x00")
payload.extend(b"\x1b\x61\x00")
payload.extend(b"\n\n\n\n")
payload.extend(b"\x1d\x56\x00")  # Full cut

subprocess.run(
    ["lp", "-d", PRINTER, "-o", "raw"],
    input=bytes(payload),
    check=True,
)

print("Receipt submitted successfully.")
rehan@le-raj:~/pos80 $


```