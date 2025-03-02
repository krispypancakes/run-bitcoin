# run-bitcoin
Running bitcoin knots on a Raspberry Pi 4 with an external SSD.
A step-by-step guide for dummies (me).

# Set up the pi

I use a Raspberry Pi 4, a mini SD card with 32gb, an external ssd with 2 Tb.

## create image
- Download [Ubuntu Server](https://ubuntu.com/download/raspberry-pi)
- Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

In the pi imager UI, choose your device, point to the path of the downloaded image when 
choosing the operating system and select your SD card as storage and press NEXT.

Then it will ask you for OS customization, press EDIT SETTINGS.

Here, you can set a hostname, username, password, wifi credentials and also add your public key so you can ssh into it once it is connected to wifi.

NEXT.

Wait until it says: Write Successful.

Inject sd card into pi and power on.


# Connect to Pi

Connect to your Pi using:

```
ssh <Username>@<Hostname>
```

Update the system:

```
sudo apt update
sudo apt upgrade
```
(This can take some time...)

Maybe you need to reboot the Pi:

```
sudo reboot
```

## download bitcoin knots (or bitcoin core if you wish)

Go to https://bitcoinknots.org 

Show other Downloads
Right click on aarch64, which is the correct architecture of my pi. (If you're not sure, you can check on your pi by just typing `arch`.)
And copy the link.

Create a directory and download the file.
```
mkdir btc && cd btc
wget <the link you copied>
```

## verify the download

Also refer to this manual on [bitcoin-core.org](https://bitcoincore.org/en/download/)

Download digital signatures and fingerprints from the website.

```
wget <link of fingerprints>
wget <link of signatures>
```
This will download SHA256SUMS.asc and SHA256SUMS.

Verify that checksum of release file is listed in checksums file:

```
sha256sum --ignore-missing --check SHA256SUMS 
```

Download all developer keys from bitcoin-core repo and import them:

```
git clone https://github.com/bitcoin-core/guix.sigs.git

gpg --import guix.sigs/builder-keys/*
```

```
gpg --verify SHA256SUMS.asc SHA256SUMS
```
What we are looking for is: 

```
Good signature from <bitcoin dev>

Primary key fingerprint: <key>
```

Extract the download:

```
tar -xf bitcoin-...-aarch64-linux-gnu.tar.gz
```

## mounting the external ssd

Plug in the SSD card. Check name and mount.

```
sudo sfdisk -l
```
My mountable device is `/dev/sda1`.

Create a data directory `mkdir data` and create a script `mount_disk.sh` to mount the device to it.

```
#!/bin/bash
sudo mount -o rw,uid=$(whoami),gid=$(whoami) /dev/sda1 data/
```
After creating the script, add rights to be able to execute it and run.

```
chmod +x mount_disk.sh
./mount_disk.sh
```

Explanation:
-o : options
rw: read and write access
uid: user id, enter your username
gid: group id, also enter your username
/dev/sda1: our device
data/: the directory to mount to.

We need the mounted directory to be owned by your user.

## install bitcoin knots

```
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-27.1.knots20240621/bin/*
```


## configuration
A few things need to be adjusted in the config file `btc/data/.bitcoin`
```
server=1
daemon=1
rpcallowid=127.0.0.1
rpcuser=<your user>
rpcpassword=<your password>
```
Create a user and a password to increase security.

# Run it

```
bitcoind -datadir=/home/<your user>/btc/data/.bitcoin -daemon
```

Check status:

```
bitcoin-cli -datadir=/home/<your user>/btc/data/.bitcoin getblockchaininfo
```

The process will log to the file `/home/<your user>/btc/data/.bitcoin/debug.log` relentlessly.
That's why we set up a cronjob to clear the content of the log file to not crash the node.

```
EDITOR=vim crontab -e
```

Then add following line:

```
0 0 */2 * * > /home/<your user>/btc/data/.bitcoin/debug.log
```
Save and exit. (:wq lol)

Check with `crontab -l` if the job was created.


FINALLY RUN IT:

```
bitcoind -datadir=/home/<your user>/btc/data/.bitcoin -daemon
```

Verify that it's running and check the logs:

```
bitcoin-cli -datadir=/home/<your user>/btc/data/.bitcoin getblockchaininfo
tail -f btc/data/.bitcoin/debug.log
```

Here you go. You should be set.
