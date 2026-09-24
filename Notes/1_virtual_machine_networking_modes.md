# 01 Virtual Machine Networking Modes

- NAT (Network Address Translation): The hypervisor creates an isolated private subnet where outbound traffic is translated via the host's IP address. The guest can reach external networks, but external hosts (and other VMs by default) cannot initiate connections to the guest without explicit port-forwarding rules.

- NAT Network: An internal virtual router shared across multiple VMs. VMs attached to the same NAT network can communicate directly with each other and access the external internet, while remaining shielded from unsolicited inbound connections from the physical LAN.

- Bridged Networking: The VM binds directly to the host’s physical network adapter, requesting its own IP address from the local physical DHCP server. The VM appears as an independent physical machine on your local Wi-Fi or Ethernet.

- Host-Only: Creates a completely isolated, internal virtual software switch accessible only by the host machine and other VMs assigned to that network. No traffic routes to the internet.

$\rightarrow$ Best Option for Pentesting Labs: Host-Only (or an isolated NAT Network). When running intentionally vulnerable machines like Metasploitable 3, Bridged mode exposes critical vulnerabilities directly to the physical local network (e.g., university Wi-Fi or home routers). Host-Only isolates dangerous attack surfaces while permitting full Kali-to-target communication.