# Pre-Flight Guide

Read and understand these concepts before you write any code.

---

## 1. The IP Problem & CGNAT

The internet ran out of IPv4 addresses. Today, ISPs put entire neighborhoods behind one giant router. This is called **Carrier-Grade NAT (CGNAT)**. 

Because of this, your home router does not have a real public IP address. It has a private one. This means other computers on the internet cannot easily start a direct connection with you.

---

## 2. STUN: Finding Your Internet Identity

**STUN** is a protocol that answers one question: *"What do I look like to the internet?"*

1. Your code sends a UDP packet to a STUN server (like `stun.l.google.com:19302`).
2. The server replies with your **Public IP and Port**. 
3. This is the address your team needs to reach you.

**The Golden Rule:** You must keep the **same UDP socket open**. If you close the socket and open a new one, your router will give you a different port, and the connection will fail.

---

## 3. UDP Hole Punching (The Strategy)

Your router's firewall blocks incoming traffic. You must trick it into opening a "hole."

1. **Find IPs:** Node A and Node B use STUN to find their public IPs and ports.
2. **Share:** They share these IPs with each other in the team chat.
3. **Punch:** At the *exact same time*, Node A sends a packet to Node B, and Node B sends a packet to Node A. 
4. **Connect:** Both firewalls see an outgoing packet and open a hole expecting a reply. The packets cross the internet, enter the holes, and a direct P2P connection is made.

* **Learn more:** [NAT Traversal Explained](https://tailscale.com/blog/how-nat-traversal-works)

---

## 4. Docker & Docker Compose

A Docker container is not a magic box; it is an isolated Linux process. 

Your container has its own private network (`localhost` inside the container is NOT your computer's `localhost`). For your UDP packets to reach the internet, you must explicitly tell Docker how to map your host machine's port to the container's port.

**Docker Compose** makes this easier. Instead of writing a massive `docker run` command with all your port mappings and environment variables every time you test your code, you define the rules once in a `docker-compose.yml` file.

### Important Commands for this Mission
Because your Python script requires you to type things in the terminal (using `input()`), you must start the container in the background and then connect your keyboard to it.

* `docker-compose up -d --build` : Builds your code and starts the container in the background (detached mode).
* `docker ps` : Lists all running containers. Use this to find your exact container name.
* `docker attach <container_name>` : Connects your terminal to the running Python process inside the container. **You must do this to type your targets and send messages!** * `docker-compose down` : Stops and cleans up the container when you are done.

* **Watch this:** [Docker in 100 Seconds](https://youtu.be/Gjnup-PuquQ)
* **Watch this:** [How Docker Namespaces Work](https://youtu.be/-YnMr1lj4Z8)

---

## 5. Know Your NAT Type

Different routers handle NAT differently. If you have a "Symmetric" NAT, hole punching is almost mathematically impossible. 

| NAT Type | Hole Punching? | Notes |
|----------|--------------|-------|
| Full Cone | Yes | Rare today |
| Restricted Cone | Yes | Standard home routers |
| Symmetric | **NO** | Common on Cellular 4G/5G and Corporate networks |
| CGNAT | Maybe | Adds an extra layer of difficulty |

* **Learn more:** [NAT types and port punching](https://support.dh2i.com/docs/Archive/kbs/general/understanding-different-nat-types-and-hole-punching/)

---

## Fallback Options

If your router completely blocks UDP hole punching, you must pivot. Document why you failed and try one of these alternatives:

* **Option A:** Run Docker with `--network host` to remove Docker's extra network layer.
* **Option B:** Use SSH Remote Port Forwarding through a public VPS server.
* **Option C:** Use an overlay network like Tailscale or ZeroTier to bypass the firewalls.