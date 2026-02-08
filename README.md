# Alternative WhatsApp Gateway Plugin for NetBillX

A powerful WhatsApp messaging integration plugin for NetBillX that enables automated customer notifications via WhatsApp. This plugin connects to the **go-whatsapp-web-multidevice** server, providing a self-hosted, cost-effective alternative to paid WhatsApp API services.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Version](https://img.shields.io/badge/version-1.0.0-green.svg)
![PHP](https://img.shields.io/badge/PHP-7.4%2B-purple.svg)

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Plugin Installation](#plugin-installation)
  - [WhatsApp Server Installation](#whatsapp-server-installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Linking WhatsApp Device](#linking-whatsapp-device)
  - [Device Management](#device-management)
  - [Sending Messages](#sending-messages)
- [API Reference](#api-reference)
- [Cron Job](#cron-job)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

## Features

### Device Linking
- **QR Code Login** - Scan QR code from WhatsApp mobile app
- **Pairing Code Login** - Link using phone number and 8-digit code
- **Auto-Detection** - Automatically detects when device connects
- **Auto-Save** - Device ID automatically saved to configuration

### Device Management
- View all connected WhatsApp devices
- Manual reconnect for individual devices
- Logout devices directly from dashboard
- Real-time connection status monitoring
- Refresh device list on demand

### Message Sending
- Send WhatsApp messages via REST API
- Compatible with NetBillX SMS notification system
- Support for customer notifications (invoices, reminders, etc.)
- Secure API with secret key authentication

### Auto-Reconnect
- Built-in cron job for session maintenance
- Automatic reconnection to keep WhatsApp session alive
- Works in both CLI and web environments
- Detailed logging for troubleshooting

### Security
- API key authentication
- Secure credential storage (masked inputs)
- Proxied QR code images (no direct API exposure)
- Admin-only access controls

## Requirements

| Requirement | Version |
|-------------|---------|
| NetBillX | Latest |
| PHP | 7.4+ |
| PHP cURL Extension | Required |
| go-whatsapp-web-multidevice | Latest |
| Docker (optional) | 20.0+ |

## Installation

### Plugin Installation

1. **Download the plugin files**

2. **Upload to NetBillX**
   ```
   /system/plugin/wga.php
   /system/plugin/ui/wga.tpl
   ```

3. **Access the plugin**
   - Navigate to: `Settings > Alt WA Gateway`

### WhatsApp Server Installation

Choose one of the following methods:

#### Method 1: Docker (Recommended)

```bash
docker run -d \
  --name whatsapp-server \
  --restart unless-stopped \
  -p 3000:3000 \
  -v $(pwd)/whatsapp-data:/app/storages \
  aldinokemal/go-whatsapp-web-multidevice:latest
```

#### Method 2: Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  whatsapp:
    image: aldinokemal/go-whatsapp-web-multidevice:latest
    container_name: whatsapp-server
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ./whatsapp-data:/app/storages
    environment:
      - APP_DEBUG=false
      - APP_PORT=3000
```

Then run:
```bash
docker-compose up -d
```

#### Method 3: Manual Installation (Go)

```bash
# Install Go 1.21+ first, then:
git clone https://github.com/aldinokemal/go-whatsapp-web-multidevice.git
cd go-whatsapp-web-multidevice
go build -o whatsapp-server
./whatsapp-server
```

### Environment Variables (Optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | 3000 | Server port |
| `APP_DEBUG` | false | Enable debug mode |
| `APP_WEBHOOK` | - | Webhook URL for incoming messages |
| `APP_ACCOUNT_VALIDATION` | true | Validate phone numbers |

## Configuration

1. **Navigate to Plugin Settings**
   - Go to: `Settings > Alt WA Gateway`

2. **Enter Server URL**
   - Example: `http://your-server-ip:3000`
   - For local: `http://localhost:3000`

3. **Save Changes**

4. **Link WhatsApp Device**
   - Click "Login with QR Code" or "Login with Code"
   - Follow the on-screen instructions

5. **Configure NetBillX SMS Settings**
   - Copy the API URL from the plugin page
   - Paste it in `Settings > SMS/WhatsApp Settings > Server URL`

## Usage

### Linking WhatsApp Device

#### Option 1: QR Code
1. Click **"Login with QR Code"**
2. Open WhatsApp on your phone
3. Go to **Settings > Linked Devices**
4. Tap **"Link a Device"**
5. Scan the QR code displayed
6. Wait for "Device Connected Successfully!" message

#### Option 2: Pairing Code
1. Click **"Login with Code"**
2. Enter your WhatsApp phone number (with country code, without +)
3. Click **"Get Pairing Code"**
4. Open WhatsApp on your phone
5. Go to **Settings > Linked Devices**
6. Tap **"Link a Device"**
7. Tap **"Link with phone number instead"**
8. Enter the 8-digit pairing code displayed
9. Wait for "Device Connected Successfully!" message

### Device Management

| Action | Description |
|--------|-------------|
| **Refresh List** | Reload connected devices from server |
| **Reconnect** | Manually reconnect a specific device |
| **Logout** | Disconnect device (requires re-linking) |

### Sending Messages

Messages are sent automatically through NetBillX's notification system once configured.

## API Reference

### Send Message

Send a WhatsApp message directly via API.

**Endpoint:**
```
GET /plugin/wga_sendMessage
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `phone` | string | Yes | Recipient phone number (with country code) |
| `message` | string | Yes | Message text to send |
| `secret` | string | Yes | MD5 hash of your API key |

**Example:**
```
https://yourdomain.com/index.php?_route=plugin/wga_sendMessage&phone=2348012345678&message=Hello%20World&secret=your_md5_secret
```

**Response (Success):**
```json
{
  "success": true,
  "message": "Message sent successfully"
}
```

**Response (Error):**
```json
{
  "success": false,
  "message": "Error description"
}
```

### Internal API Endpoints

These endpoints are used internally by the plugin:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `wga_getDevices` | GET | Get list of connected devices |
| `wga_getQrCode` | GET | Get QR code for device linking |
| `wga_getPairingCode` | POST | Get pairing code for device linking |
| `wga_reconnect` | POST | Reconnect a device |
| `wga_logoutDevice` | POST | Logout a device |
| `wga_saveDeviceId` | POST | Save device ID to config |

## Cron Job

The plugin registers a cron hook that automatically reconnects the WhatsApp session to keep it alive.

**How it works:**
- Runs on every NetBillX cron cycle
- Only executes if Server URL and Device ID are configured
- Sends reconnect request to maintain session
- Logs success/failure to system logs

**CLI Output:**
```
WGA Cron: Reconnect successful for device abc123
```

**Ensure NetBillX cron is running:**
```bash
* * * * * php /path/to/NetBillX/cron.php
```

## Troubleshooting

### Common Issues

#### "$ is not defined" Error
- **Cause:** jQuery not loaded
- **Solution:** Ensure `admin/footer.tpl` is included before scripts

#### QR Code Shows 403 Forbidden
- **Cause:** Direct browser access to API server blocked
- **Solution:** Plugin proxies QR image through PHP (automatic)

#### "Server URL not configured"
- **Cause:** Server URL not saved in settings
- **Solution:** Enter and save the WhatsApp server URL

#### "Device ID not configured"
- **Cause:** No device linked yet
- **Solution:** Link a device using QR Code or Pairing Code

#### QR Code Not Loading
- **Cause:** WhatsApp server not running or unreachable
- **Solution:**
  - Check if server is running: `docker ps`
  - Check server URL is correct
  - Check firewall allows port 3000

#### Pairing Code Not Showing
- **Cause:** API response format different
- **Solution:** Check browser console for "Pairing code response:" log

### Checking Logs

**NetBillX Logs:**
- Check `system/logs/` directory
- Look for entries starting with `WGA`

**Docker Logs:**
```bash
docker logs whatsapp-server
```

### Testing Connection

1. Ensure WhatsApp server is running
2. Test API directly:
   ```bash
   curl http://your-server:3000/app/devices
   ```
3. Check response in browser console (F12)

## Security Recommendations

1. **Use HTTPS** - Set up SSL/TLS for production
2. **Firewall** - Restrict port 3000 to your server only
3. **Reverse Proxy** - Use nginx/apache as reverse proxy
4. **API Key** - Keep your API key secure
5. **Regular Updates** - Keep both plugin and server updated

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Author

**Focuslinks Digital Solutions**

- Website: [https://focuslinkstech.com.ng/](https://focuslinkstech.com.ng/)
- GitHub: [https://github.com/Focuslinkstech/](https://github.com/Focuslinks-Tech/)
- Telegram: [https://t.me/focuslinkstech](https://t.me/focuslinkstech)
- Email: focuslinkstech@gmail.com

---

## Support

If you find this plugin helpful, consider:

- Starring the repository
- Reporting bugs
- Suggesting new features
- Improving documentation

For support, please contact via [Telegram](https://t.me/focuslinkstech) or open an issue on GitHub.

---

**Disclaimer:** This plugin is not affiliated with WhatsApp Inc. Use responsibly and in accordance with WhatsApp's Terms of Service.
