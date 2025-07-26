# Raspberry Pi Security Camera Setup Guide

## Prerequisites

### Hardware
- Raspberry Pi 4 (recommended) or Pi 3B+
- Raspberry Pi Camera Module or USB Camera
- MicroSD card (32GB recommended)
- Stable internet connection

### Software
- Raspberry Pi OS (latest version)
- Python 3.7 or higher

## Installation Steps

### 1. Install Required Python Packages

```bash
#Update system packages
sudo apt update && sudo apt upgrade -y

#Install Python packages
pip3 install opencv-python numpy requests picamera2

#Install additional system packages
sudo apt install python3-opencv python3-numpy -y
```

### 2. Enable Camera Interface

```bash
# Enable camera interface
sudo raspi-config
# Navigate to Interface Options > Camera > Enable

# Reboot
sudo reboot
```

### 3. Create Project Directory & Subfolders

```bash
mkdir -p /home/pi/security_camera/detections /home/pi/security_camera/logs
cd /home/pi/security_camera
```

### 4. Create Configuration File (`config.json`)
Create a file named `config.json` in your project directory with the following content:

```json
{
   "monitoring_start": "18:00",
   "monitoring_end": "08:00",
   "alert_cooldown": 300,
   "detection_threshold": 0.5,
   "min_detection_area": 3000,
   "email": {
      "enabled": true,
      "smtp_server": "smtp.gmail.com",
      "smtp_port": 587,
      "sender_email": "your_email@gmail.com",
      "sender_password": "your_app_password",
      "recipient_email": "alert@example.com"
   },
   "pushbullet": {
      "enabled": false,
      "api_key": "your_pushbullet_api_key"
   },
   "telegram": {
      "enabled": false,
      "bot_token": "your_bot_token",
      "chat_id": "your_chat_id"
   }
}
```

## Alert Setup Options

### Option 1: Email Alerts (Recommended)
1. Enable 2-Factor Authentication on your Gmail account.
2. Generate an **App Password** for mail access:
   - Navigate to Google Account > Security > 2-Step Verification > App Passwords.
3. Update the `email` section in `config.json` with your credentials and recipient info.

### Option 2: Pushbullet Notifications
1. Create a Pushbullet account at [pushbullet.com](https://pushbullet.com).
2. Get your API key from Account Settings.
3. Install the Pushbullet app on your phone.
4. Enable and configure Pushbullet in `config.json`.

### Option 3: Telegram Bot
1. Create a Telegram bot via [@BotFather](https://t.me/BotFather).
2. Obtain the Bot token.
3. Get your Chat ID by messaging the bot and checking updates at:
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
4. Enable and configure Telegram in `config.json`.

## Configuration Parameters

| Parameter             | Description                              | Default  |
|-----------------------|------------------------------------------|----------|
| `monitoring_start`    | Start time for monitoring (24h format)    | "18:00"  |
| `monitoring_end`      | End time for monitoring (24h format)      | "08:00"  |
| `alert_cooldown`      | Minimum seconds between alerts             | 300      |
| `detection_threshold` | Confidence threshold for human detection   | 0.5      |
| `min_detection_area`  | Minimum pixel area for valid detection     | 3000     |

## Running the Script

### Manual Start
```bash
cd /home/pi/security_camera
python3 security_camera.py
```

### Auto-Start on Boot Using systemd

1. Create the systemd service file:
```bash
sudo nano /etc/systemd/system/security-camera.service
```

2. Paste the following configuration:
```ini
[Unit]
Description=Security Camera Service
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/security_camera
ExecStart=/usr/bin/python3 /home/pi/security_camera/security_camera.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

3. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable security-camera.service
sudo systemctl start security-camera.service
```

4. Check the status:
```bash
sudo systemctl status security-camera.service
```

## Troubleshooting

### Common Issues
- **Camera not working?**  
  Test with: 
  ```bash
  libcamera-hello --timeout 5000
  ```

- **Permission errors?**  
   Add user to video group:  
   ```bash
   sudo usermod -a -G video pi
   ```

- **Email authentication fails?**  
   - Ensure 2FA is enabled  
   - Use generated App Password, not the regular password

- **View logs**:  
   Real-time:  
   ```bash
   sudo journalctl -u security-camera.service -f
   ```
   Application logs:  
   ```bash
   tail -f /home/pi/security_camera.log
   ```

## Performance Optimization

- Reduce camera resolution in the script for faster processing (e.g., 320x240).
- Increase sleep time or process every nth frame.
- Use OpenCV GPU support:  
   ```bash
   pip3 install opencv-contrib-python
   ```

## Security Considerations

- Change default passwords.
- Use strong email app passwords.
- Secure your network.
- Keep system updated:
   ```bash
   sudo apt update && sudo apt upgrade
   ```
- Configure firewall:
   ```bash
   sudo ufw enable
   sudo ufw allow ssh
   ```

## Monitoring and Maintenance

- Regularly check disk space (detection images can grow).
- Monitor logs for errors.
- Test alerts periodically.
- Backup your configuration files regularly.

## File Structure

   ```bash
   /home/pi/security_camera/
   ├── security_camera.py # Main Python script
   ├── config.json # Configuration file
   ├── security_camera.log # Application logs
   └── detections/ # Folder for detection images
      ├── detection_YYYYMMDD_HHMMSS.jpg
      └── ...
   ```