# Ticket Printer Application

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A web-based ticket submission system that automatically prints tickets on ESC/POS thermal printers connected to a Raspberry Pi. Users can submit tickets through a simple web interface, which are then automatically printed with timestamps and formatted output.

Perfect for:
- Q&A sessions and feedback collection at events
- Anonymous comment boards
- Interactive installations
- Physical message boards

## Description

This application provides a complete solution for collecting and printing tickets in physical form. It consists of:

- **Web Interface**: A clean, responsive frontend that can be hosted on Netlify or run locally
- **Backend API**: A Flask-based API running on Raspberry Pi that handles printer communication
- **Multiple Printer Support**: Works with USB, Serial, Network, and Bluetooth-connected printers
- **Optional Database Logging**: Integrate with Convex to log all submitted tickets
- **Spam Protection**: Built-in reCAPTCHA integration

The system is designed to be flexible and easy to deploy. You can run everything on a single Raspberry Pi, or separate the frontend (hosted on Netlify) from the backend (on your Pi) for remote access.

## Features

- 📝 Simple web interface for ticket submission
- 🖨️ Automatic printing on Netum 58mm thermal printer
- ⏰ Timestamp and date stamping
- 💾 Logged submissions (optional Convex database integration)
- 🌐 Accessible from any device on your network
- 🔒 Built-in reCAPTCHA spam protection

## Quick Start

Want to get started quickly? See [QUICK_START.md](QUICK_START.md) for a step-by-step guide to deploy the application.

For detailed setup instructions, continue reading below.

## Configuration

Before starting, you'll need to configure the following:

### Required Configuration

1. **reCAPTCHA Site Key** - Replace `YOUR_RECAPTCHA_SITE_KEY` in `frontend/index.html` (line 183)
   - Get your key from [Google reCAPTCHA Admin](https://www.google.com/recaptcha/admin)
   - Or set via Netlify environment variable `RECAPTCHA_SITE_KEY`

2. **API URL** - Replace `YOUR_API_URL_HERE` in `frontend/index.html` (line 217)
   - Set this to your ngrok, Cloudflare tunnel, or direct Raspberry Pi URL
   - Example: `https://abc123.ngrok.io` or `https://your-tunnel.trycloudflare.com`

### Optional Configuration

3. **Convex Database** (Optional) - Set `CONVEX_DEPLOYMENT` in Netlify environment variables
   - Only needed if you want to log tickets to a database
   - If not configured, tickets will still print but won't be logged

See `env.example` for all available environment variables.

## Setup Instructions

### 1. Install Dependencies

On your Raspberry Pi 4B, install the required Python packages:

```bash
# Install Python packages
pip3 install -r requirements.txt

# Install system dependencies for USB printer
sudo apt-get update
sudo apt-get install -y python3-pip python3-dev libusb-1.0-0-dev

# For USB printer access (important!)
sudo usermod -a -G lp dialout pi
sudo apt-get install -y printer-driver-all
```

### 2. Connect Your Printer

#### For USB Connection:
1. Plug in your Netum thermal printer via USB
2. Find the USB vendor and product IDs:
   ```bash
   lsusb
   ```
   You should see something like: `Bus 001 Device 003: ID 0416:5011`
3. The first 4 digits (0416) are the vendor ID
4. The last 4 digits (5011) are the product ID
5. Update the values in `app.py` if needed

#### For Serial/GPIO Connection:
If your printer is connected via GPIO serial pins:
```bash
sudo raspi-config
# Navigate to Interface Options > Serial Port > Enable
```

### 3. Configure the Application

Edit the configuration in `app.py` or set environment variables:

```bash
# For USB printer (default)
export PRINTER_TYPE=usb
export USB_VENDOR=0x0416
export USB_PRODUCT=0x5011

# For serial printer
export PRINTER_TYPE=serial
export SERIAL_PORT=/dev/ttyAMA0

# For network printer
export PRINTER_TYPE=network
export NETWORK_HOST=192.168.1.100
```

### 4. Run the Application

```bash
python3 app.py
```

The application will start on `http://0.0.0.0:5000`

### 5. Access from Other Devices

Find your Raspberry Pi's IP address:
```bash
hostname -I
```

Then access the web interface from any device on your network:
```
http://[RASPBERRY_PI_IP]:5000
```

### 6. Run as a Service (Optional)

To run the application automatically on boot:

```bash
# Create a systemd service file
sudo nano /etc/systemd/system/ticket-printer.service
```

Add the following content:

```ini
[Unit]
Description=Ticket Printer Application
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/ticket-printer-app
ExecStart=/usr/bin/python3 /home/pi/ticket-printer-app/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Then enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable ticket-printer
sudo systemctl start ticket-printer
sudo systemctl status ticket-printer
```

## Usage

1. Open a web browser and navigate to `http://[YOUR_PI_IP]:5000`
2. Enter your name (optional)
3. Enter your question or comment
4. Click "Print Ticket"
5. The ticket will automatically print with the specified format

## Ticket Format

The tickets are printed in this format:

```
================================
TICKET
--------------------------------
From: [Name]
Time: [Current Time]
Date: [Current Date]
--------------------------------
Question/Comment
[Your message]
--------------------------------
================================
```

## Troubleshooting

### Printer Not Detected

If the printer is not detected:
1. Check USB connection
2. Verify printer is powered on
3. Run `lsusb` to see if the device is recognized
4. Try different USB ports
5. Check permissions: `sudo chmod 666 /dev/bus/usb/00X/00Y`

### Permission Errors

If you get permission errors:
```bash
sudo usermod -a -G lp,dialout pi
sudo chmod 666 /dev/ttyUSB0  # or /dev/ttyAMA0 for GPIO
# Then log out and log back in
```

### Test Printer Connection

You can test the printer connection:
```bash
python3 << EOF
from escpos.printer import Usb
p = Usb(0x0416, 0x5011)
p.text("Test print\n")
p.cut()
EOF
```

### View Logs

If running as a service:
```bash
sudo journalctl -u ticket-printer -f
```

## API Endpoints

- `GET /` - Web interface
- `POST /submit_ticket` - Submit a ticket to print
  - Body: `{"from_name": "John Doe", "question": "Your question here"}`
- `GET /health` - Health check endpoint

## Requirements

- Python 3.7+
- Netum 58mm ESC/POS thermal printer (https://amzn.to/43Vx9DT)
- Raspberry Pi 4B (https://amzn.to/4ooA5jS)
- 58mm Receipt Paper (https://amzn.to/4ag6q8Y)
- Flask web framework
- python-escpos library

## Deployment Options

This application supports multiple deployment configurations:

### Local Only (Raspberry Pi)
- Run `app.py` directly on the Raspberry Pi
- Access via local network: `http://[PI_IP]:5000`

### Frontend on Netlify + Backend on Raspberry Pi (Recommended)
- Frontend: Deploy `frontend/` folder to Netlify
- Backend: Run `backend/api.py` on Raspberry Pi
- Expose backend via ngrok or Cloudflare Tunnel
- See `README_NETLIFY.md` for detailed instructions

## Environment Variables

Copy `env.example` to `.env` and configure:

```bash
cp env.example .env
# Edit .env with your settings
```

Key variables:
- `PRINTER_TYPE` - Type of printer connection (usb, serial, network, bluetooth)
- `USB_VENDOR` / `USB_PRODUCT` - USB printer IDs (find with `lsusb`)
- `RECAPTCHA_SITE_KEY` - Your Google reCAPTCHA site key
- `CONVEX_DEPLOYMENT` - Convex database URL (optional)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to contribute to this project.

## Code of Conduct

This project adheres to a Code of Conduct that all contributors are expected to follow. Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before participating.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for a detailed history of changes and versions.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Feel free to modify and use as needed!
