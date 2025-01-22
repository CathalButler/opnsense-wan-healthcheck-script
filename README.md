# WAN Health Check Script for OPNsense

This script checks the status of your WAN connection and sends a ping to [Healthchecks.io](https://healthchecks.io) every 5 minutes. If the WAN is down, it logs the event without notifying Healthchecks.io. If Healthcheck doesn't receive a ping after 5mins, it can be configured to send an alert using their integrations.

---

## Requirements
1. **Access to OPNsense:** Ensure you have admin access to your OPNsense system.
2. **Healthchecks.io URL:** Sign up at [Healthchecks.io](https://healthchecks.io) and create a new check. Use the generated URL in the script.

---

## Script
Save the following script as `check_wan_health.sh` on your OPNsense system:

```bash
#!/bin/sh

# Configuration
HEALTHCHECKS_URL="https://hc-ping.com/your-unique-id"  # Replace with your Healthchecks.io URL
PING_TARGET="8.8.8.8"  # Public IP to verify WAN connectivity (e.g., Google DNS)

# Check if the WAN is up
if ping -c 1 -W 2 "$PING_TARGET" > /dev/null 2>&1; then
    # If the WAN is up, ping Healthchecks.io
    curl -fsS --retry 3 "$HEALTHCHECKS_URL" > /dev/null 2>&1
    echo "$(date): WAN is up, Healthchecks.io notified." >> /var/log/check_wan_health.log
else
    # If the WAN is down, log the event
    echo "$(date): WAN is down, Healthchecks.io not notified." >> /var/log/check_wan_health.log
fi
```

---

## Installation

### 1. **Create the Script**
1. Log in to your OPNsense system.
2. Navigate to `/usr/local/bin` or another suitable directory.
3. Create the script file:
   ```bash
   sudo vi /usr/local/bin/check_wan_health.sh
   ```
4. Paste the script content and save it.

### 2. **Make the Script Executable**
```bash
sudo chmod +x /usr/local/bin/check_wan_health.sh
```

---

## Add the Script to Configd
To integrate the script with OPNsense’s configd system:

### 1. **Create a Configd Action File**
1. Navigate to `/usr/local/opnsense/service/conf/actions.d/`:
   ```bash
   sudo vi /usr/local/opnsense/service/conf/actions.d/actions_check_wan_health.conf
   ```
2. Add the following content:
   ```ini
   [restart]
   command:/usr/local/bin/check_wan_health.sh
   parameters:
   type:script
   message:Checking WAN health and notifying Healthchecks.io
   description:Check WAN health and send ping to Healthchecks.io
   ```

### 2. **Reload Configd**
After creating the file, reload configd to register the new action:
```bash
sudo service configd restart
```

---

## Schedule the Script with Cron
1. Log in to the OPNsense web interface.
2. Navigate to **System > Settings > Cron**.
3. Add a new cron job with the following settings:
   - **Command:** Select `Check WAN health and send ping to Healthchecks.io` from the dropdown.
   - **Parameters** `restart`
   - **Interval:** Every 5 minutes or whatever fits.
4. Apply the settings.

---

## Configuration
1. Replace `HEALTHCHECKS_URL` in the script with the URL provided by Healthchecks.io.
2. (Optional) Update `PING_TARGET` if you want to use a different target for WAN connectivity checks.

---

## Logs
The script writes logs to `/var/log/check_wan_health.log`. You can view these logs to troubleshoot or confirm script functionality:
```bash
cat /var/log/check_wan_health.log
```

---

## Notes
- The script retries the Healthchecks.io ping up to 3 times to ensure reliability.
- Adjust the `PING_TARGET` if your want to use a different IP or domain.

---
