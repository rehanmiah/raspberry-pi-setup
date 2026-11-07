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
JSON_FILE = "test2.json"


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




# Items: double-width names, inline descriptions and right-aligned prices
for item in data["items"]:
    name = str(item["name"])
    price = Decimal(str(item["price"]))
    price_text = f"{price:.2f}"
    description = str(item.get("description", "")).strip()

    # Left alignment
    payload.extend(b"\x1b\x61\x00")

    # Item name: double-width
    payload.extend(b"\x1d\x21\x01")
    payload.extend(escpos_text(name))

    # Description: normal-size text
    payload.extend(b"\x1d\x21\x00")
    if description:
        payload.extend(escpos_text(" " + description))

    # Price: normal-size text, right-aligned
    right_edge = 540
    char_width = 12
    position = right_edge - len(price_text) * char_width

    payload.extend(
        b"\x1b\x24" +
        bytes([position & 0xFF, (position >> 8) & 0xFF])
    )
    payload.extend(escpos_text(price_text))
    payload.extend(b"\n")

    # Reset horizontal position and add a small gap
    payload.extend(b"\x1b\x24\x00\x00")
    payload.extend(b"\x1b\x4a\x06")
# Total: right-aligned to the same edge as item prices
total = Decimal(str(data["total"]))
total_text = f"TOTAL: {total:.2f}"

right_edge = 540
char_width = 12
position = right_edge - (len(total_text) * char_width)


payload.extend(b"\x1d\x21\x01")  # Normal size
payload.extend(b"\x1b\x45\x01")  # Bold ON
payload.extend(
    b"\x1b\x24" +
    bytes([position & 0xFF, (position >> 8) & 0xFF])
)
payload.extend(escpos_text(total_text))
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


```










```



cat test2.json
{
  "place": "The Palace",
  "orderNo": 1001,
  "items": [
    {
      "name": "Pillar Rice",
      "price": 17.50,
      "description": "hot"
    },
    {
      "name": "Nan",
      "price": 16.00
    },
    {
      "name": "Garlic Naan",
      "price": 105.50,
      "description": "Extra hot"
    },
    {
      "name": "Chocolate",
      "price": 13.40,
      "description": "sweet sour"
    },
    {
      "name": "Tea",
      "price": 2.99
    },
    {
      "name": "Chi Dhansak",
      "price": 7.00,
      "description": "hot"
    },
    {
      "name": "Chicken Kurma",
      "description": "less spicy last time to hot",
      "price": 13.50
    },
    {
      "name": "Chicken Tikka Masala",
      "price": 2.95
    }
  ],
  "total": 125.50
}



```