# No IP Address assigned

When a Linux virtual machine boots, it should normally receive an IP address automatically from the network's **DHCP (Dynamic Host Configuration Protocol)** server. In a typical home network, the DHCP server is your Wi-Fi router.

If no IP address is assigned, there are several possible reasons and fixes.

---

## 1. Verify the Network Interface

First, ensure that the network interface exists and is enabled.

```bash
ip link
```

Example:

```text
1: lo: ...
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
```

If the interface is present and in the `UP` state, proceed to the next step.

---

## 2. Check NetworkManager Device Status

Check whether NetworkManager detects the interface.

```bash
nmcli device status
```

Example:

```text
DEVICE   TYPE      STATE          CONNECTION
enp0s3   ethernet  disconnected   --
```

If the interface is **disconnected**, no DHCP request will be sent.

---

## 3. Check Existing Connection Profiles

List all configured NetworkManager connection profiles.

```bash
nmcli connection show
```

Example:

```text
NAME    UUID                                  TYPE      DEVICE
ens33   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  ethernet  --
```

Sometimes the **connection profile** is configured for a different interface than the one currently present.

For example:

- Actual interface: `enp0s3`
- Connection profile: `ens33`

This commonly happens when a **VMware image** is used inside **VirtualBox** because each hypervisor presents different virtual network hardware.

---

## 4. Create a New DHCP Connection

If no suitable connection profile exists, create one for the correct interface.

```bash
sudo nmcli connection add type ethernet ifname enp0s3 con-name enp0s3
```

This creates a new NetworkManager connection profile that uses DHCP by default.

---

## 5. Bring the Connection Up

Activate the newly created connection.

```bash
sudo nmcli connection up enp0s3
```

If successful, NetworkManager requests an IP address from the DHCP server.

---

## 6. Verify the Assigned IP Address

Check whether an IP address has been assigned.

```bash
ip addr
```

Example:

```text
inet 192.168.1.25/24
```

If an address is present, the DHCP configuration is working correctly.

---

# Alternative: Modify an Existing Connection

Instead of creating a new connection, you can update an existing one to use the correct interface.

For example:

```bash
sudo nmcli connection modify ens33 connection.interface-name enp0s3
sudo nmcli connection up ens33
```

This is useful when the existing connection profile simply references the wrong network interface.

---

# Common Causes

| Problem | Solution |
|----------|----------|
| Interface exists but no IP address | Verify that a NetworkManager connection profile exists. |
| Connection profile references `ens33` while interface is `enp0s3` | Create a new connection profile or modify the existing one. |
| DHCP never assigns an address | Ensure the VirtualBox network adapter is connected and configured correctly (e.g., Bridged Adapter or NAT). |
| `nmcli connection up` reports "No suitable device found" | The connection profile is bound to the wrong interface name. |

---

# Notes

- `ip addr add` only assigns a **temporary** IP address. The address is lost after a reboot.
- `nmcli` creates a **persistent** NetworkManager configuration that survives reboots.
- DHCP (`ipv4.method auto`) is the default method used by NetworkManager unless configured otherwise.
- VMware images imported into VirtualBox often require the connection profile to be recreated or updated because the virtual network interface name changes (for example, `ens33` → `enp0s3`).