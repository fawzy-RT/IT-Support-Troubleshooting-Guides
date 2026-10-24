# 🌐 Network Troubleshooting Guide

A practical Windows networking troubleshooting guide covering common IT support problems, useful commands, possible causes, and step-by-step solutions. Don't worry if you got network problems 😎

I made this guide to help with common network problems you might face in a real company or in everyday life. Most of these I dealt with myself at work, and a few of them at home too

## Table of Contents

1. Internet Not Working
2. Connected to Wi-Fi but No Internet
3. Invalid IP Address (APIPA)
4. DNS Not Working
5. IP Address Conflict
6. One Device Cannot Connect
7. Multiple Devices Cannot Connect
8. Slow Internet or Network
9. Ethernet Not Working
10. Wi-Fi Network Not Showing

---

## 1. Internet Not Working

A user reports that websites are not opening. The computer appears to be connected, but there is no internet access.

**Possible Causes (most common first):**
- Network cable disconnected or Wi-Fi issue
- Network adapter disabled
- Incorrect IP configuration
- DHCP or gateway issue
- DNS resolution problem
- Router, firewall, or ISP outage

**Diagnosis**

First thing I check is the network configuration:

```
ipconfig
```

I usually go with `ipconfig /all` when I need more detail — DHCP server, DNS, MAC address, lease time, all in one view.

After that, I test the local network:

```
ping <default-gateway>
ping <default-gateway> -t
```

The `-t` keeps pinging until you stop it with Ctrl + C. I use it a lot when the connection is flaky and I want to see if it drops over time.

Then I test internet connectivity:

```
ping 1.1.1.1
```

If 1.1.1.1 replies but websites still don't open, it's almost always a DNS problem. I check with:

```
nslookup google.com
```

**Solution**

- Cable or adapter issue → reconnect the cable or enable the adapter (if authorized).
- Incorrect IP config → investigate DHCP and network settings.
- Gateway unreachable → check the local connection, switch port, and VLAN.
- IP works but DNS fails → investigate the DNS config or server.
- Multiple devices affected → it's probably infrastructure, escalate.

**Verification**

Run the same tests again, open the website that originally failed, and write down what worked.

---

## 2. Connected to Wi-Fi but No Internet

Windows shows the computer connected to Wi-Fi, but websites and apps won't load.

**Possible Causes (most common first):**
- Captive portal requires sign-in
- DHCP or gateway issue
- DNS server unavailable
- Weak Wi-Fi signal or interference
- VPN or proxy misconfiguration
- Access point, router, or ISP outage

**Diagnosis**

First, make sure the computer is on the correct Wi-Fi network. Users sometimes connect to a guest network by mistake and wonder why nothing works.

Then check if other devices on the same Wi-Fi have internet. If they do, the issue is on this computer. If they don't, the issue is on the network side.

```
ipconfig /all
ping <default-gateway>
nslookup example.com
```

Also check if there's a sign-in page, VPN, or proxy blocking access. Captive portals catch me off guard sometimes — the icon says connected, but nothing loads until you sign in.

**Solution**

- Captive portal → complete the sign-in.
- Wrong SSID → reconnect to the correct network.
- DHCP or gateway issue → investigate the network config.
- DNS issue → check the configured DNS server.
- AP or ISP issue → report it to the network team.

**Verification**

Confirm the computer has valid network settings and can reach the websites and apps the user needs.

---

## 3. Invalid IP Address (APIPA)

A computer gets an IP like 169.254.10.25 instead of one from the company network.

Windows assigns an APIPA address when it can't get a valid IPv4 from DHCP. Whenever I see 169.254.x.x, I immediately think DHCP problem.

**Possible Causes (most common first):**
- DHCP server unavailable
- DHCP scope exhausted
- Ethernet or Wi-Fi connection issue
- Incorrect VLAN configuration
- DHCP traffic blocked somewhere on the network

**Diagnosis**

```
ipconfig /all
```

Look for an IPv4 starting with 169.254. That's the APIPA range.

Then try:

```
ipconfig /renew
```

If it comes back with a valid IP, the DHCP server was probably just slow or the lease had expired. If it still comes back as APIPA, the problem is deeper — DHCP server down, scope full, or DHCP traffic being blocked.

**Solution**

- Check the cable or Wi-Fi connection.
- Make sure the adapter is enabled.
- Investigate DHCP availability and scope capacity.
- Ask the network admin to check VLAN or DHCP relay settings if needed.

Don't assign a random static IP just to bypass the problem. The address has to match the approved network config, otherwise you'll just create new problems.

**Verification**

Run `ipconfig /all` again. Make sure the computer got the expected IPv4, gateway, and DNS. Then test access to network resources.

---

## 4. DNS Not Working

The computer has network connectivity, but websites or internal services can't be reached by hostname. Accessing something by IP works, but by name it fails.

**Possible Causes (most common first):**
- Incorrect DNS server configuration
- DNS server unavailable or unreachable
- Stale local DNS cache
- Missing or incorrect DNS record
- Required VPN disconnected

**Diagnosis**

First, check the configured DNS servers:

```
ipconfig /all
```

Then test hostname resolution:

```
nslookup example.com
```

See which DNS server responds and whether the name actually resolves. If the server doesn't respond, you have a connectivity issue. If it responds but can't resolve the name, it's a DNS server or record issue.

If everything looks correct but it still fails, flush the local cache:

```
ipconfig /flushdns
```

Then try again. The amount of times this alone fixed the problem is honestly surprising.

**Solution**

- Wrong DNS settings → fix them according to the approved network config.
- DNS server unreachable → investigate connectivity to the server.
- Wrong DNS record → ask the responsible admin to check it.
- VPN required → reconnect to the approved VPN.
- Stale cache → flush and retry.

**Verification**

Run `nslookup` again and confirm the name resolves. Test the original website or app.

---

## 5. IP Address Conflict

Windows reports an IP address conflict, or a device keeps losing network connectivity.

This happens when two devices are using the same IP. Common scenario: someone sets a static IP on their PC without telling anyone, and it clashes with a DHCP reservation.

**Possible Causes (most common first):**
- Two devices assigned the same static IP
- Static IP overlapping with DHCP scope
- Incorrect DHCP reservation
- Misconfigured network settings

**Diagnosis**

```
ipconfig /all
getmac /v
```

Then use authorized DHCP management or IPAM tools to review leases, reservations, and conflicting assignments. If you don't have access, escalate to whoever does.

**Solution**

- Identify the two devices in conflict.
- Check DHCP reservations and static IP config.
- Fix the duplicate assignment in the approved config.
- Escalate if you need access to DHCP or IPAM tools.

Don't randomly change IPs on managed devices. You'll just create another conflict somewhere else.

**Verification**

Make sure the device has a unique, approved IP and the connection stays stable.

---

## 6. One Device Cannot Connect

One computer can't access the network, while nearby computers work normally.

When only one device is affected, the problem is almost always on that device — not on the network.

**Possible Causes (most common first):**
- Network adapter disabled
- Incorrect IP or DNS settings
- Faulty cable or Wi-Fi connection
- Driver problem
- VPN, proxy, or firewall restriction
- Device-specific security policy

**Diagnosis**

First, check the physical connection or Wi-Fi status. Then open Device Manager and look at the network adapter:

```
devmgmt.msc
```

After that, the usual checks:

```
ipconfig /all
ping <default-gateway>
nslookup example.com
```

Compare the results with a working device on the same network if you can. That usually tells you exactly what's different.

**Solution**

- Connection issue → reconnect the cable or Wi-Fi.
- Incorrect config → fix it according to network requirements.
- Driver issue → follow the approved driver repair process.
- VPN, proxy, or security restriction → investigate the relevant setting or policy.
- Still stuck → collect the test results and escalate.

**Verification**

Confirm the device can reach the resource or app that originally failed.

---

## 7. Multiple Devices Cannot Connect

Several users report network issues at the same time.

When multiple devices are affected, the issue is usually on the shared infrastructure — not on individual computers. This is the kind of thing that gets everyone calling you at once.

**Possible Causes (most common first):**
- Router or gateway failure
- Switch or access point outage
- DHCP or DNS service failure
- VLAN or routing issue
- ISP outage
- Recent network config change

**Diagnosis**

First, figure out who's affected and where. Is it one floor? One building? Everyone?

Then check if it affects both wired and wireless. If both are down, it's probably upstream. If only Wi-Fi is down, it's probably the access points.

Test the gateway from an authorized device:

```
ping <default-gateway>
```

Check monitoring tools and incident history if available. Look for recent changes — someone applying a "quick fix" to the firewall is a classic cause.

**Solution**

- Follow the org's incident-handling process.
- Collect details about affected users, locations, and services.
- Escalate infrastructure, DHCP, DNS, routing, or ISP issues to the responsible team.
- Don't make production network changes without authorization.

**Verification**

Test representative devices from affected areas and confirm services are back. Keep monitoring for a while — sometimes the issue comes back.

---

## 8. Slow Internet or Network

Websites and apps open, but loading takes longer than usual. Performance is inconsistent.

Slow is harder to troubleshoot than "doesn't work". You need to compare and measure.

**Possible Causes (most common first):**
- Weak Wi-Fi signal or interference
- Network congestion
- High latency or packet loss
- DNS lookup delays
- VPN overhead
- Slow application or computer performance
- Upstream or ISP problem

**Diagnosis**

Check for packet loss or delays to the gateway:

```
ping -n 20 <default-gateway>
```

Look at the response times and whether any packets are lost. A few ms is normal. 100ms+ or dropped packets means something's wrong.

Then check the route:

```
tracert example.com
```

This shows each hop toward the destination. Some routers don't respond to these probes, so an incomplete trace doesn't automatically mean there's a fault.

Compare wired vs wireless if possible. If wired is fast and Wi-Fi is slow, it's a Wi-Fi issue. If both are slow, it's upstream.

**Solution**

- Weak Wi-Fi → improve the connection or move closer to the AP.
- Confirmed congestion or packet loss → report the evidence to the network team.
- DNS delays → investigate the DNS config and response times.
- VPN-related → check the approved VPN config.
- Application-specific → investigate the app or contact its support.

**Verification**

Repeat the tests and check if the affected websites or apps perform normally.

---

## 9. Ethernet Not Working

A desktop connected through Ethernet can't establish a working connection.

Wired connections are usually more reliable than Wi-Fi, so when they fail, it's often a physical issue.

**Possible Causes (most common first):**
- Damaged or disconnected Ethernet cable
- Faulty wall port
- Network adapter disabled
- Driver problem
- Switch port or VLAN configuration issue
- DHCP failure

**Diagnosis**

Check both ends of the cable first. Then look for link or activity lights on the network adapter and switch — no lights usually means no link.

Test with a known-good cable or port if authorized. This tells you immediately if the cable is the problem.

```
devmgmt.msc
ipconfig /all
```

Check if the adapter shows up correctly and if it has a valid IP.

**Solution**

- Damaged cable → replace with a known-good one.
- Disabled adapter → enable it if authorized.
- Driver issue → follow the approved driver repair process.
- Switch port or VLAN problem → escalate to the network admin.
- DHCP issue → investigate the address assignment.

**Verification**

Confirm the Ethernet link is active, the computer receives the expected config, and required resources are reachable.

---

## 10. Wi-Fi Network Not Showing

A computer can't find an expected wireless network in the available list.

**Possible Causes (most common first):**
- Wi-Fi is disabled
- Airplane mode is on
- Wireless adapter missing or malfunctioning
- Driver issue
- Weak signal or distance from the AP
- Unsupported Wi-Fi band or channel
- Access point or SSID config problem

**Diagnosis**

First, confirm Wi-Fi is enabled and Airplane mode is off. You'd be surprised how often this is the answer.

Then check if nearby devices can see the same SSID. If they can, the issue is on this computer. If they can't, the issue is on the AP side.

```
devmgmt.msc
```

Check the wireless adapter and driver status. Move closer to the access point and refresh the network list — sometimes the signal is just too weak.

**Solution**

- Wi-Fi disabled → enable it.
- Adapter or driver issue → investigate and repair the confirmed problem.
- Compatibility issue → check if the adapter supports the configured band or channel.
- AP or SSID problem → escalate to the network admin.

**Verification**

Confirm the expected SSID shows up and the computer connects with the approved credentials and security settings.

---

## Essential Windows Networking Commands

These are the commands I use most when checking network problems.

| Command | Purpose |
|---------|---------|
| `ipconfig /all` | Detailed IP configuration |
| `ipconfig /renew` | Requests a DHCP lease |
| `ipconfig /release` | Releases the current DHCP lease |
| `ipconfig /flushdns` | Clears the local DNS cache |
| `ping <destination>` | Tests ICMP reachability |
| `nslookup <hostname>` | Checks DNS resolution |
| `tracert <hostname>` | Shows the route toward a destination |
| `getmac /v` | Displays MAC addresses and adapter info |

**Important:** A failed ping doesn't always mean the destination is unreachable. Firewalls and network devices may block ICMP, so always consider the other test results before deciding where the problem is.

---

## General Troubleshooting Checklist

Use this when working through a network issue.

- Identify the exact problem and what the user was trying to do.
- Determine if one device or multiple devices are affected.
- Collect useful information before changing settings.
- Check physical connectivity and adapter status.
- Inspect the IP address, gateway, and DNS config.
- Run relevant tests and compare the results.
- Change one setting at a time so the result is easier to understand.
- Test the original problem again after applying a fix.
- Record the confirmed cause, solution, and verification.
- Remove passwords, internal addresses, and sensitive details before publishing.

---

## Project Status

This guide is a work in progress. I'm adding examples and improving the steps as I learn and practice. Commands and procedures should be tested in an appropriate environment before being marked as verified.
