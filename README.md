
# Seaweed: Iodine DNS Tunneling (Red Team Training Use Only)

> ⚠️ **For Authorized Red Team / SOC Training Only**  
> This setup is intended strictly for internal cybersecurity education, detection engineering, and red/blue team simulations. It must not be used for unauthorized access, evasion, or malicious behavior. All testing must occur within lab environments or under written authorization.

---

## 🔍 What Is This?

This project demonstrates how to establish a **covert DNS tunnel using [Iodine](https://github.com/yarrick/iodine)**. DNS tunneling allows you to move data between systems even when other outbound channels (e.g., HTTP, SSH) are blocked by perimeter firewalls. 

This is a **realistic attacker TTP (MITRE T1071.004)** and can be used to:
- Simulate exfiltration through egress-filtered networks
- Test SOC/EDR detection rules
- Train blue teams on DNS anomaly response
- Demonstrate full C2 over DNS in highly restricted environments

---

## ⚙️ What You'll Need

- ✅ A registered domain (we use `iodonetesting.com`)
- ✅ AWS Route 53 for DNS management
- ✅ A public-facing Linux server (e.g., AWS EC2) to run Iodine
- ✅ A client machine behind DNS-only egress (or test NATed environments)
- ✅ Debian or Ubuntu (or any Linux distro with Iodine support)

---

## 🏗️ Step-by-Step Setup

### 🔹 Step 1: Register a Domain

Use a registrar like GoDaddy, Namecheap, etc.  
**Example Domain**: `iodonetesting.com`

---

### 🔹 Step 2: Delegate Domain to Route 53

1. Open AWS Console → **Route 53**
2. Create a **Hosted Zone** for `iodonetesting.com`
3. Copy the 4 AWS nameservers listed under `NS`
4. Update your registrar (GoDaddy, etc.) to use those nameservers

Wait for DNS propagation (use `dig` to check):
```bash
dig NS iodonetesting.com
```

---

### 🔹 Step 3: Create a Subdomain for DNS Tunneling

1. In Route 53, create a **second public hosted zone** for:
   ```
   tunnel.iodonetesting.com
   ```
2. Copy its NS values (e.g., `ns-xxx.awsdns-xx.com.`)

3. Go back to the **main zone (`iodonetesting.com`)**
   - Create an `NS` record:
     - **Name**: `tunnel`
     - **Type**: `NS`
     - **Value**: paste the 4 NS records from the `tunnel` zone

---

### 🔹 Step 4: Point Your Server Hostname

1. In the `tunnel.iodonetesting.com` hosted zone, create an A record:

   - **Name**: `iodine`
   - **Type**: `A`
   - **Value**: your server’s public IP

2. Confirm DNS resolution from client machine:

```bash
dig iodine.tunnel.iodonetesting.com @1.1.1.1
```

---

### 🔹 Step 5: Install Iodine

**On the server (e.g., Sepsis):**
```bash
sudo apt update
sudo apt install iodine -y

sudo iodined -f -P hunter2 10.0.0.1 tunnel.iodonetesting.com
```

**On the client:**
```bash
sudo apt update
sudo apt install iodine -y

sudo iodine -f -P hunter2 -T A -r iodine.tunnel.iodonetesting.com tunnel.iodonetesting.com
```

---

## ✅ Tunnel should now be live.

You should see the following:
- `dns0` interface assigned `10.0.0.2`
- Server tunnel IP: `10.0.0.1`
- Codec negotiation completed (Base128)
- Connection setup complete

---

Next steps (e.g., ping, SSH, exfil) will be added in future updates.
