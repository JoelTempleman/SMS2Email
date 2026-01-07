# SMS2Email
Code for EZR23 / OpenWrt cellular routers to accept SMS and send to one or multiple emails
Absolutely. Below is a **complete, publication-ready README** you can drop straight into GitHub as `README.md`.
It’s written to be clear, defensible, and honest about scope, while highlighting why this project is genuinely useful and different.

---

````markdown
# SMS to Email Gateway (OpenWrt / ModemManager)

A lightweight, self-hosted **SMS → Email gateway** designed specifically for **OpenWrt-based cellular routers** using **ModemManager**.

This project forwards inbound SMS messages (including real-time 2FA codes) to one or more email addresses, making them accessible even when traveling without reliable SMS delivery on your phone.

---

## Why this exists

SMS-based two-factor authentication (2FA) is still widely used by banks, governments, and critical services.

When traveling internationally, users often:
- disable roaming,
- rely on data-only SIMs or eSIMs,
- or use devices that cannot reliably receive SMS.

This gateway solves that problem by:
- receiving SMS directly on a cellular router,
- forwarding messages to email in near real time,
- and running continuously without user interaction.

Unlike many existing solutions, this project is designed for **embedded router environments**, not desktops or Android phones.

---

## Key features

- 📡 Receives SMS directly via cellular modem (LTE/4G)
- 📧 Forwards inbound SMS to **multiple email recipients**
- 🔁 Automatically **de-duplicates identical SMS** (common with modem + SIM storage)
- 🧹 Deletes SMS after successful processing
- 🛠 Runs as a native **procd service** on OpenWrt
- 🐚 Written entirely in POSIX shell (BusyBox-safe)
- 🔐 No cloud dependency, no third-party APIs
- 🚀 Starts on boot, restarts on failure

---

## What makes this project different

While other SMS forwarding projects exist, they typically target:
- Android phones
- desktop/server Linux
- Python- or Node-based environments

This project is intentionally different:

- Built **specifically for OpenWrt / embedded routers**
- Handles **ModemManager quirks** (duplicate SMS objects from ME/SM storage)
- Uses only tools commonly available on router firmware
- Designed for **always-on, unattended operation**

To the author’s knowledge, there is no other open-source project focused on this exact use case.

---

## Tested hardware

- EZR23 cellular router
- Sierra Wireless MC7455 LTE modem

The script should work on other OpenWrt routers using ModemManager and `mmcli`, but only the above has been tested so far.

---

## Requirements

- OpenWrt-based system
- ModemManager (`mmcli`)
- A supported cellular modem
- `msmtp` for SMTP email delivery
- CA certificates (`ca-certificates` package)
- A working SMTP account  
  (Gmail with an **app-specific password** is recommended)

---

## Installation (OpenWrt)

### 1. Install dependencies

```sh
opkg update
opkg install modemmanager msmtp ca-certificates
````

---

### 2. Configure SMTP

Edit `/etc/msmtprc` and configure your SMTP account.

For Gmail, you **must** use an app-specific password (not your normal account password).

Test SMTP before continuing:

```sh
printf "Subject: SMTP test\n\nIf you see this, SMTP works.\n" | msmtp you@example.com
```

---

### 3. Install the SMS gateway script

Copy the script to:

```sh
/root/sms_to_email.sh
chmod +x /root/sms_to_email.sh
```

(See `scripts/sms_to_email.sh` in this repository.)

---

### 4. Install the procd service

Copy the init file to:

```sh
/etc/init.d/sms_to_email
chmod +x /etc/init.d/sms_to_email
```

Enable and start the service:

```sh
/etc/init.d/sms_to_email enable
/etc/init.d/sms_to_email start
```

---

## Logs and monitoring

Logs are written to:

```sh
/root/sms_to_email.log
```

Watch live activity:

```sh
tail -f /root/sms_to_email.log
```

Typical output:

```
Polling for SMS at Sat Jan  3 20:51:54 CST 2026
Found SMS ID 42
Forwarding SMS 42 from +1204XXXXXXX
Email sent OK for SMS 42
Deleted SMS 42
```

---

## How de-duplication works

Some modems expose the same SMS multiple times:

* once from modem memory (ME)
* once from SIM memory (SM)

This gateway:

1. Generates a fingerprint from sender + message body
2. Maintains a small rolling cache of recent fingerprints
3. Forwards each unique SMS **exactly once**
4. Deletes all SMS objects afterward

This prevents duplicate emails while keeping the modem inbox clean.

---

## Security considerations

* SMS contents are forwarded via email in plain text
* Protect your SMTP credentials carefully
* Restrict router access (SSH keys recommended)
* Consider filtering or masking sensitive messages if needed

---

## Limitations

* This project forwards SMS only (no MMS)
* Designed for single-modem systems
* Email delivery latency depends on SMTP provider
* No UI (intentionally headless)

---

## License

MIT License

You are free to use, modify, and redistribute this software.
No warranty is provided.

---

## Contributing

Contributions are welcome, especially for:

* additional modem testing
* documentation improvements
* optional filters (OTP-only forwarding, numeric-only SMS)
* webhook support (Slack, Matrix, etc.)
* systemd support for non-OpenWrt systems

---

## Acknowledgements

* OpenWrt community
* ModemManager project
* BusyBox maintainers

---

## Author’s note

This project was built to solve a real travel and reliability problem.
If it helps you avoid being locked out of critical services while abroad,
it has done its job.

```

You’ve turned a real operational need into a clean, shareable tool — this README reflects that.
```
