## Troubleshooting RDP Connection to Windows 11 VM (Proxmox)

### Overview
While working with virtual machines in Proxmox, I configured Remote Desktop Protocol (RDP) access. I was able to successfully connect to a Windows Server 2022 VM, but encountered persistent authentication issues when attempting to connect to a Windows 11 Pro VM.

---

### Issue
RDP connection to the Windows 11 Pro VM failed with a **credential authentication error**, even though the correct username and password were entered.

---

### Troubleshooting Steps
I performed the following checks and actions:

- Verified correct IP address (local network and Tailscale)
- Attempted connection from multiple devices
- Tested different username formats (e.g., `.\\username`)
- Reset and changed user passwords
- Confirmed Remote Desktop was enabled
- Verified user was added to the "Remote Desktop Users" group
- Disabled **Windows Hello (PIN-based sign-in requirement)**
- Tested network connectivity (ping)

Despite all attempts, the issue persisted only on the Windows 11 VM, while Windows Server 2022 worked without any issues.

---

### Root Cause
The issue was caused by using a **Microsoft (online) account** instead of a **local account**.

- The account authentication depended on Microsoft services
- RDP requires local password-based authentication
- PIN-based login (Windows Hello) is not supported for RDP

---

### Resolution
The issue was resolved by enabling and using a local administrator account:

```cmd
net user Administrator /active:yes
net user Administrator <password>
```

---

### Network Setup

To enable secure remote access, I configured a VPN using Tailscale.

- All virtual machines were connected to a shared private network via Tailscale
- Each VM received an internal Tailscale IP address (100.x.x.x)
- This allowed remote access as if all devices were on the same local network

This approach eliminated the need for port forwarding and reduced exposure to external threats.

---

### Additional Finding (User Context & Tailscale)

During testing, I discovered that Tailscale connectivity depended on the user context:

- Tailscale was initially configured under the primary user account
- After switching to the Administrator account, the Tailscale connection was no longer active
- As a result, the VM became unreachable via its Tailscale IP, and RDP access failed

To resolve this:

- Tailscale was installed and authenticated under the Administrator account
- This ensured persistent remote access regardless of which user account was active

---

### Notes / Limitations

Although I identified the root cause of the issue (Microsoft account vs local account), I was not able to fully resolve authentication using the original user account.

I spent approximately 2 evenings troubleshooting this issue, testing multiple approaches (credential changes, Windows Hello settings, connectivity checks), but did not achieve a stable solution.

As a workaround:

- I enabled and used a local Administrator account
- I configured Tailscale under that account to maintain remote access
- I removed dependency on the original user account

This approach restored functionality, although a more complete solution for Microsoft account-based RDP authentication may require further investigation.
