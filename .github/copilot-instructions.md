## Copilot instructions for ONTAP MetroCluster documentation

### Repository overview
Product: ONTAP MetroCluster

NetApp MetroCluster combines array-based clustering with synchronous replication to deliver continuous availability, duplicating mission-critical data on a transaction-by-transaction basis across two geographically separated sites. MetroCluster enhances the built-in high availability and nondisruptive operations of ONTAP storage software by providing an additional layer of protection for the entire storage and host environment.

### Repository structure
- `install-ip/` – Installation and configuration for MetroCluster IP configurations, including IP switch cabling, port assignments, ONTAP Mediator setup, software configuration, and RCF (Reference Configuration File) usage.
- `install-fc/` – Installation and configuration for fabric-attached (FC) MetroCluster configurations, including FC switch cabling (Brocade and Cisco), FC-to-SAS bridge setup, port assignments, and RCF usage.
- `install-stretch/` – Installation and configuration for stretch MetroCluster configurations (two-node, bridge-attached and direct-attached SAS).
- `manage/` – Day-to-day management topics including switchover, healing, and switchback operations; data protection concepts; NVFAIL monitoring; and configuration monitoring.
- `maintain/` – Maintenance procedures for hardware components such as FC and IP switches, ATTO FibreBridge bridges, drive shelves, and IP interface management; includes firmware and RCF upgrades.
- `disaster-recovery/` – Step-by-step recovery procedures after a disaster, covering forced switchover, hardware replacement, and completing switchback for both MetroCluster FC and IP configurations.
- `upgrade/` – Controller upgrade, technology refresh, hardware expansion (two-node to four-node, four-node to eight-node), and transition procedures between configuration types.
- `transition/` – Procedures for transitioning from MetroCluster FC configurations to MetroCluster IP configurations (both disruptive and nondisruptive methods).
- `tiebreaker/` – Installation, configuration, and monitoring documentation for MetroCluster Tiebreaker software, which runs on a third-site Linux host to detect site failures and trigger alerts.
- `releasenotes/` – Release notes for MetroCluster features, platform support updates, Mediator changes, and Tiebreaker changes.
- `_include/` – Shared AsciiDoc content snippets and CSV cabling worksheets reused across multiple sections; not published directly.
- `media/` – Images and diagrams referenced by documentation pages.
- `redirects/` – URL redirect mappings for moved or renamed pages.

### Product-specific context

**Architecture and components:**
- A MetroCluster configuration consists of two ONTAP clusters located at geographically separated sites, each configured as an HA pair (except two-node stretch configurations).
- *DR group*: The fundamental replication unit; a four-node DR group has two nodes at each site. Configurations can have one DR group (four-node) or two DR groups (eight-node).
- *MetroCluster IP*: Uses IP switches for back-end connectivity and storage replication over the cluster interconnect network; supports ONTAP Mediator for automatic unplanned switchover (AUSO).
- *MetroCluster FC (fabric-attached)*: Uses FC switch fabrics (two redundant fabrics) and FC-to-SAS bridges (ATTO FibreBridge) to connect storage; supports ONTAP AUSO and Tiebreaker.
- *Stretch MetroCluster*: Two-node configurations where storage is directly attached (SAS optical) or bridge-attached, without FC switch fabrics between sites.
- *ONTAP Mediator*: A service installed on a third-site Linux host that provides a tie-breaking vote for automatic unplanned switchover in MetroCluster IP configurations; cannot be used simultaneously with Tiebreaker on the same configuration.
- *MetroCluster Tiebreaker*: Software on a third-site Linux host that monitors up to 15 MetroCluster configurations (IP, FC, and stretch) and triggers alerts on site failure; cannot be used simultaneously with ONTAP Mediator on the same MetroCluster IP configuration.
- *ATTO FibreBridge*: FC-to-SAS protocol bridge used in fabric-attached and stretch MetroCluster configurations to connect SAS disk shelves to FC switches.
- *ISL (Inter-Switch Link)*: The long-haul connection between FC or IP switches at the two MetroCluster sites.
- *RCF (Reference Configuration File)*: A switch configuration script provided by NetApp for Brocade FC, Cisco FC, and supported MetroCluster IP switches.

**Key concepts:**
- *Switchover*: The operation where one MetroCluster site takes over the other site's storage and SVM workloads; can be negotiated (planned) or forced (after a disaster).
- *Switchback*: The operation that returns workloads to the original site after recovery from a switchover.
- *Healing*: The intermediate step between switchover and switchback that resynchronizes and heals data and root aggregates; performed separately for data aggregates and root aggregates.
- *Mirrored aggregates*: RAID aggregates that are synchronously mirrored across MetroCluster sites as the default data protection mechanism.
- *Unmirrored aggregates*: Aggregates on a single site with no cross-site mirror; supported in MetroCluster IP (ONTAP 9.8 and later) and all MetroCluster FC/stretch configurations, but must be taken offline during switchover.
- *ADP (Advanced Disk Partitioning)*: Supported in MetroCluster IP configurations to improve storage utilization.
- *SVM (storage virtual machine)*: SVM configuration is continuously mirrored between MetroCluster clusters over the cluster peering network using the Configuration Replication Service.
- *AllLinksSevered*: A cluster status indicating that all links to the partner MetroCluster site have been lost; triggers Tiebreaker alerts and automatic switchover conditions.

**Naming conventions and terminology:**
- Configuration types are referred to as *MetroCluster IP*, *MetroCluster FC* (or *fabric-attached MetroCluster*), and *stretch MetroCluster*.
- Nodes follow a naming pattern: `node_A_1`, `node_A_2` (site A) and `node_B_1`, `node_B_2` (site B); clusters are `cluster_A` and `cluster_B`.
- *FC-VI adapter*: The Fibre Channel Virtual Interface adapter used for HA interconnect in fabric-attached MetroCluster configurations.
- Switch vendors supported: Brocade and Cisco for FC switches; Broadcom (BES-53248), Cisco (Nexus series), and NVIDIA (SN2100) for MetroCluster IP switches.
- *MACSEC*: Layer 2 encryption option available for Cisco IP switches in MetroCluster IP configurations.
- All SAN Array (ASA) systems are supported in MetroCluster configurations; ASA documentation references match the corresponding AFF model documentation.

### Typical user workflows

**Install MetroCluster IP:** Rack and cable hardware → Configure IP switches with RCF files → Set up ONTAP software on controllers → Configure cluster peering → Configure ONTAP Mediator (optional) → Test configuration

**Install fabric-attached MetroCluster:** Rack and cable hardware → Configure FC switches (Brocade or Cisco) with RCF files → Install and configure FibreBridge FC-to-SAS bridges → Set up ONTAP software → Configure cluster peering → Test configuration

**Perform negotiated switchover and switchback:** Verify MetroCluster health → Send AutoSupport notification → Perform negotiated switchover → Perform maintenance or recovery tasks → Heal data aggregates → Heal root aggregates → Perform switchback → Verify successful switchback

**Recover from a disaster (MetroCluster IP or FC):** Identify failure type → Perform forced switchover → Power on and reconfigure disaster site hardware → Restore connectivity → Reassign disks → Heal configuration → Perform switchback → Verify recovery

**Upgrade controllers:** Choose upgrade method (switchover/switchback or system controller replace command) → Prepare network configuration → Perform switchover → Uninstall old controllers → Set up and boot new controllers → Apply RCF files and set boot arguments → Perform switchback → Complete upgrade

**Transition MetroCluster FC to MetroCluster IP:** Assess transition requirements → Choose disruptive or nondisruptive procedure → Prepare IP controllers → Move cluster connections → Configure new IP switches → Complete transition → Verify MetroCluster health
