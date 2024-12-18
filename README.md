## Mine Bitcoin and Run a Node Without Breaking the Bank

Mining Bitcoin and running a node doesn’t have to mean sky-high electricity bills or expensive hardware. With a bit of ingenuity and the right tools, you can set up a cost-efficient Bitcoin node and start mining on a budget. Here’s how to do it using a Raspberry Pi and a Bitaxe miner.

### What You’ll Need
- **Raspberry Pi 4**
- **1TB SSD** with a powered USB adapter
- **Bitaxe Miner**
- A reliable **WIFI internet connection**

### Step 1: Preparing the Raspberry Pi and the SSD

#### Install the Operating System
First, you’ll need to install a 64-bit OS (aarch64) on your Raspberry Pi. This will provide the necessary architecture to run Bitcoin Core and manage the blockchain data.

1. Download the 64-bit Raspberry Pi OS from the official site. [RaspberryPI OS](https://www.raspberrypi.com/software/)
2. Flash it onto an SD card using tools like Balena Etcher or Raspberry Pi Imager.
3. Insert the SD card into the Raspberry Pi and boot it up.

#### Format and Mount the SSD
The blockchain data requires a large, fast storage medium. We’ll use a 1TB SSD for this purpose.

1. Connect the SSD to the Raspberry Pi using a powered USB adapter.
2. Format the drive to the `ext4` filesystem:
   ```bash
   sudo mkfs.ext4 /dev/sda1
   ```
3. Create a mount point:
   ```bash
   sudo mkdir -p /mnt/ssd
   ```
4. Mount the SSD:
   ```bash
   sudo mount /dev/sda1 /mnt/ssd
   ```
5. Configure automatic mounting on boot by editing `/etc/fstab`:
   ```bash 
   sudo nano /etc/fstab
   ```
    Add the following line:
   ```
   /dev/sda1  /mnt/ssd  ext4  defaults,noatime,nofail  0  2
   ```

### Step 2: Compiling and Installing Bitcoin Core
The official Bitcoin Core binaries aren’t distributed for the aarch64 architecture, so we’ll compile it from source.

1. Clone the Bitcoin Core repository:
   ```bash
   git clone https://github.com/bitcoin/bitcoin.git
   ```
2. Install dependencies and build tools:
   ```bash
   sudo apt update && sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3 libevent-dev libboost-all-dev libssl-dev
   ```
3. Build Bitcoin Core:
   ```bash
   cd bitcoin
   ./autogen.sh
   ./configure --disable-wallet --without-gui
   make -j$(nproc)
   sudo make install
   ```

### Step 3: Configuring Bitcoin Core

#### Configure the Node
Edit the `bitcoin.conf` file to set up your node:
```bash
sudo nano /mnt/ssd/bitcoin-data/bitcoin.conf
```
Add the following:
```ini
server=1
rpcuser=your_rpc_username
rpcpassword=your_rpc_password
rpcallowip=192.168.2.0/24
rpcbind=0.0.0.0
rpcport=8332
datadir=/mnt/ssd/bitcoin-data
dbcache=300
maxconnections=30
```

#### Configure the Bitcoin Core Daemon as a Service
Create a systemd service to start Bitcoin Core on boot:
```bash
sudo nano /etc/systemd/system/bitcoind.service
```
Add:
```ini
[Unit]
Description=Bitcoin Core Node
After=network.target

[Service]
User=pi
ExecStart=/usr/local/bin/bitcoind -conf=/mnt/ssd/bitcoin-data/bitcoin.conf -datadir=/mnt/ssd/bitcoin-data
Restart=on-failure

[Install]
WantedBy=multi-user.target

> Note: Replace `pi` with your username if it’s different.
```
Enable and start the service:
```bash
sudo systemctl enable bitcoind
sudo systemctl start bitcoind
```

### Step 4: Sync the Node
Once Bitcoin Core is running, it will begin syncing the blockchain. This process can take days, depending on your SSD speed, internet connection, and Raspberry Pi model. Monitor the logs to track progress:
```bash
tail -f /mnt/ssd/bitcoin-data/debug.log
```

### Step 5: Setting Up the Bitaxe Miner
While the node is syncing, you can set up your Bitaxe miner.

1. Plug in your Bitaxe miner and connect to its Wi-Fi network.
2. Access its settings page (usually `http://192.168.4.1` in your browser).
3. Configure your wallet address and Wi-Fi settings.
4. Restart the Bitaxe, and it will begin mining.

### Step 6: Test the Node and Start Mining
Once the node is fully synced, test that you can access it remotely:
```bash
curl --user your_rpc_username:your_rpc_password \
     --data-binary '{"jsonrpc":"1.0","id":"test","method":"getblockchaininfo","params":[]}' \
     -H 'content-type:text/plain;' \
     http://192.168.2.37:8332/
```

### Hardware Costs

Here is an estimate of the hardware costs for this setup:

1. [RaspberryPI OS](https://www.raspberrypi.com/): Approximately $50
2. 1TB SSD with Powered USB Adapter: Approximately $100
3. [Bitaxe Miner](https://www.solosatoshi.com/shop/): Start at $180

> Note that the prices may vary depending on the region and the retailer.
> Pi 5 is also available now with m.2 support. 
> There are more powerful miners available as well.

Total Hardware CostCost of the Setup
$330 (approximate)

### Power Consumption and Costs

| Component           | Power Usage (Watts)   | Description                           |
|---------------------|-----------------------|---------------------------------------|
| Raspberry Pi 4      | 3-7                  | Depends on idle or active state       |
| 1TB SSD             | 2-5                  | Low-power SSD with powered adapter    |
| Bitaxe Miner        | ~5                   | Efficient Bitcoin mining device       |
| **Total (Idle)**    | ~10                  | Pi, SSD, and Bitaxe in low activity   |
| **Total (Under Load)** | ~17               | Pi syncing, SSD active, Bitaxe mining |

---

### Electricity Costs

| Metric              | Value                | Description                           |
|---------------------|-----------------------|---------------------------------------|
| Daily Consumption   | 0.408 kWh            | Based on 17 Watts for 24 hours        |
| Monthly Consumption | 12.24 kWh            | Daily consumption multiplied by 30    |
| Daily Cost          | $0.06               | At $0.15 per kWh                      |
| Monthly Cost        | $1.84               | Daily cost multiplied by 30           |

### Summary
With an upfront hardware investment of \$330 and a monthly electricity cost of less than $2, this is one of the most cost-effective ways to mine Bitcoin and run a full node.

### Conclusion
You’ve now set up a cost-efficient Bitcoin node on a Raspberry Pi and started mining Bitcoin with a Bitaxe. This setup is not only economical but also aligns with Bitcoin’s decentralized ethos. Enjoy contributing to the Bitcoin network and happy mining!


