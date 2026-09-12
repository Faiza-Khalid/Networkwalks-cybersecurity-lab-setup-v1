# Networkwalks-cybersecurity-lab-setup-v1
Kali Linux Attack Lab on Oracle VirtualBox
# README — Kali Linux Attack Lab on Oracle VirtualBox
### Private subnet `10.0.0.0/24` · NATNetwork · Static IP `10.0.0.2/24` · Full Internet Access
*Evidence screenshots: S1–S5 (referenced per step)*

---

## 1. Task Requirements

- Use **VirtualBox** as the virtualization base (latest recommended version).
- Set up **Kali Linux** as the attacking/hacker machine.
- Set the lab network in subnet **10.0.0.0/24** using a **NAT Network (NATNetwork)**.
- Enable **clipboard** and **file drag/drop** in the VM settings.
- Enable **shared folders**, sharing the host's **/downloads** folder.
- Kali Linux IP address must be **10.0.0.2/24**.
- Kali Linux must have **full Internet access**.

---

## 2. Problem Statements

**Problem Statement 1 — Lab build (design problem).**
Build a realistic, repeatable penetration-testing environment in which the attack VM lives on a *controlled private subnet* (`10.0.0.0/24`) with a *predictable static address* and *full outbound Internet*, while host↔VM convenience features (clipboard, drag/drop, shared folder) work — **without** exposing the host OS or the production LAN to attack traffic.

**Problem Statement 2 — Connectivity failure encountered during the build (documented in snapshot step 10 picture).**
After assigning the static IP `10.0.0.2` with gateway `10.0.0.1`, the Kali VM could not resolve domain names (`ping google.com` → *"Temporary failure in name resolution"*), and all outbound traffic failed with **"Destination Host Unreachable"** (100% packet loss to external IPs such as `8.8.8.8`). Manually overriding DNS in `/etc/resolv.conf` to `8.8.8.8` **failed**, because there was no working route to reach the DNS server. Root cause: a **subnet mismatch** between the VirtualBox NATNetwork prefix and the static IPv4 profile — a Layer-3 fault, not a DNS fault.

---

## 3. Environment

| Component | Value |
|---|---|
| Hypervisor | Oracle VirtualBox 7.x (latest recommended) + Extension Pack |
| Guest (attacker VM) | Kali Linux 2026.2 amd64 official appliance (`kali-linux-2026.2-virtualbox-amd64`) |
| Lab network | `NatNetwork`, IPv4 prefix `10.0.0.0/24`, DHCP enabled, gateway `10.0.0.1` |
| VM address | Static `10.0.0.2/24`, DNS `8.8.8.8` |

---

## 4. Step-by-Step Instructions

### Step 1 — Install Oracle VirtualBox (latest recommended) + Extension Pack
- Download and install the latest recommended VirtualBox build for your host OS, plus the matching Extension Pack.
- 🔐 **Cyber logic:** The hypervisor is the security boundary of the whole lab. Current builds patch known VM-escape and privilege-escalation CVEs — a compromised or malware-infested attack VM must never be able to reach the host. The Extension Pack provides the USB/clipboard/drag-drop capabilities required in later steps.

<img width="960" height="464" alt="image" src="https://github.com/user-attachments/assets/cebf6b14-9bcb-4014-8e86-108daf312acf" />

### Step 2 — Deploy Kali Linux as the attacking machine
- Import the official Kali Linux 2026.2 VirtualBox appliance (or create a VM and install the Kali ISO); keep the VM name recognizable, e.g. `kali-linux-2026.2-virtualbox-amd64` *(visible in provied picture)*.
- 🔐 **Cyber logic:** Compartmentalization. Kali ships 600+ offensive tools; keeping them — and any malware samples or exploit detonations — on a dedicated, disposable box isolates that risk from your daily OS. A "burned" attack VM is restored from snapshot, not rebuilt.

<img width="960" height="476" alt="image" src="https://github.com/user-attachments/assets/694fa5ca-3520-4a05-9a47-0da6d3bf0800" />

### Step 3 and 4 — Create the NAT Network on 10.0.0.0/24 
- VirtualBox Manager → **File/Tools → Network Manager → NAT Networks** → **Create**.
- Set **Name:** `NatNetwork`, **IPv4 Prefix:** `10.0.0.0/24`, tick **Enable DHCP**, leave IPv6 off → **Apply** *(the picture shows exactly this state, DHCP Server: Enabled)*.
- 🔐 **Cyber logic:** RFC 1918 private addressing keeps lab traffic segregated from your real LAN, and NAT gives the VM *default-deny inbound* posture — it can initiate outbound sessions (updates, wordlists, C2-style listeners over the Internet) but is unreachable from outside, exactly like a host sitting behind a firewall. A fixed, documented /24 also mirrors real engagement practice: `10.0.0.0/24` becomes your Rules-of-Engagement scope boundary.
— Attach the VM's NIC to the NAT Network
- VM **Settings → Network → Adapter 1** → Enable → **Attached to: NAT Network** → **Name: NatNetwork** → OK.
- 🔐 **Cyber logic:** Deliberately avoids *Bridged* mode, which would place the attack box directly on your home/corporate LAN — an accidental `nmap` sweep against real infrastructure is both dangerous and potentially illegal. NAT Network confines recon/exploitation to the lab segment while still allowing Internet egress.

<img width="960" height="504" alt="S3" src="https://github.com/user-attachments/assets/dbcb612a-9dd7-484c-9f6a-aafd5c7191ba" />


### Step 5 — Enable Shared Clipboard and Drag'n'Drop (Bidirectional)
- VM **Settings → General → Advanced** tab → **Shared Clipboard: Bidirectional** and **Drag'n'Drop: Bidirectional** → OK.
- 🔐 **Cyber logic:** You will constantly move hashes, payloads, screenshots and notes between host and VM. Using the hypervisor's isolated channel means you do **not** have to stand up network file-sharing services (SMB/SSH from the host), which would enlarge the attack surface and create a lateral-movement path that a malware sample could abuse.
  
- <img width="587" height="388" alt="image" src="https://github.com/user-attachments/assets/f162891c-ee57-4612-aff3-159b1c9cf8ba" />

### Step 6 and 7 — Share the host `/downloads` folder  
- VM **Settings → Shared Folders** → **+** → **Folder Path:** the host's `/downloads` folder → tick **Auto-mount** (and Make Permanent) → OK.
- Boot Kali: the share appears on the desktop as **`sf_Downloads`**. If permission is denied: `sudo usermod -aG vboxsf kali`, then log out/in.
- 🔐 **Cyber logic:** One controlled, auditable transfer point for wordlists, exploit files and captured evidence (pcaps, hashes, screenshots) — the lab equivalent of an engagement workspace that separates "tooling & loot" from the OS. Auto-mount guarantees evidence always lands in the same location, supporting tidy chain-of-custody habits.
 — Configure the static IP 10.0.0.2/24

<img width="587" height="388" alt="image" src="https://github.com/user-attachments/assets/843cd010-0cb2-4fc8-a7de-1cccabeb2e8f" />

- In Kali run `nm-connection-editor` → **Wired connection 1** → **IPv4 Settings** tab → **Method: Manual**.
- **Address:** `10.0.0.2` · **Netmask:** `24` · **Gateway:** `10.0.0.1` · **DNS servers:** `8.8.8.8` → **Save**, then reconnect the connection *(S5 shows this exact profile)*.
- 🔐 **Cyber logic:** An attack box needs a *stable identity*. Reverse shells, listener configs, scan reports and target-side artifacts all reference the attacker IP; a DHCP-drifting address breaks reproducibility and engagement documentation. A pinned `10.0.0.2` also lets you write precise scope/firewall rules, and `8.8.8.8` keeps name resolution independent of the lab DHCP lease.

<img width="960" height="504" alt="S4" src="https://github.com/user-attachments/assets/72f598a2-97a4-4140-a892-d04783d64107" />


### Step 8 and 9 — Verify Layers 1–3 with `ip a`
- In a terminal run `ip a`; expect `eth0 … inet 10.0.0.2/24 brd 10.0.0.255 scope global … state UP` 
- 🔐 **Cyber logic:** Baseline the stack bottom-up (OSI model): confirm link state and addressing **before** blaming DNS or applications. This is the same discipline used mid-operation when verifying whether a pivot or C2 channel died at L2/L3 or higher up.
 — Verify full Internet: DNS + ICMP + HTTPS
- `ping google.com` → expect the name to **resolve** (e.g. `142.250.202.14`) with **0% packet loss** — S2 shows 7/7 received.
  
- <img width="960" height="504" alt="S1" src="https://github.com/user-attachments/assets/7dcce95a-dda1-49f9-99cc-a35f93a3db62" />

- Open Firefox and load `https://www.google.com` (run a search) — provied picture shows live results.
- 🔐 **Cyber logic:** Proves name resolution (UDP/53), reachability (ICMP) and encrypted web (TCP/443) in one pass — an attack VM depends on the Internet for `apt` updates, wordlists and exploit retrieval. Splitting the tests (ping-with-name vs. browser) tells you *which layer* is broken the next time something fails.

- <img width="960" height="504" alt="S2" src="https://github.com/user-attachments/assets/5064647f-2a24-458a-9fe4-e9e53155c0ed" />


### Step 10 — Snapshot the known-good state 
- VirtualBox Manager → select VM → **Snapshots → Take**; name it (e.g. *"learning experience of Network Resolution & Subnet Mismatch Fix"*) and write the problem + fix in the description → **Apply** as in provided picture.
- 🔐 **Cyber logic:** Snapshots are the lab's incident-recovery and forensics control: after detonating malware or breaking the stack, roll back to a known-good baseline in seconds. The description doubles as an engagement log of what broke and how it was fixed.

<img width="960" height="504" alt="S5" src="https://github.com/user-attachments/assets/533ec738-8899-43d0-a8de-3007860460f4" />
---

## 5. Troubleshooting — Problems & Solutions

### Problem A: `ping google.com` → "Temporary failure in name resolution"
- **Attempt 1 (failed):** manually set `nameserver 8.8.8.8` in `/etc/resolv.conf`.
- **Why it failed:** DNS is an application-layer service; with no Layer-3 route, queries to `8.8.8.8` had nowhere to go. Fixing DNS cannot fix a dead route.

### Problem B: "Destination Host Unreachable", 100% loss to external IPs
- **Root cause:** **subnet mismatch** — the NATNetwork prefix was not `10.0.0.0/24`, so the configured gateway `10.0.0.1` lay outside the VM's local segment; ARP for the gateway never resolved and every packet died at the first hop.
- **Solution:** set the NATNetwork IPv4 Prefix to exactly `10.0.0.0/24` step 3 and keep the Manual profile `10.0.0.2/24`, gw `10.0.0.1`, DNS `8.8.8.8` *(S5)*, then reconnect. Verified by `ip a` + `ping` step 8 and 9.
- 🔐 **Cyber logic:** Read error messages per OSI layer. *"Destination Host Unreachable"* is a local-segment L2/L3 error (gateway ARP failure); *"name or service not known"* is a DNS-layer error. Correctly attributing a failure to its layer is the difference between minutes and hours of debugging — the same skill used when troubleshooting C2 channels, pivots and firewall egress on real assessments.

---

## 6. Verification Matrix (requirement → evidence)

| # | Requirement | Where it is satisfied | Evidence |
|---|---|---|---|
| 1 | VirtualBox base, latest recommended | Step 1 | Oracle VirtualBox Manager in all screenshots |
| 2 | Kali as attacker machine | Step 2 | `kali@kali` prompt / VM name (S2, S5) |
| 3 | NATNetwork `10.0.0.0/24` | Steps 3–4 | NatNetwork, prefix `10.0.0.0/24`, DHCP Enabled (S1) |
| 4 | Clipboard & drag/drop | Step 5 | Settings → General → Advanced = Bidirectional |
| 5 | Shared `/downloads` folder | Step 6 | `sf_Downloads` on Kali desktop (S5) |
| 6 | Static IP `10.0.0.2/24` | Steps 7–8 | Manual IPv4 profile (S5) + `ip a` (S2) |
| 7 | Full Internet access | Step 9 | `ping google.com` 0% loss (S2) + Google search in Firefox (S4) |
| 8 | Documentation of the fix | Step 10 | Snapshot with problem/solution description (S3) |

**Result:** all task requirements are met — Kali sits at `10.0.0.2/24` on the `NatNetwork 10.0.0.0/24`, with bidirectional clipboard/drag-drop, the auto-mounted `sf_Downloads` shared folder, and verified full Internet access (DNS + ICMP + HTTPS).
