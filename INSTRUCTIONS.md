# Setup Instructions

#### 0. Clone this repository somewhere.
- Run `git clone https://github.com/The-Yerbas-Endeavor/wyerbas-bsc.git`. This creates a `wyerbas-bsc` folder containing the files necessary for running the authority node.
- Make sure it is cloned, and not just downloaded. We need the .git metadata for version tracking.

#### 1. Add Change address to yerbas wallet:
- Ensure Yerbas daemon is running: `yerbasd`.
- Run `./yerbas-cli createmultisig 3 "[\"020a16865dcfbb809c03bef35cf1f96b3811dc1a77e3c156ab82d0cdbf1a6f7870\", \"033061b8b122fd581aadce0c3cd0fdcc537e38daf795dbbbdf70ffc67dc1bb5daf\", \"028fc2832a72b4d12fbd9e6ea488f5ecac08caa9e174f23becd9a299e308f66f2e\", \"0260a59e7298e339e0c6bd53142c9279a671c24e95e42672d96a82bf72b7b428ea\", \"0245fe3d0363673a483a3c330829eeb7663930a46e9ef74d67f11fd7557d1a206b\"]"`
- Ensure that the output is `8zMwZPRNZT5Ltz7ACEVAbejKB6v89wjADn`. This will be the CHANGE_ADDRESS where all holdings are collated at every payout.
- Run `yerbas-cli importaddress "REDEEM_SCRIPT" "" true true` where REDEEM_SCRIPT is from running `yerbas-cli create multisig...`.

#### 2. Add BSC private key.
- Copy the private key file  `cp ~/wyerbas-bsc/settings/EXAMPLE.private.DO_NOT_SHARE_THIS.json ~/wyerbas-bsc/settings/private.DO_NOT_SHARE_THIS.json`
- In `settings/private.DO_NOT_SHARE_THIS.json`, replace `0xExampleWhichYouShouldReplace` with your BSC_WALLET_PRIVATE_KEY. Ensure that the double quotes remain around your private key.
- DO NOT REVEAL THIS AT ALL COSTS.

#### 3. Create your database to record payouts history.
- Install `sqlite3`: `sudo apt install sqlite3`
- Go to the `wyerbas-bsc/database` folder: `cd wyerbas-bsc/database`
- Create the database file and enter sqlite3 console: `sqlite3 wyerbas.db`
- Run the initialization script: `.read schema_authority.sql`
- Once done, you can simply exit the console via `Ctrl+X` or `Ctrl+D`.

#### 4. Setup SSL.
- Setup SSL related software:
    - `sudo apt install snap`
    - `sudo snap install --classic certbot`
    - `sudo ln -s /snap/bin/certbot /usr/bin/certbot`
- Generate certs: `sudo certbot certonly --standalone --register-unsafely-without-email`
    - You may be asked to accept terms of service. Press Y to agree.
    - You will be asked to enter the domain name for the certificate. Enter yours correspondingly based on your node index: `n0.yerbas.org`, `n1.yerbas.org`, ..., `n4.yerbas.org`.
    - This step may fail if port 80 and 443 is blocked. Please temporarily allow access to these ports.
- Upon success, you will see a message saying something like:
    ```
    ...
    Certificate is saved at: /etc/letsencrypt/live/n4.yerbas.org/fullchain.pem
    Key is saved at:         /etc/letsencrypt/live/n4.yerbas.org/privkey.pem
    ...
    ```
    Note down these two paths (we will refer to them as CERTIFICATE_PATH and KEY_PATH respectively).
- Set certbot data dir permissions: 
    - `sudo groupadd ssl-cert`
    - `echo $USER | xargs -I{} sudo usermod -a -G ssl-cert {}`
    - `sudo chgrp -R ssl-cert /etc/letsencrypt`
    - `sudo chmod -R g=rX /etc/letsencrypt`
    - Re-login.
- Add CERTIFICATE_PATH and KEY_PATH to wyerbas settings:
    - Copy the ssl file  `cp ~/wyerbas-bsc/settings/ssl-EXAMPLE.json ~/wyerbas-bsc/settings/ssl.json`
    - Open `wyerbas/settings/ssl.json`
    - Set value of `certPath` and `key_path` to CERTIFICATE_PATH and KEY_PATH respectively.
    - e.g. `ssl.json`:
        ```
        {
          "certPath": "/etc/letsencrypt/live/n4.yerbas.org/fullchain.pem",
          "keyPath": /etc/letsencrypt/live/n4.yerbas.org/privkey.pem"
        }
        ```

#### 5. Install nodejs and yarn, and setup project dependencies.
- `sudo apt install nodejs npm`
- `sudo npm install -g yarn`
- `cd wyerbas-bsc`; `yarn install`

#### 6. Ensure that port 15430 is open. Use https://www.yougetsignal.com/tools/open-ports/ to check that port 8443 on your node can be reached with the authority daemon running. If not, check your firewall/VPS provider/ISP settings to make sure that port 8443 is not blocked.

#### 7. Launch yerbas daemon and authority daemon.
- In the appropriate directory, `./yerbasd -daemon` or if running this (as below) in `tmux` or `screen`, you may leave the ` -daemon` out.
- In your wyerbas directory, `node authorityDaemon.js` (If `node authorityDaemon.js` does not work, try `nodejs authorityDaemon.js`).
- You can run these daemons in a `tmux` or in `screen`. This allows you to leave the SSH session while still running and recording stdout/stderr debug messages in the background.

#### 8. Users access the system via the [web application](https://wrap.yerbas.org/). The original repository's Github Pages [web application](https://wyerbas.github.io/wyerbas-frontend/) also redirects here. The source code can be found [here](https://github.com/The-Yerbas-Endeavor/wyerbas-frontend). The web application interacts directly with the authority nodes. Alternatively you use the CLI (`node cli.js`; `help`) to interact with the authority nodes.

# Checking consensus among the authority nodes :

#### Verifying shared credentials
- In `wyerbas/settings/public.json`, check that the IP locations and BSC wallet addresses of every node match the collected list.
- In `wyerbas/settings/yerbas.json`, check that the change address matches the one computed in step 1 (`8mGFvm9FtczLDjeZ18YAiWpGcbYRqsjF8s`), and that the tax payout addresses match the collected list.
- In `wyerbas/settings/smartContract.js`, check that the contractAddress is what has just been published, `0x2100591c0b692c53a0E11cc328646309e6ea12eF`.
- The above is easily done via the `consensus` command run inside cli.js
    - `node cli.js`
    - `consensus`
    - When finished, you may close the consensus window with `.exit`

#### Verifying smart contract
- You can explore the BSC Smart Contract here: `https://bscscan.com/token/0x2100591c0b692c53a0E11cc328646309e6ea12eF`. For the folks who are new to this, this is basically like blockexplorer but you see all interactions with the smart contract.
- The Smart Contract source code has been uploaded and verified here: `https://bscscan.com/address/0x2100591c0b692c53a0E11cc328646309e6ea12eF#code`.
- Verify that the `_authorityAddresses` (Line 203) has been set to the BSC wallet addresses in the collected list.
- If you are up for it, you can read through the smartContract to verify the multisignature design.
