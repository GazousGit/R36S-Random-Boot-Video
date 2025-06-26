# R36S-Random-Boot-Video

Hi,

Here's a little trick I did to get some randomized video to be played on ArkOS boot.
(This should work with any OS unix based but I only tested it with ArkOS)

All you need is to be able to connect to your device with SSH to add the script and create a service.

The videos files goes in the boot partition of your SD card (note that you can of course change anything in the script about that)

Someone should be mentionnend cause they were the first (or one of the first) to achieve this, I just fancied things a bit:
https://www.reddit.com/r/R36S/comments/1h5xvgm/boot_sound/ 

Their account is deleted from reddit or so, so I have no idea who this is but thank you anyway 💗

## _Instructions_
### Requirements
- R36S with ArkOS (or any corersponding handled device with a UNIX OS although I can't guarantee success, you might need to install packages)
- An ethernet or Wifi to USB-C adapter (personnaly I use a TP-Link ethernet to USB-C UE300C and it works like a charm)
- A computer (Windows, Mac, Linux)
    - If you're on **linux or MacOS** I will assume you know how to connect with to an IP via SSH
        - If you don't then look it up, it's actually easier than with windows
    - If you're on **windows** you will also need some software to ease your life
        - **Putty**, so you can easly connect with SSH (Windows terminal or others tools could do, bu I like putty, up to you if want something else)
            - [Download Putty](https://www.putty.org/)
        - **WinSCP** (Optionnal) it will allow you to navigate easly into ArkOS folders and do neat stuff, you don't really need it for this tutorial
            - [Download WinSCP](https://winscp.net/eng/index.php)


### Before anything
I'm in a good mood, so here's my personnal collection of boot videos, enjoy : 
👉[The holy Grail of boot videos](https://drive.google.com/drive/folders/16npZavAaKatOQUN4gcF6alFIhi_7RNMh?usp=sharing)👈

Put your SD card in your computer and add a folder into the boot partition **boot_videos**

Place your videos in mp4 format there (try to stick to short videos like 10-15sec with a short name)

Nothing force you to use the boot partition, you could as well put your video folder in the EASYROM partition but you will have to change the folder in the script (search **VIDEO_DIR** below) 

Be very careful because the root folder is not called EASYROM of course, I think it's called roms or roms2, you could easly check that through putty or winscp.


### Connect to your R36S with SSH
1. Connect your R36S using wifi or ethernet via USB-C (Don't use your phone since you want your R36S to be on the same network as your computer)
2. Go to the **Options** menu and select **NETWORK INFO** so you can see your the assigned IP to your device
3. Click A to continue and you will go back out to the Options menu
4. Back in the **Options**, Select **Enable Remote Services** so that SSH is enabled before attempting to connect.

Now start putty and put the IP you got from the step 2 above (you can still go to network info from here)

It will ask you User and Password -> credentials are **ark/ark**

Once you are logged in, you will want to connect as root (admin)
To be able to do that you will need to set a password for your root user

| ❗  If it doesn't work, maybe something changed so I'd suggest you check the original ArkOS wiki about SSH : [ArkOS wiki info about connecting with SSH](https://github.com/christianhaitian/arkos/wiki/Frequently-Asked-Questions---CHI#q-how-do-i-ssh-into-arkos)   |
|-----------------------------------------|

### Set up root access
Execute the command :
```sh
sudo passwd root
```
It will ask you a password so type it and press enter, it will then ask to confirm the password so repeat the process and press enter again.

Now you need to allow root to connect with ssh so execute this command :
```sh
sudo nano /etc/ssh/sshd_config
```
Then scroll down and find the line that start with **#PermitRootLogin**

You need to change the line from
```
#PermitRootLogin whatever_is_written_here
```
To
```
PermitRootLogin yes
```
Do not forget to remove the Hashtag or the line will be ignored

Now press **ctrl + X** and type **y** to save changes

Reboot your device by executing this command :
```sh
sudo reboot
```

Your device will restart and you have to do the same process (Step 1 to 4) to connect to SSH in the options.
(I am well aware you can keep ssh always on, but I wouldn't do it, especialy if you play with wifi on on your device)

Now connect again to your R36S with putty but do not use **ark/ark** as user password to connect but use **root/yourpassword**

Congrats you're connected as admin, now let's do the such awaited work

| ❗  If it doesn't work, maybe something changed so I'd suggest you check the original ArkOS wiki about SSH : [ArkOS wiki info about connecting with SSH](https://github.com/christianhaitian/arkos/wiki/Frequently-Asked-Questions---CHI#q-how-do-i-ssh-into-arkos)   |
|-----------------------------------------|

### Set up the random video script
We'll create a new file to put the script content, execute this command :

```sh
touch /usr/local/bin/play-random-video
```

Then we open this file with our text editor, execute this command :

```sh
sudo nano /usr/local/bin/play-random-video
```

Copy and paste the content below into your file

```bash
#!/bin/bash

VIDEO_DIR="/boot/boot_videos"  # Your MP4 videos directory, feel free to change this
VIDEO_FILES=("$VIDEO_DIR"/*.mp4)

# Check if any videos exist
if [ ${#VIDEO_FILES[@]} -eq 0 ]; then
    echo "No MP4 files found in $VIDEO_DIR!" >&2
    exit 1
fi

# Pick a random video
RANDOM_VIDEO=$(printf "%s\n" "${VIDEO_FILES[@]}" | shuf -n 1)

# Play the video
/usr/bin/ffplay -autoexit "$RANDOM_VIDEO"

```

Now press **ctrl + X**
Type **y** to save changes

Execute the commande below (important) :

```sh
sudo chmod +x /usr/local/bin/play-random-video
```

Now we open an existing file to put our script into a service, execute this command :

```sh
sudo nano /etc/systemd/system/play-video.service
```

Make sure the file have the same content as below (I think you just need to change the **ExecStart** line)

```bash
[Unit]
Description=Play a random boot video
Before=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/play-random-video
Restart=no

[Install]
WantedBy=default.target
```

Now press **ctrl + X**
Type **y** to save changes

Then we execute 2 commands to enable the service (one after the other) :

```sh
sudo systemctl daemon-reload
```
```sh
sudo systemctl enable play-video.service
```

Now reboot your device and you should see your video :)
```sh
sudo reboot
```