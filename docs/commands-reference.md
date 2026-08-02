# Commands Reference

Commands used during this project, scoped to authorized testing on personally
owned lab equipment. All addresses are placeholders — replace them with your
own lab's actual values if you're reproducing similar tests on hardware you
own or are authorized to administer. **Do not run scanning tools against
networks or hosts you don't own or have explicit permission to test.**

## Windows client network validation

```powershell
ipconfig /all
ping <LAB-GATEWAY-IP>
nslookup netgear.com
ping 1.1.1.1
```

Confirms local addressing, gateway reachability, DNS resolution, and general
Internet reachability from the management workstation.

## Kali Linux VM addressing

```bash
ip -br address
ip route
nmcli device show eth0
```

Confirms the VM's assigned address and routing table — useful for verifying
whether a VM is in NAT or Bridged mode and what network it actually landed on.

## Authorized host discovery

```bash
sudo nmap -sn <LAB-SUBNET>/24
```

A ping-sweep style scan to confirm which lab devices are active, run only
against your own subnet.

## Router service enumeration

```bash
sudo nmap -sV --top-ports 20 <LAB-GATEWAY-IP>
```

Service/version detection against the top 20 most common ports. Scope this
tightly — there's rarely a reason to run a full 65535-port scan against your
own home router.

## HTTP/HTTPS behavior check

```bash
curl -I http://<LAB-GATEWAY-IP>
curl -k -I https://<LAB-GATEWAY-IP>
```

Confirms whether HTTP is still reachable even after preferring HTTPS in the
router's admin settings, and what authentication challenge (if any) each
protocol returns. `-k` skips certificate validation — appropriate here only
because the target is a self-signed local management certificate you already
know about.

## TLS / cipher review

```bash
sudo nmap -p 443 --script ssl-enum-ciphers <LAB-GATEWAY-IP>
```

Lists supported TLS versions and cipher suites. Useful for spotting
deprecated protocol support (e.g., TLS 1.0/1.1) — review the summary rather
than publishing the raw output, since detailed cipher/fingerprint data can
help profile a specific device.

## Windows endpoint exposure test

```bash
sudo nmap -sS -sV --top-ports 100 <LAB-CLIENT-IP>
```

An authorized external scan of your own Windows endpoint from another lab
host, used to confirm which locally listening services are actually reachable
through the host firewall.

## Windows Firewall log review

```powershell
Get-Content "$env:SystemRoot\System32\LogFiles\Firewall\pfirewall.log" -Tail 50
```

Requires firewall logging to be enabled first
(`Set-NetFirewallProfile -LogBlocked True` or via the GUI). Used to correlate
inbound drops against an authorized scan's source address and target ports.

## Windows service and listener review

```powershell
Get-NetFirewallRule | Where-Object {
    $_.DisplayName -like "*<service-name>*" -or
    $_.DisplayGroup -like "*<service-name>*"
} | Select-Object DisplayName, Enabled, Profile, Direction, Action

Get-Service <service-name> | Select-Object Name, DisplayName, Status

Get-NetTCPConnection -State Listen |
    Where-Object LocalPort -in <port1>,<port2>

Get-AuthenticodeSignature <path-to-executable> |
    Select-Object Path, Status, StatusMessage
```

Used to enumerate firewall rules, service state, active listeners, and code
signatures for endpoint services found during network scanning — the
PowerShell-side half of correlating "what's actually running" with "what's
actually reachable."
