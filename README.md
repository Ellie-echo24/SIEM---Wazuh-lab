# SIEM---Wazuh-lab
VMware Workstation Homelab SIEM Deployment with Wazuh - Attack detection and MITRE Att&amp;ck Mapping
<p>A home lab project demonstrating SIEM deployment, real-time attack simulation, and alert triage, developed entirely in VMware Workstation using an isolated attacker victim network topology.</p>
<h2>Overview</h1>
<p>This project implements Wazuh, an open-source SIEM platform, as an all-in-one deployment on an Ubuntu Server virtual machine. A separate Kali Linux virtual machine is used to conduct controlled attack simulations, demonstrating that the environment can identify, generate, and classify malicious activity—not simply complete the installation process.</p>
<h2>Demonstrated capabilities</h2>
<p>- Configuring the Wazuh manager, indexer, and dashboard components.

- Developing an isolated VMware network using Host-only networking, with NAT enabled for external internet connectivity.

- Resolving common Linux administration issues, including LVM and disk-space allocation, incomplete package removal, and service-port conflicts.

- Conducting an SSH brute-force simulation with Hydra.

- Investigating generated security alerts through the Wazuh Threat Hunting dashboard.

- Evaluating Wazuh’s automated mapping of detected activity to the MITRE ATT&CK framework.

</p>
<h2>Lab Architecture</h2>

| System | Role | Network Configuration |
|---|---|---|
| Kali Linux | Attacker machine | Host-only: `192.168.21.0/24` |
| Ubuntu Server 26.04 (`ghostface`) | Wazuh manager, indexer, and dashboard in an all-in-one deployment | Host-only: `192.168.21.0/24`<br>NAT: `192.168.165.0/24` for package installation |

<p>Both virtual machines run under VMware Workstation. The Host-only network isolates attacker-to-target traffic from the host machine’s physical network. A second network interface was added to the Wazuh virtual machine using NAT to provide temporary internet access for downloading packages during installation. The reason for this configuration is documented in the [Build Log](./BUILD_LOG.md).</p>

<h2>Setup</h2>

The complete step-by-step build log, including the issues encountered and their resolutions, is available in [`docs/setup-log.md`](../docs/setup-log.md).

### Overview

- Created two virtual machines in VMware Workstation:
  - Kali Linux as the attacker machine.
  - Ubuntu Server 26.04 as the target and Wazuh host.
- Configured Host-only networking to allow communication between the virtual machines while isolating the lab from the host LAN.
- Installed Wazuh 4.14.7 as an all-in-one deployment containing the manager, indexer, and dashboard using the official installation script.
- Verified access to the Wazuh dashboard.
- Confirmed that the Wazuh manager was ingesting its own system logs.

<h2>Issues Encountered and Resolutions</h2>

The installation did not succeed on the first attempt—or the third. These issues are documented intentionally because diagnosing and resolving failed installations demonstrates practical troubleshooting skills more effectively than a clean installation alone.

| Issue | Cause | Resolution |
|---|---|---|
| Wazuh installer script failed to download | The Host-only network did not provide an internet route. | Added a second network interface using NAT. |
| Filebeat installation failed during the installation process | The root LVM volume was undersized at approximately 19 GB, causing the system to run out of disk space during package downloads. | Extended the logical volume using `lvextend` and resized the filesystem with `resize2fs`, then reran the installer. |
| `dpkg --purge wazuh-manager` repeatedly failed | Broken `prerm` and `postrm` maintainer scripts referenced paths that had already been deleted during a previous failed installation. | Renamed the broken maintainer scripts in `/var/lib/dpkg/info/` to bypass them, then purged the package successfully. |
| Ports `1515` and `55000` were already in use during reinstallation | Wazuh processes from the failed installation were still running and occupying the ports. | Used `lsof -i` to identify the processes, terminated them by PID, and reran the installation. |
## Attack Simulation

An SSH brute-force simulation was conducted from the Kali Linux attacker machine against the Wazuh-monitored Ubuntu Server. Hydra was used with a custom wordlist containing only incorrect passwords. Because the legitimate password was already known, the purpose of the exercise was to generate realistic failed-authentication events and evaluate Wazuh’s detection capabilities—not to compromise the system.

```bash
hydra -l admin_ghostface -P wrong-passwords.txt ssh://192.168.21.129 -t 4
```

The exact command and custom wordlist are available in the [`attack-scripts/`](../attack-scripts/) directory.

### Results

Wazuh’s default detection rules identified the simulated attack without requiring any custom rules. The detected activity was also automatically classified against the MITRE ATT&CK framework.

| Result | Details |
|---|---|
| Authentication failures | 112 failed authentication attempts were recorded during the attack window. |
| Authentication successes | Five successful authentications were identified as legitimate operator SSH sessions. These were verified using the source IP address and event timing and were not associated with the simulated attack. |
| MITRE ATT&CK classifications | The alerts were mapped to Password Guessing, SSH, Brute Force, and Valid Accounts techniques. |

This confirmed that the Wazuh deployment could detect, alert on, and classify suspicious SSH authentication activity using its default detection ruleset.
