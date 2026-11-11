# Configuring CoreDNS

A guide to standing up a small local DNS server with **CoreDNS** for a home
LAN, including resolving static host names, forwarding everything else,
logging, autostart on boot, and making bare short names (`barn`, `home`)
resolve without depending on each client's resolver configuration.

> **Network context for this example**
>
> | Item | Value |
> |---|---|
> | LAN subnet | `192.168.0.0/24` |
> | CoreDNS server IP | `192.168.0.127` |
> | Domain (zone) | `mylan` |
> | Static host 1 | `192.168.0.250` → `home` |
> | Static host 2 | `192.168.0.45` → `barn` |
> | Upstream forwarders | `192.168.1.1` (LAN gateway), `8.8.8.8` (public) |

---

## 1. Goals

1. Resolve `home` / `home.mylan.` → `192.168.0.250` and `barn` / `barn.mylan.` → `192.168.0.45`.
2. Resolve **bare short names** (`barn`, `home`) without the client needing a `search` domain.
3. **Fall through** for every other name and forward it to `192.168.1.1`, then `8.8.8.8`.
4. **Log** what CoreDNS does (queries, client, responses).
5. **Start automatically** on every boot.

---

## 2. The Corefile

Drop this in `/etc/coredns/Corefile`.

```corefile
# CoreDNS Corefile
#   Zone:        mylan
#   Subnet:      192.168.0.0/24
#   CoreDNS IP:  192.168.0.127

# Note: "." is included in the zone list so that bare short names
# (e.g., "barn") are answered directly from the hosts file aliases,
# instead of being forwarded upstream. The short-name column in the
# hosts file (e.g., "192.168.0.45  barn.mylan.  barn") is what makes
# this work — keep both columns when you edit it.
mylan. . {
    # --- Logging -------------------------------------------------------
    # Log every query CoreDNS receives and its response.
    log

    # Augment each log line with the client source address.
    whoami

    # --- Diagnostics / ops --------------------------------------------
    # Serve a basic health endpoint (http://192.168.0.127:8080/health).
    health

    # Log and serve SERVFAIL for malformed/misrouted requests
    # instead of dropping them silently.
    errors

    # Tell systemd / orchestrator when CoreDNS is ready to take traffic.
    ready

    # --- Static records for DHCP-assigned hosts ------------------------
    # /etc/coredns/hosts contains lines like:
    #   192.168.0.250  home.mylan.  home
    #   192.168.0.45   barn.mylan.  barn
    # (the trailing short-name alias is what makes bare "home"/"barn"
    # resolve, thanks to the "." zone above)
    #
    # `fallthrough` ensures names not present in the file are not
    # short-circuited here and continue on to `forward`.
    hosts /etc/coredns/hosts {
        fallthrough
    }

    # --- Upstream forwarding ------------------------------------------
    # Anything not answered above is forwarded to the LAN gateway first,
    # then to public DNS.
    forward . 192.168.1.1 8.8.8.8 {
        prefer_udp
    }
}
```

### What each plugin does here

| Plugin | Purpose |
|---|---|
| `log` | Log every query and its response (your "logging of CoreDNS actions"). |
| `whoami` | Add the client source address to each log line. |
| `health` | `http://192.168.0.127:8080/health` liveness endpoint. |
| `errors` | Answer `SERVFAIL` for malformed/misrouted requests instead of dropping. |
| `ready` | Signal readiness to systemd/orchestrators. |
| `hosts … { fallthrough }` | Serve the static records; `fallthrough` lets unknown names continue to `forward`. |
| `forward .` | Upstream resolution for everything the zone doesn't answer. |

---

## 3. The hosts file

Drop this in `/etc/coredns/hosts`.

```
# /etc/coredns/hosts
# Static hostname mappings for MYLAN (192.168.0.0/24)
# Format: <IP>  <fqdn>  <aliases>

192.168.0.127  coredns.mylan.  coredns
192.168.0.250  home.mylan.     home
192.168.0.45   barn.mylan.     barn
```

> **Format is identical to `/etc/hosts`:** `IP  FQDN  [aliases]`.
> The trailing short-name alias is load-bearing — it's what makes bare
> `barn` / `home` work (see §6).

---

## 4. Install & run (native + systemd)

```bash
# Make sure the binary is on PATH
which coredns
#   -> if not installed:  apt install coredns  /  dnf install coredns
#      or point ExecStart at your static binary's absolute path.

# Place the config files
sudo mkdir -p /etc/coredns
sudo cp Corefile /etc/coredns/Corefile
sudo cp hosts     /etc/coredns/hosts
sudo chmod 644 /etc/coredns/Corefile /etc/coredns/hosts

# Install the unit (see §5)
sudo cp coredns.service /etc/systemd/system/coredns.service

sudo systemctl daemon-reload
sudo systemctl enable --now coredns        # enable = start on boot
sudo systemctl status coredns
```

### Troubleshooting `status=217/USER`

`217/USER` means CoreDNS started, hit an error, and exited itself. The
real reason is in the journal — read it first:

```bash
sudo journalctl -u coredns -n 50 --no-pager
sudo /usr/local/bin/coredns -conf /etc/coredns/Corefile   # see stderr directly
```

Common causes (and fixes) in a minimal-but-hardened unit:

| Symptom | Likely cause | Fix |
|---|---|---|
| `No such user: coredns` / `Failed at step USER` | `User=coredns` but the user doesn't exist | `sudo useradd -rS coredns` — **or** remove the line |
| `Permission denied` on config or log path | `ProtectSystem=strict` / `ProtectHome=true` | Drop those directives; add `ReadWritePaths=` for what you need |
| Wrong binary path | `ExecStart` points at a non-existent path | Correct `ExecStart=` (see `which coredns`) |

A safe, minimal unit that will reliably start is in §5.

---

## 5. systemd unit

`coredns.service`:

```ini
[Unit]
Description=CoreDNS DNS Server
Documentation=https://coredns.io
Wants=network-online.target
After=network-online.target

[Service]
# Adjust the binary path if yours lives elsewhere.
#   which coredns   /   dpkg -L coredns   /   rpm -ql coredns
ExecStart=/usr/local/bin/coredns /etc/coredns/Corefile
Restart=on-failure
RestartSec=2
LimitNOFILE=65536

# --- Optional hardening (re-enable one at a time once it's running) ---
# NoNewPrivileges=true
# PrivateTmp=true
# User=coredns          # requires: sudo useradd -rS coredns
# ProtectSystem=strict
# ProtectHome=true
# ReadWritePaths=/etc/coredns

[Install]
WantedBy=multi-user.target
```

`enable` is the actual "start on boot" step; `Wants/After=network-online.target`
keeps CoreDNS from starting before the uplink it forwards to is reachable.

---

## 6. Why bare short names work (the `.` zone trick)

**The problem:** a client typing `dig barn` sends a query for `barn.` (an
absolute, single-label name). CoreDNS's `mylan.` zone doesn't match `barn.`,
so it falls through to `forward` and asks upstream — which knows nothing
about your LAN → NXDOMAIN.

**The fix — two pieces working together:**

1. **Short-name aliases in the hosts file.** CoreDNS's `hosts` plugin stores
   *every* name on a line, so `barn.mylan.  barn` registers both keys.
2. **A `.` in the zone list** (`mylan. . {`). Adding the root zone to the
   same server block means a query for `barn.` now lands in a zone where the
   `hosts` plugin can find the `barn.` alias → `192.168.0.45`. Names not in
   the hosts table still hit `fallthrough` → `forward`, so upstream
   resolution is unchanged.

> **Caveat:** this is a homelab pattern, not a purist one (real DNS clients
> are supposed to append their own `search` domain). It's a clean way to make
> `barn` work everywhere without depending on each client's `resolv.conf`.
> It also composes fine with the "proper" fix below.

### The alternative (or complementary) approach: DHCP option 15

The DNS-correct way is to have the client append the search domain. Set the
**domain name (DHCP option 15)** to `mylan` on your DHCP server so clients
receive a `search mylan` line:

```
# example: dnsmasq
domain=mylan
```

With option 15 set, `dig barn` from a client tries `barn.mylan` and it works
even without the `.` zone. The `.` zone makes the *server* robust to clients
that don't have the search domain — belt and suspenders.

Verify what a client actually has:

```bash
cat /etc/resolv.conf
#   want to see:
#     nameserver 192.168.0.127
#     search mylan
```

---

## 7. Verifying it all works

```bash
# Bare short name (the case that previously failed)
dig @192.168.0.127 barn        +short    # -> 192.168.0.45

# FQDN
dig @192.168.0.127 barn.mylan. +short    # -> 192.168.0.45
dig @192.168.0.127 home        +short    # -> 192.168.0.250

# Confirm forwarders still work (internet name)
dig @192.168.0.127 example.com +short

# Watch CoreDNS log a query
sudo journalctl -u coredns -f
#   (in a separate shell:)
dig @192.168.0.127 barn        +short
```

---

## 8. Files

| File | Purpose |
|---|---|
| `Corefile` | CoreDNS zone/plugins config (logging, hosts, forward, `.` zone). |
| `hosts` | Static name→IP records for `home`, `barn`, and the CoreDNS box. |
| `coredns.service` | systemd unit for autostart + restart-on-failure. |

