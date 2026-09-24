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
