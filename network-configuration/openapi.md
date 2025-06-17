---
icon: network-wired
---

# WireGuard Configuration

#### **Step 1: Install WireGuard on All Nodes**

**Repeat this step on `node1`, `node2`, and `node3`.**

1.  **Update package lists:**

    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```
2.  **Install WireGuard tools:**

    ```bash
    sudo apt install wireguard -y
    ```

***

#### **Step 2: Generate Keys for All Nodes**

**Repeat this step on `node1`, `node2`, and `node3`.** Each node needs a unique private and public key pair.

1.  **Generate private key:**

    ```bash
    wg genkey | sudo tee /etc/wireguard/privatekey
    ```
2.  **Generate public key from private key:**

    ```bash
    sudo cat /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
    ```
3.  **Set correct permissions for the private key:**

    ```bash
    sudo chmod 600 /etc/wireguard/privatekey
    ```
4.  **Display keys (write these down for each node!):**

    ```bash
    echo "Private Key: $(sudo cat /etc/wireguard/privatekey)"
    echo "Public Key: $(sudo cat /etc/wireguard/publickey)"
    ```

***

#### **Step 3: Configure WireGuard Interface on Each Node**

Now, you'll create the WireGuard configuration file (`/etc/wireguard/wg0.conf`) on each node.

**Important:** Replace `[NODE_NAME]_private_key_string`, `[NODE_NAME]_public_key_string`, and `[NODE_NAME]_internal_ip` with the actual values you recorded in Step 2 and your Pi's actual LAN IP addresses.

**Configuration for `node1` (`192.168.1.101`)**

```bash
sudo nano /etc/wireguard/wg0.conf
```

Paste the following, replacing placeholders (BELOW IS EXAMPLE ONLY):

```bash
[Interface]
PrivateKey = node1_private_key_string
Address = 10.21.22.1/24 # CURRENT NODES WIREGUARD IP
ListenPort = 51820 
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PreDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer] # Peer 1: node2
PublicKey = node2_public_key_string
AllowedIPs = 10.21.22.2/32 # Allow traffic to node2's WG IP and LAN IP
Endpoint = hosur-vpn.asherslife.in:51820 # For Hosur Based Nodes
PersistentKeepalive = 25 # Keep the tunnel active

[Peer] # Peer 2: node3
PublicKey = node3_public_key_string
AllowedIPs = 10.21.22.3/32 # Allow traffic to node3's WG IP and LAN IP
Endpoint = vpn.asherslife.in:51820 # For Bangalore Based Nodes
PersistentKeepalive = 25 # Keep the tunnel active
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

**Configuration for all other nodes as well**

***

#### **Step 4: Enable and Start WireGuard on All Nodes**

**Repeat this step on `node1`, `node2`, and `node3`.**

1.  **Bring up the WireGuard interface:**

    ```bash
    sudo wg-quick up wg0
    ```
2.  **Enable WireGuard to start on boot:**

    ```bash
    sudo systemctl enable wg-quick@wg0
    ```
3.  **To Restart WireGuard Service:**\


    ```bash
    sudo systemctl restart wg-quick@wg0
    ```
4.  **Check WireGuard status:**

    ```bash
    sudo wg
    ```

    You should see your interface details and the peers. If the connection is successful, you'll see a `latest handshake` timestamp and `transfer` data.

***

#### **Step 5: Test the Mesh Network**

From any node, you should now be able to ping the _WireGuard IP addresses_ of the other nodes.

If you can successfully ping between the `10.0.0.x` addresses, your WireGuard mesh network is active!

***

#### **Optional: Enable IP Forwarding (if nodes need to route traffic for each other)**

If you want your Raspberry Pis to act as routers for other devices on their respective LANs to reach the WireGuard network, or if you plan to introduce more complex routing, you'll need to enable IP forwarding.

**Repeat this on all nodes:**

1.  **Edit `sysctl.conf`:**

    ```bash
    sudo nano /etc/sysctl.conf
    ```
2.  **Uncomment (remove `#`) or add this line:**

    ```bash
    net.ipv4.ip_forward = 1
    ```
3. **Save and exit.**
4.  **Apply changes immediately:**

    ```bash
    sudo sysctl -p
    ```
