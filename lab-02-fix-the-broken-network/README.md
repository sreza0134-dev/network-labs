# Lab 2: Fix the Broken Network

I took the office network from Lab 1 and broke five things on purpose. For each one I looked at the problem the way a user would see it, found the cause with commands, fixed it, and checked that it worked. This is the same work as a helpdesk ticket.

The network is the one from Lab 1.

![Topology](../lab-01-vlan-office-network/screenshots/01-topology.png)

## The five faults

| # | What the user saw | Cause | Fix |
|---|---|---|---|
| 1 | The internet works, but the office server does not | PC1's switch port was in the Guest VLAN | Moved the port back to the Staff VLAN |
| 2 | A PC can talk to its neighbour, but nothing else | Wrong default gateway on PC2 | Set the gateway to 192.168.1.1 |
| 3 | A PC has no address at all | The Staff DHCP pool was missing | Created the pool again |
| 4 | Guests have no internet | The last rule of the guest access list said deny | Changed it to permit |
| 5 | Pinging an address works, but websites by name do not | DHCP gave out a wrong DNS server | Put the correct DNS server in the pool |

## Fault 1: Staff PC cannot reach the server

**What the user saw:** PC1 can reach the internet, but not the office server.

**How I found the cause:** `ipconfig` showed the address 192.168.1.76 and the gateway 192.168.1.65. Those belong to the Guest network. A Staff PC should have an address between 192.168.1.1 and 192.168.1.62. Then I ran `show vlan brief` on SW2. Port Fa0/1, the port for PC1, was in VLAN 20 (Guest).

**Fix:** I put the port back in VLAN 10 and made PC1 ask for a new address. It got 192.168.1.11 and could reach the server again.

![Symptom](screenshots/01-fault1-symptom.png)
![Cause](screenshots/02-fault1-cause.png)
![Fixed VLAN](screenshots/03-fault1-fixed-vlan.png)
![Fixed PC1](screenshots/04-fault1-fixed-pc1.png)

## Fault 2: Wrong gateway

**What the user saw:** PC2 can ping PC1, but not the server and not the internet.

**How I found the cause:** PC1 is in the same network, so the ping did not need the router. The server and the internet need the router, and the router is reached through the gateway. PC2's gateway was 192.168.1.2. A ping to 192.168.1.1 (the real router) worked, and a ping to 192.168.1.2 did not.

**Fix:** I changed the gateway to 192.168.1.1. After that the server and the internet both worked.

![Symptom](screenshots/05-fault2-symptom.png)
![Cause](screenshots/06-fault2-cause.png)
![Fixed settings](screenshots/07-fault2-fixed-ip.png)
![Fixed ping](screenshots/08-fault2-fixed-ping.png)

## Fault 3: No address from DHCP

**What the user saw:** PC1 has the address 0.0.0.0 and cannot reach anything. In Packet Tracer a PC with no answer shows 0.0.0.0. A real Windows PC would show an address starting with 169.254.

**How I found the cause:** I ran `show ip dhcp pool` on the router. Only the GUEST pool was in the list. The STAFF pool was gone, so nobody answered the Staff devices.

**Fix:** I created the STAFF pool again with the same settings. PC1 asked again and got 192.168.1.11.

![Symptom](screenshots/09-fault3-symptom.png)
![Cause](screenshots/10-fault3-cause.png)
![Fixed pool](screenshots/11-fault3-fixed-pool.png)
![Fixed PC1](screenshots/12-fault3-fixed-pc1.png)

## Fault 4: Guests have no internet

**What the user saw:** The guest laptop is connected to the Wi-Fi, but a ping to the internet fails with "Destination host unreachable" from 192.168.1.65. That is the router's own address, so the router was refusing the traffic.

**How I found the cause:** I ran `show access-lists` on the router. The last rule of the guest list said `deny ip any any`, and its counter was going up. It should say `permit`.

**Fix:** I changed the rule to `permit ip any any`. Then I checked two things. The internet worked, and the server was still blocked for guests, as it should be.

**A second problem:** After the fix, the internet still did not work, and the rule counter did not move. That meant the traffic never reached the router. The laptop had lost its address and showed 0.0.0.0, because its settings had been switched to a fixed address. I set it back to DHCP, and then the internet worked.

![Rule broken](screenshots/13-fault4-acl-broken.png)
![Symptom](screenshots/14-fault4-symptom.png)
![Cause](screenshots/15-fault4-cause.png)
![Fixed rule](screenshots/16-fault4-fixed-acl.png)
![Fixed ping](screenshots/17-fault4-fixed-ping.png)

## Fault 5: Websites do not load by name

**What the user saw:** Pinging 203.0.113.2 works, but `nslookup www.example.com` fails.

**How I found the cause:** The ping by number worked, so the network was fine. The problem was the name lookup. `nslookup` showed that PC1 was asking 192.168.1.99, and the answer timed out. A ping to 192.168.1.99 got no reply, because no device has that address. When I asked the real DNS server at 192.168.1.130 directly, it answered correctly.

**Fix:** The wrong address came from the DHCP pool. I put the correct DNS server, 192.168.1.130, in the STAFF pool. PC1 asked again, and the name lookup worked.

![Symptom](screenshots/18-fault5-symptom.png)
![Cause](screenshots/19-fault5-cause.png)
![Fixed pool](screenshots/20-fault5-fixed-pool.png)
![Fixed PC1](screenshots/21-fault5-fixed-pc1.png)

## How I look for a problem

1. Ask what the user sees, and what changed.
2. Check the address with `ipconfig`.
3. Ping the gateway, then a device in another network, then the internet. This shows where it stops.
4. If the ping by number works but the name does not, look at DNS.
5. Look at the router and the switch to find the setting that is wrong.
6. Fix one thing, and test again.

## What I learned

- A wrong address often points to a wrong VLAN or to DHCP.
- A wrong gateway means a PC can only talk inside its own network.
- Ping by number and ping by name tell you if the problem is the network or DNS.
- The counters in an access list show which rule caught the traffic. If a rule's counter does not move, the traffic is not reaching it, so look at an earlier step.
- After a fix, test the next layer too. In fault 4 the first fix was right, but a second problem was hiding behind it.

## Files

- `fixed-network.pkt` is the Packet Tracer file after all five fixes. The healthy network before I broke it is in Lab 1.
- `screenshots/` has the pictures used above.

## Notes

- I made all five faults myself, in the simulator, so the problems are practice problems and not real tickets.
- This was built in Cisco Packet Tracer, not on real equipment.
