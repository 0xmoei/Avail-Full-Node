![ alt text](https://github.com/0xmoei/Avail-Full-Node/blob/main/sync.png)

#Avail Full Node Guide
## System Requirements
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 4GB | 8GB |
| CPU (amd64/x86 architecture) | 2 core | 	4 core |
| Storage (SSD) | 20-40 GB | 200-300 GB |
**OS Recommended Ubuntu 22.04**

## Install Full Node

1. Install Dependecies:

    ```bash
    sudo apt-get -y update &&
    sudo apt-get install screen -y &&
    sudo apt-get -y install build-essential &&
    sudo apt-get -y install --assume-yes git clang curl libssl-dev protobuf-compiler && 
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh &&
    source ~/.cargo/env &&
    rustup default stable &&
    rustup update &&
    rustup update nightly && 
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```
    
2. Remove old files of Avail Node (if you participated in Kate testnet):

    ```bash
sudo systemctl stop availd && \
sudo systemctl disable availd && \
rm /etc/systemd/system/availd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .avail && \
rm -rf avail && \
rm -rf $(which availd)
    ```
    
3. Create Screen:
* Step 4 takes a little longer to finish. The "screen" command can help you recover your terminal screen if it gets interrupted

 ```bash
   screen -S avail
    ```

4. Build the lastest version of Avail Node (v1.8.0.0):

    ```bash
    mkdir -p $HOME/avail-node &&
    cd $HOME/avail-node &&
    git clone https://github.com/availproject/avail.git &&
    cd avail &&
    mkdir -p output &&
    mkdir -p $HOME/avail-node/data &&
    git checkout v1.8.0.0 &&
    cargo run --locked --release -- --chain goldberg -d ./output
    ```
     <img src="https://github.com/0xmoei/Avail-Full-Node/blob/main/build1.png">
    Wait for the process to complete.

     <img src="https://github.com/0xmoei/Avail-Full-Node/blob/main/build2.png">
    Now if your terminal is like the above pic, press Ctrl + C 

5. Create systemD (**Before creating the service, please read all this step notes, then proceed with the command**)

Find you `[$HOME]` directory (you need this later)
 ```bash
   $home
    ```

My `[$HOME]` is /root in the following example
  
       <img src="https://github.com/0xmoei/Avail-Full-Node/blob/main/%24home.png">

Create a .service file
    ```bash
    sudo touch /etc/systemd/system/availd.service && sudo nano /etc/systemd/system/availd.service
    ```

    Now you are editing .service file, paste the following command in it
**(Change '[$HOME]' in the command below before pasting it into the service. Please read more below)**:

    ```
    [Unit] 
    Description=Avail Validator
    After=network.target
    StartLimitIntervalSec=0

    [Service] 
    User=root 
    ExecStart=[$HOME]/avail-node/avail/target/release/data-avail --base-path [$HOME]/avail-node/data --chain goldberg --port 30333  --rpc-cors=all --rpc-external --rpc-methods=unsafe --rpc-port 9933 --prometheus-port 9615  --validator --name "my-node"
    Restart=always 
    RestartSec=120

    [Install] 
    WantedBy=multi-user.target
    ```

    **In the above command, please note the following information:**
   
    -  `[$HOME]` if you typed the command $HOME, then copy the path to replace in `[$HOME]` above as shown in the example above. Replace `[$HOME]` with `/root`
`[$HOME]/avail-node/avail/target/release/data-avail` will be `/root/avail-node/avail/target/release/data-avail`
`[$HOME]/avail-node/data` will become `/root/avail-node/data`.
 
    - `--name` replace 'my-node' with your favorite node name.
    - Ports `30333, 9933, 9615` must be opened in the firewall. If you are using a VPS, configure it to allow TCP/UDP connections through these ports. If you're using a VPS, please make sure the port is open from the provider's side.

After editing, press Ctrl + O and then Enter, then press Ctrl + X to exit.

6. Open ports
    ```bash
sudo apt-get install ufw && sudo ufw enable && sudo ufw allow 30333/tcp && sudo ufw allow 9933/tcp && sudo ufw allow 9615/tcp
    ```

7. Enable and start the Node:

    ```bash
    systemctl enable availd.service && systemctl start availd.service
    ```

8. Check Node status:

    ```bash
    systemctl status availd.service
    ```
<img src="https://github.com/0xmoei/Avail-Full-Node/blob/main/sync.png">
Press Ctrl + C to quit

9. Check Node logs:

    ```bash
    journalctl -f -u availd
    ```
<img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/service-status.png">
If the finalized number equals the target number, your node is synced

To check your node, visit [https://telemetry.avail.tools/](https://telemetry.avail.tools/). Your node will be displayed after the synchronization process is complete and the node starts running.
<img src="https://github.com/hiephtdev/huong-dan-chay-full-node-avail/blob/main/check-tool.png">

#### Contact me

X: [https://twitter.com/0xMoei](https://twitter.com/0xMoei)
