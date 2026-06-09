# os-proxyman

**OPNsense Proxy Manager** — an Nginx Proxy Manager-style reverse proxy GUI,
built as a native OPNsense plugin powered by Caddy.

Manage all your reverse proxy hosts, TLS certificates, and access control
rules from inside the OPNsense web UI — no SSH, no config files, no fuss.

---

## Features

| Feature | Status |
|---|---|
| Proxy host management (add / edit / toggle / delete) | ✅ v1.0 |
| Automatic TLS via Let's Encrypt & ZeroSSL | ✅ v1.0 |
| Internal self-signed certificates | ✅ v1.0 |
| Custom OPNsense-managed certificates | ✅ v1.0 |
| IP-based access control lists (CIDR allow/deny) | ✅ v1.0 |
| WebSocket proxying | ✅ v1.0 |
| Custom upstream request headers | ✅ v1.0 |
| Upstream health checks | ✅ v1.0 |
| Live Caddy status indicator | ✅ v1.0 |
| DNS-01 ACME challenge (wildcard certs) | 🗓 v1.1 |
| HTTP Basic Auth per host | 🗓 v1.1 |
| Multiple upstreams / load balancing | 🗓 v1.2 |
| pfSense CE / pfSense Plus port | 🗓 v1.3 |
| Redirect-only hosts | 🗓 v1.2 |

---

## Requirements

- OPNsense 23.7 or later
- `os-caddy` plugin installed (provides the Caddy binary)
- Ports 80 and 443 not occupied by OPNsense's built-in web GUI
  (move the GUI to a non-standard port under System → Settings → Administration)

---

## Installation

### From OPNsense plugin browser (once listed)

System → Firmware → Plugins → search **os-proxyman** → Install

### Manual install (development / testing)

```bash
# On your OPNsense box
fetch https://github.com/YOUR_USERNAME/os-proxyman/releases/latest/download/os-proxyman.txz
pkg add os-proxyman.txz
service configd restart
```

After install, navigate to **Services → Proxy Manager** in the OPNsense menu.

---

## First-time setup

1. Go to **Services → Proxy Manager → Settings**
2. Enter your ACME registration email (required for Let's Encrypt)
3. Choose your ACME CA (production or staging for testing)
4. Save settings
5. Move the OPNsense GUI to a port other than 80/443 if you haven't already
   (System → Settings → Administration → TCP Port)
6. Add your first proxy host

---

## Adding a proxy host

1. Click **Add Host**
2. Enter the domain name (e.g. `plex.example.com`)
3. Enter the upstream IP/hostname and port (e.g. `192.168.1.20`, port `32400`)
4. Choose TLS mode:
   - **ACME** — Caddy automatically obtains and renews a certificate
   - **Internal** — self-signed cert, trusted by OPNsense's CA
   - **Custom** — choose a cert from OPNsense certificate manager
   - **Off** — plain HTTP only
5. Click Save, then **Apply Changes**

---

## Contributing

Pull requests are welcome. This plugin is designed to grow with the community.

### Development setup

```bash
git clone https://github.com/YOUR_USERNAME/os-proxyman
cd os-proxyman

# Sync to an OPNsense dev box (adjust IP)
rsync -avz src/ root@192.168.1.1:/usr/local/opnsense/
ssh root@192.168.1.1 "service configd restart"
```

### Project structure

```
src/opnsense/
├── mvc/app/
│   ├── controllers/OPNsense/ProxyManager/
│   │   ├── IndexController.php          # UI page controller
│   │   └── Api/SettingsController.php   # REST API
│   ├── models/OPNsense/ProxyManager/
│   │   ├── ProxyManager.xml             # Data schema
│   │   └── ProxyManager.php             # Model class
│   └── views/OPNsense/ProxyManager/
│       ├── index.volt                   # Main UI template
│       ├── menu.xml                     # Sidebar navigation entry
│       └── ACL.xml                      # Permission definitions
├── scripts/proxyman/
│   ├── generate.py                      # Caddyfile renderer
│   └── status.py                        # Caddy health checker
├── service/conf/actions.d/
│   └── actions_proxyman.conf            # configd action definitions
└── rc.d/caddy                           # FreeBSD init script
```

### Running CI locally

```bash
# Python
python3 -m py_compile src/opnsense/scripts/proxyman/*.py

# PHP
find src -name "*.php" -exec php -l {} \;

# XML
python3 -c "
import xml.etree.ElementTree as ET, glob
for f in glob.glob('src/**/*.xml', recursive=True):
    ET.parse(f)
    print('OK:', f)
"
```

---

## Port 80 / 443 conflict FAQ

**Q: My OPNsense GUI is on 443 — will this break it?**

A: Yes, you need to move the GUI first.
System → Settings → Administration → TCP Port → set to e.g. `8443`.
Caddy then owns 80 and 443.

**Q: Can Caddy and the GUI coexist on 443?**

A: Not on the same IP. If you have multiple WAN IPs, you can bind each to
a different listener. Most home setups only have one, so move the GUI port.

---

## License

MIT — use it, fork it, sell it, do whatever you want with it.
If you build something cool, open a PR or drop a star.

---

## Acknowledgements

Built on the shoulders of:
- [Caddy](https://caddyserver.com/) — the reverse proxy engine
- [OPNsense](https://opnsense.org/) — the platform and MVC framework
- [os-caddy](https://github.com/opnsense/plugins) — the Caddy package that makes this possible
- [Nginx Proxy Manager](https://nginxproxymanager.com/) — the UX inspiration
