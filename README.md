## JNN/CPN Operator Field Guide (Practical Reference)

**Purpose:** This guide helps you operate the JNN Shelter and CPN Cases day-to-day and troubleshoot common problems quickly. **Always refer to official TMs for detailed procedures and safety.**

**I. The Big Picture: What Am I Looking At?**

*   **What it Does:** The JNN system creates a network bubble (like Wi-Fi and phone lines) for your unit, using secure (SIPRNET) and non-secure (NIPRNET) connections. It lets you talk, share data, and reach back to the main Army network.
*   **Main Parts:**
    *   **JNN Shelter:** The big brain/hub (usually at BDE/DIV). Contains the main routers, switches, servers, and crypto gear.
    *   **CPN Cases:** The user access points (at BN/BDE/DIV Command Posts). Connects your computers and phones back to the JNN shelter.
    *   **Transmission (STT/HCLOS):** The long-haul connection (Satellite dish or Radio tower) that links your JNN bubble to the outside world or other units. You mostly just need to ensure they are connected and powered.

**II. Setup & Power-Up Essentials**

1.  **Placement:** Choose stable, level ground. Ensure clear line-of-sight for STT dish or HCLOS antennas if used.
2.  **Power:**
    *   Connect **UPS cases** first. Verify correct voltage (120 VAC).
    *   Connect JNN shelter / CPN cases **to the UPS.**
    *   **Ground** all equipment properly.
3.  **Cabling (Key Connections):**
    *   **Users:** Ethernet cables from computers/phones to patch panels/switches in CPN cases.
    *   **CPN to JNN:** Usually **TFOCA fiber** between Signal Entry Panels (SEPs). Follow unit SOP/diagram.
    *   **JNN/CPN to STT/HCLOS:** Usually **TFOCA fiber or Coax** between SEPs and the STT/HCLOS unit.
4.  **Power On Sequence (General):**
    *   Turn on **UPS** units.
    *   Turn on **CPN case breakers/power** strips.
    *   Turn on **JNN shelter breakers/power** strips.
    *   Follow specific **STT/HCLOS power-up** sequence if applicable (check their TMs, allow warm-up time).
    *   Wait for equipment to boot (routers/switches take several minutes).

**III. Daily Checks & Quick Look**

*   **Power Lights:** Check UPS, Routers, Switches, TACLANEs, Modems - are power lights ON?
*   **Status Lights:**
    *   **Switches:** Link lights ON for connected user ports? Flashing indicates activity (good).
    *   **Routers:** Status lights generally solid green? (Varies by model).
    *   **TACLANEs:** RUN light steady/blinking green? INIT/ALARM lights OFF?
    *   **Modems (STT/LOS):** Power ON? Link/Sync lights indicating connection (check TM for specific lights)?
*   **Cooling:** Fans running? Airflow clear? Equipment not overheating?
*   **Cables:** Connections secure? No obvious damage?

**IV. Troubleshooting Common Problems (Start Here!)**

**General Steps:** Check the simple things first! Power -> Cables -> Lights -> Basic Commands -> Report Issue.

**(A) Symptom: User Can't Connect to Network (NIPR or SIPR)**

1.  **Check User End:**
    *   **Cable:** Is the Ethernet cable plugged in securely at the computer AND the CPN wall plate/patch panel? Try a different cable/port if possible.
    *   **PC:** Is the PC powered on? Does it have an IP address (usually automatic/DHCP)? (Ask user or S6 to check).
2.  **Check CPN Switch:**
    *   Find the switch port the user connects to (trace cable or use diagram).
    *   **Lights:** Is the link light for that port ON? Is it flashing (activity)? If OFF, check cable again. If cable is good, port may be bad or administratively down.
    *   **(Switch Console):** Log into the switch (`enable`).
        *   Check interface status: `show interface <type><number> status` (e.g., `show interface GigabitEthernet0/1 status`). Look for status like `connected` (good) vs. `notconnect` or `err-disabled`.
        *   If `err-disabled`, port security may have tripped. Report to NETOPS or follow unit SOP for `shutdown` then `no shutdown` on the interface (USE WITH CAUTION & AUTHORIZATION).
        *   Check VLAN assignment: `show vlan brief`. Ensure port is in the correct user VLAN.
        *   Check MAC address: `show mac address-table interface <type><number>`. See if the user's PC MAC address is learned.
3.  **Check Connectivity from Router (Use KVM/Console Cable):**
    *   Log into the main **Router** for that network (NIPR or SIPR) in the CPN or JNN.
    *   Type `enable` (enter password if needed).
    *   Check the router's connection to the switch: `show ip interface brief`
        *   Find the interface connecting to the user switch (e.g., `GigabitEthernet0/1`).
        *   Look at **Status** and **Protocol**. Both should say **"up"**.
        *   If Status is "down" or "admin down", it's a physical issue (check switch port/cable) or configuration problem (Report to NETOPS after checking switch).
        *   If Status is "up" but Protocol is "down", check for speed/duplex mismatch (`show interface <type><number>`). Also check the switch port status as above.
    *   Try to **ping** the user's gateway IP address (find this IP on the `show ip interface brief` output for the user VLAN/subnet interface): `ping <gateway_IP>`
        *   Success looks like `!!!!!`. Failure looks like `.....`.
        *   If gateway ping works but user ping fails, the issue is likely between the router and the user (Switch config/status, Cabling, PC). Double-check switch steps above.
4.  **Check Internal Routing (Router Console):**
    *   If pings to gateway fail or other internal resources are unreachable, check routing neighbor status: `show ip ospf neighbor`. Are expected neighbors listed in `FULL` state?
5.  **If Still Not Working:** After these checks, gather all info (port status, VLAN, ping results, OSPF status if checked) and report issue to NETOPS/Supervisor.

**(B) Symptom: Cannot Reach External Websites / Other Units (Slow or No Connection)**

1.  **Check Local Connectivity:** Can users reach local servers/printers? (If yes, issue is likely towards the outside).
2.  **Check Router Interfaces (Use KVM/Console Cable):**
    *   Log into the main **Tier 1 Router** (the one connecting towards STT/HCLOS).
    *   Type `enable`.
    *   Check the interface connecting to the transmission system (STT/HCLOS Modem/Mux): `show ip interface brief`
        *   Find the interface (e.g., `Serial0/0/0`, `GigabitEthernet0/0`).
        *   Status/Protocol should be **"up / up"**. If not, check cables, modem lights, and report to NETOPS.
3.  **Check Basic Routing (Router Console):**
    *   Ping the **next-hop router** (the IP address of the device on the *other* side of the STT/HCLOS link - get this from NETOPS/diagram): `ping <next_hop_IP>`
        *   If this fails, the issue is likely the transmission link itself or the device at the other end.
    *   Check the route table briefly: `show ip route`
        *   Look for a line starting with `S*` or `*`. This is the default route (0.0.0.0/0) pointing towards the outside world. Is it present and pointing to the correct next-hop IP? (Report issues to NETOPS).
    *   Trace the path: `traceroute <external_IP_address>` (e.g., `traceroute 8.8.8.8`)
        *   This shows the hops your traffic takes. Look for `* * *` (timeouts) which indicate where the connection might be failing.
4.  **Check Transmission System:**
    *   **STT:** Check modem status lights (Link, Sync, Rx/Tx Power levels, BER). Check ACU panel for satellite lock / pointing errors. Check for physical obstructions. Weather? Power cycle modem if SOP allows.
    *   **HCLOS:** Check radio/modem status lights (Link/Sync, Signal Strength/Quality, BER). Check antenna alignment visually. Weather? Power cycle modem/radio if SOP allows.
5.  **Check Firewall (If applicable):** Sometimes firewalls block traffic. Check status lights. (Firewall logs are usually checked by NETOPS).
6.  **Check TACLANE (If SIPRNET):** Is the TACLANE RUN light green? Check cables. (Detailed TACLANE troubleshooting requires HMI/Console access - See Section VI or report to NETOPS).
7.  **If Still Not Working:** Report issue to NETOPS/Supervisor. Provide details: What sites are unreachable, results of pings/traceroute, status of router interfaces and modem lights.

**(C) Symptom: VoIP Phone Not Working / Not Registering**

1.  **Check Phone:**
    *   **Power:** Is the phone screen on? Does it show an error message?
    *   **Cable:** Is the Ethernet cable plugged securely into the phone AND the CPN wall plate/switch?
2.  **Check Switch Port:**
    *   Find the switch port the phone connects to.
    *   **Lights:** Link light ON? PoE light ON (if phone uses Power over Ethernet)?
3. **Check Connectivity from Router (Use KVM/Console Cable):**
    *   Log into the Router managing voice (often the main SIPR Router).
    *   `enable`.
    *   Check the interface status for the voice VLAN: `show ip interface brief` (Status/Protocol should be up/up).
    *   Ping the phone's IP address (if known): `ping <phone_IP>`
    *   Ping the Call Manager server IP address (get from NETOPS/diagram): `ping <call_manager_IP>`
    *   Check routing to Call Manager: `show ip route <call_manager_IP>`
4.  **If Still Not Working:** After checking connectivity to phone and Call Manager, report issue to NETOPS/Supervisor. Call Manager issues are usually handled by NETOPS.

**(D) Symptom: TACLANE Tunnel Down / No SIPRNET Access (When NIPRNET Works)**

1.  **Check TACLANE:**
    *   **Lights:** Power light ON? RUN light **solid or blinking GREEN**? INIT/ALARM lights OFF? If ALARM is RED, report immediately.
    *   **Cables:** Secure connections on Plain Text (PT/Black) and Cipher Text (CT/Red) ports?
2.  **Check Connected Router Interfaces (Use KVM/Console Cable):**
    *   Log into the routers connected to **both** sides of the TACLANE (PT and CT sides).
    *   `enable`.
    *   Check interface status: `show ip interface brief`
        *   Interfaces connected to TACLANE PT and CT ports should be **"up / up"**. If not, check cables/TACLANE ports/connected switch ports.
    *   **(Router Console):** Check crypto status (relevant for the CT side router, may also apply to PT if part of IPsec tunnel):
        *   `show crypto isakmp sa`: Look for QM_IDLE.
        *   `show crypto ipsec sa`: Look for encrypts/decrypts increasing.
3.  **Basic TACLANE Check (Console - Requires HMI Cable):**
    *   Connect console cable to TACLANE HMI port.
    *   Check Status menus: Key status (should be loaded/valid)? Tunnel status? Interface status? (Refer to TACLANE TM for details).
4.  **If Still Not Working:** After checking lights, router interfaces, basic crypto status, and TACLANE HMI status, **report to NETOPS/COMSEC Custodian immediately.** Provide all gathered info.

**(E) Symptom: Specific Port Seems Blocked (But Others Work)**

1.  **Identify the Port:** Note the physical port number on the switch/patch panel.
2.  **Check Switch Configuration (Requires Console Access & Guidance):**
    *   This often involves checking for specific switch features like **Port Security** (which might disable a port if an unauthorized device connects) or VLAN assignments.
    *   Log into the switch: `enable`
    *   Check detailed interface status: `show interface <type><number> status`. Look for `connected`, `notconnect`, `err-disabled`.
    *   Check port security status: `show port-security interface <type><number>`. See if security action is `Shutdown`.
    *   Check VLAN assignment: `show vlan brief`. Ensure port is in correct VLAN.
    *   Check MAC address table: `show mac address-table interface <type><number>`.
    *   **Action:** If port is `err-disabled` due to port-security, **follow unit SOP/get authorization** before attempting recovery (usually `shutdown` then `no shutdown` on the interface). Otherwise, note status (VLAN, MAC, etc.) and **Report to NETOPS**.
3.  **Check Firewall Rules (NETOPS Task):** Inform NETOPS that a specific port/service seems blocked; they will check the firewall rules.

**(F) Symptom: Intermittent or No Connection to STT/HCLOS Unit**

1.  **Physical Checks First:**
    *   **Cables:** Check the main cable (Fiber or Coax) between the JNN/CPN SEP and the STT/HCLOS unit. Ensure secure connection at both ends. Check for visible damage.
    *   **Power:** Verify power to the STT/HCLOS unit and its modem/radio components.
2.  **Check JNN/CPN Router Interface (Use KVM/Console Cable):**
    *   Log into the **Tier 1 Router** (the one connecting to STT/HCLOS).
    *   `enable`.
    *   Check the specific interface: `show ip interface brief`
        *   Find the interface (e.g., `Serial0/0/0`, `GigabitEthernet0/0`).
        *   Is Status/Protocol consistently **"up / up"**? If it flaps (`up/down`), check `show interface <type><number>` for errors (CRC, input/output, resets, carrier transitions). Note the error types and frequency.
        *   If it's down, proceed to check the transmission system.
    *   Use `show interface <type><number>` to check for errors.
        *   Look for increasing `input errors`, `CRC`, `output errors`, `resets`, `carrier transitions`. High or rapidly increasing errors indicate a problem with the link or hardware. Note specific error types.
    *   If Serial interface, check physical layer: `show controllers <type><number>`. Look for DCD/DSR/CTS signals, clocking status, framing errors.
3.  **Check Transmission System Status Lights:**
    *   **STT:** Check modem status lights (Link, Sync, Rx/Tx Power levels, BER). Check ACU panel for satellite lock / pointing errors. Check for physical obstructions. Weather? Power cycle modem if SOP allows.
    *   **HCLOS:** Check radio/modem status lights (Link/Sync, Signal Strength/Quality, BER). Check antenna alignment visually. Weather? Power cycle modem/radio if SOP allows.
4.  **Check External Factors:**
    *   **STT:** Heavy rain/snow (rain fade)? Physical obstruction (new tent, vehicle)? Has the dish been bumped?
    *   **HCLOS:** Physical obstruction (trees, buildings)? Heavy fog/rain? Has the antenna been bumped?
5.  **Check Internal Routing (Router Console):**
    *   If pings to gateway fail or other internal resources are unreachable, check routing neighbor status: `show ip ospf neighbor`. Are expected neighbors listed in `FULL` state?
5.  **If Still Not Working or Intermittent:** After checking router interface status/errors/controllers, modem/radio lights, and external factors, report issue to NETOPS/Supervisor. Provide detailed findings.

**V. Essential Cisco Commands (Quick Reference)**

*   **Getting In:**
    *   Connect via Console Cable (PuTTY/TeraTerm: 9600, 8, N, 1, No Flow Control) or SSH.
    *   `enable` (Enter Privileged EXEC mode - may need password).
*   **Checking Status (USE THESE MOST):**
    *   `show ip interface brief` or `sh ip int br`: Shows all interfaces, their IP address, and crucially, their **Status** (physical layer - cable plugged in?) and **Protocol** (data link layer - talking correctly?). **"up / up" is GOOD.** "down / down" or "admin down" means check cables or port config. "up / down" often means config mismatch (e.g., speed/duplex) or problem on the other end.
    *   `ping <ip_address>`: Sends test packets. `!!!!!` = Success. `.....` = Failure/Timeout. `U.U.U` = Unreachable.
    *   `traceroute <ip_address>` (or `tracert`): Shows the path (router hops) packets take. Good for finding *where* a connection fails (look for `* * *` timeouts).
*   **Looking Deeper (More Advanced Checks):**
    *   `show interface <type><number>` (e.g., `show interface GigabitEthernet0/1`): Detailed stats, **errors (CRC, input/output, resets, carrier transitions)**, speed/duplex for one interface.
    *   `show controllers <type><number>` (e.g., `show controllers Serial0/0/0`): Shows hardware-level status for WAN interfaces (check clocking, framing, signals like DCD/DSR/CTS).
    *   `show ip route`: Shows the routing table. Look for default route (`S*`) and specific routes.
    *   `show ip ospf neighbor`: Check status of OSPF routing neighbors (should be `FULL`).
    *   `show crypto isakmp sa`: Check ISAKMP (Phase 1) VPN/Crypto status (look for `QM_IDLE`).
    *   `show crypto ipsec sa`: Check IPsec (Phase 2) VPN/Crypto status (look for increasing encrypts/decrypts).
    *   **(Switch):** `show vlan brief`: Check VLAN assignments for ports.
    *   **(Switch):** `show port-security interface <type><number>`: Check port security status (violations, status like `err-disabled`).
    *   **(Switch):** `show mac address-table interface <type><number>`: See MAC addresses learned on a port.
    *   `show running-config` or `show run`: Shows the current live configuration.
*   **Getting Out:**
    *   `exit`: Move back one command level.
    *   `end` or `Ctrl+Z`: Go straight back to `enable` mode from config mode.
    *   `disable`: Exit `enable` mode.

**VI. Reporting Issues**

*   When troubleshooting steps indicate configuration issues, hardware failures, crypto problems, or if you've exhausted the checks in this guide, **report the issue to NETOPS/S6/Supervisor.**
*   Provide clear info: What symptom, what unit/location, **ALL troubleshooting steps taken (including command outputs if possible)**, and what results you saw.