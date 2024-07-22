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


## connect to Pi

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


## mounting the external ssd


## configuration

## run it