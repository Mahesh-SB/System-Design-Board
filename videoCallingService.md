## HLD
![HLD-Diagram](https://github.com/user-attachments/assets/b1484efb-981d-4cb0-8280-8c21e402d7c6)

###
🚧 What Is the NAT Traversal Issue ?  
NAT (Network Address Translation) is used by routers to allow multiple devices in a private network (like your home Wi-Fi) to share a single public IP address. While this improves security and conserves IPs, it creates problems for peer-to-peer (P2P) communication, like video calls.

🧩 The Problem:  
When two users behind NATs try to connect directly (P2P), each user only knows their private IP—not helpful over the internet. Since NAT hides internal IPs, the devices can’t figure out how to reach each other directly.

Imagine:  
User A is behind a home Wi-Fi NAT. User B is on mobile data behind a carrier-grade NAT. How can A and B connect directly? They don't know how to route packets to each other.    

🛠️ Solution: NAT Traversal    
To solve this, NAT traversal techniques are used:  
1. STUN (Session Traversal Utilities for NAT)  
Helps each peer discover their public IP and port.  
Works if both sides are behind simple NATs.  
If successful, the call is direct P2P.   
2. TURN (Traversal Using Relays around NAT) 
Used when STUN fails (e.g., in symmetric NATs or corporate firewalls).  
A TURN server relays the media traffic between peers.  
Adds latency and cost, but ensures connectivity.  
3. ICE (Interactive Connectivity Establishment)  
A framework that uses both STUN and TURN.  

Tries all possible connection methods (host, reflexive, relayed) and picks the best path.

🔍 Why It Matters in Video Calling
Latency-sensitive: You want the fastest possible connection.
Unpredictable networks: Mobile, corporate, public Wi-Fi – all behave differently.
Fallback mechanisms are crucial for reliability.

✅ Summary
NAT traversal is about helping two devices behind firewalls/NATs find a way to talk directly or fall back to relaying via TURN servers. Without it, video calls would often fail or degrade.
