🔧 Unbound DNS Installation Guide for Windows

This guide walks you through installing and configuring Unbound — a validating, recursive, and caching DNS resolver — on Windows. Most tutorials focus on Linux, so this tutorial can help to fills the gap for Windows users.
🖥️ System Info
This setup was tested on:
💻 Hardware: Intel i3-8100T (HP ProDesk 400), 8GB RAM
🪟 OS: Windows 10 Pro
🧠 Future use: Integrated with Pi-hole running on Raspberry Pi 3


📥 1. Download Unbound
Install the latest Windows version:
🔗 Unbound Setup 
https://www.nlnetlabs.nl/projects/unbound/download/
The default install location is:
<pre> C:\Program Files\Unbound\ </pre>


🌐 2. Download Root Hints File
The root.hints file is a list of root DNS server IP addresses used by Unbound to start recursive DNS resolution from scratch.
📥 To download:
```
    https://www.internic.net/domain/named.root
```
Save the file as root.hints
Move it to:
<pre>
C:\Program Files\Unbound\ 
</pre>


🔑 3. Generate root.key for DNSSEC
The root.key file is used by Unbound to validate DNSSEC signatures from the root zone.
🛠 Steps:
Open an elevated Command Prompt (Run as Administrator), and go to:
```
cd "C:\Program Files\Unbound"
```
and run:
```
unbound-anchor.exe -a "C:\Program Files\Unbound\root.key"
```
This generates root.key in the Unbound folder.


⚙️ 4. Modify service.conf Configuration File
Open the C:\Program Files\Unbound\service.conf file in your preferred editor (e.g., Notepad). Replace its contents with the one of the following config mode:

UNBOUND - RECURSIVE MODE + DNSSEC ENABLE
```
server:

    # Basic setup
    verbosity: 1
    interface: 127.0.0.1
    port: 53
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.1.0/16 allow

    do-ip4: yes
    do-ip6: no
    prefer-ip6: no
    do-udp: yes
    do-tcp: yes

    # Enable DNSSEC validation (disable val-permissive-mode)
    val-permissive-mode: no
    module-config: "validator iterator"

    # Trust anchors and root servers
    root-hints: "C:\Program Files\Unbound\root.hints"
    auto-trust-anchor-file: "C:\Program Files\Unbound\root.key"

    # Security & Privacy
    hide-version: yes
    identity: ""
    hide-identity: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    harden-algo-downgrade: yes
    harden-short-bufsize: yes

    # EDNS & Buffer size (reduce fragmentation issues)
    edns-buffer-size: 1232
    max-udp-size: 1232

    # Prefetch to reduce future latency
    prefetch: yes
    prefetch-key: yes
    serve-expired: yes
    serve-expired-ttl: 1800
    serve-expired-client-timeout: 1800

    # Caching performance
    msg-cache-size: 100m
    rrset-cache-size: 200m
    key-cache-size: 64m
    msg-cache-slabs: 4
    rrset-cache-slabs: 4
    infra-cache-slabs: 4
    key-cache-slabs: 4

    # Threading - use max 2 cores
    num-threads: 2
    num-queries-per-thread: 225
    outgoing-range: 450
    outgoing-num-tcp: 40
    incoming-num-tcp: 60
    tcp-idle-timeout: 600
    so-rcvbuf: 4m
    so-sndbuf: 4m

    # Performance features
    rrset-roundrobin: yes
    minimal-responses: yes
    qname-minimisation: yes

    # Prevent DNS leaks via private address space
    private-address: 10.0.0.0/8
    private-address: 172.16.0.0/12
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: fd00::/8
    private-address: fe80::/10
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32

    # Required CA bundle for TLS (still safe to include, even if not forwarding)
    tls-cert-bundle: "C:\Program Files\Unbound\ca-bundle.crt"
```
UNBOUND - FORWARDING MODE

In Forwarding mode Unbound needs a list of trusted root certificates to verify Cloudflare's TLS cert.

If ca-bundle.crt doesn’t exist, download it from:
```
https://curl.se/ca/cacert.pem
```
Rename cacert.pem to ca-bundle.crt, then move it to Unbound installation folder:
```
C:\Program Files\Unbound\
```
```
server:

    # Basic setup
    verbosity: 1
    interface: 127.0.0.1
    port: 53
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.1.0/16 allow

    do-ip4: yes
    do-ip6: no
    prefer-ip6: no
    do-udp: yes
    do-tcp: yes

    # Disable Unbound's own DNSSEC validation
    val-permissive-mode: yes
    module-config: "iterator"

    # Trust anchors and root servers
    root-hints: "C:\Program Files\Unbound\root.hints"
    auto-trust-anchor-file: "C:\Program Files\Unbound\root.key"

    # Security & Privacy
    hide-version: yes
    identity: ""
    hide-identity: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    harden-algo-downgrade: yes
    harden-short-bufsize: yes

    # EDNS & Buffer size (reduce fragmentation issues)
    edns-buffer-size: 1232
    max-udp-size: 1232

    # Prefetch to reduce future latency
    prefetch: yes
    prefetch-key: yes
    serve-expired: yes
    serve-expired-ttl: 1800
    serve-expired-client-timeout: 1800

    # Caching performance
    msg-cache-size: 100m
    rrset-cache-size: 200m
    key-cache-size: 64m
    msg-cache-slabs: 4
    rrset-cache-slabs: 4
    infra-cache-slabs: 4
    key-cache-slabs: 4

    # Threading - only use 2 cores
    num-threads: 2
    num-queries-per-thread: 225
    outgoing-range: 452
    outgoing-num-tcp: 40
    incoming-num-tcp: 60
    tcp-idle-timeout: 600
    so-rcvbuf: 4m
    so-sndbuf: 4m

    # Use this only if you're running multiple Unbound instances (Linux)
    # so-reuseport: no

    # Performance features
    rrset-roundrobin: yes
    minimal-responses: yes
    qname-minimisation: yes

    # Prevent DNS leaks via private address space
    private-address: 10.0.0.0/8
    private-address: 172.16.0.0/12
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: fd00::/8
    private-address: fe80::/10
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32

    # Required CA bundle for TLS
    tls-cert-bundle: "C:\Program Files\Unbound\ca-bundle.crt"

forward-zone:
    name: "."
    forward-tls-upstream: yes

    # Cloudflare DNS servers with DNS-over-TLS
    forward-addr: 1.1.1.1@853#cloudflare-dns.com
    forward-addr: 1.0.0.1@853#cloudflare-dns.com
    forward-addr: 2606:4700:4700::1111@853#cloudflare-dns.com
    forward-addr: 2606:4700:4700::1001@853#cloudflare-dns.com

```
💡 Note: If port 53 is already in use (e.g., by Pi-hole), change it to another port and reflect that in your DNS setup later.


🔁 5. Restart Unbound Service
Open an elevated Command Prompt and run:
```
net stop Unbound
```
then
```
net start Unbound
```
If Unbound fails to start, check the config with:
```
unbound-checkconf "C:\Program Files\Unbound\service.conf"
```


🧪 6. Test DNS Resolution
Open Command Prompt and run:
<pre>
nslookup google.com 127.0.0.1
</pre>
If Unbound is working correctly, it will return the IP address of google.com.

🧪 7. Test DNSSEC 
Go to:
```
https://dnscheck.tools/
```
or
```
https://www.cloudflare.com/ssl/encrypted-sni/
```

🌐 8. Integrate Unbound with Router, Pi-hole (if available), or Install Locally on Your Laptop

Example:
If your Unbound server IP is `192.168.1.252` and it runs on port `53`:

Using Pi-hole:
  Go to Pi-hole's DNS settings and enter `192.168.1.252#53` in the custom DNS field.

Without Pi-hole (Router only):
  Simply set your router’s DNS to point to `192.168.1.252`.

Installed on a Laptop:
  If you install Unbound locally on your laptop, it can protect your DNS traffic when connected to public Wi-Fi.
  Set your DNS to `127.0.0.1` to route queries through your local Unbound instance.

✅ Done!
You now have a fully working Unbound DNS resolver on Windows, acting as a secure and private DNS backend for your Pi-hole or home network. 🎉

Thank You.
