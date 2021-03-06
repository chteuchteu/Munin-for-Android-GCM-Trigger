# Munin for Android GCM Trigger
Python script called by munin, sending signals to [Munin for Android GCM Proxy](https://github.com/chteuchteu/Munin-for-Android-GCM-Proxy).

You have to install this script on the master server of your munin installation. Here's how it works:

1. munin detects an alert and calls this script
2. This script sends the signal with plugin information to the proxy, relaying the info to Google Cloud Messaging
3. An alert notification appears on your devices

> Tip: Watch + Star this repository to be notified when a new version of this script comes out!

## Installation
You only have to install this script once, even if several Android devices will be notified.

### 1. Install & configure the script

The script relies on the [requests](https://github.com/kennethreitz/requests) library to communicate with Google Cloud
Messaging. Make sure the lib is present on your system by running the following command first (you might need to `sudo` it on some systems) : 

```bash
pip install requests
```

You must put this script in a directory accessible by munin. A good place would be /home/munin:

```bash
sudo mkdir /home/munin
sudo chown "$USER":munin /home/munin
```

Clone this repository on your server to download the script, or just download it as a ZIP archive:
    
```bash
# Navigate to the script final location
cd /home/munin

# Clone the repo
git clone https://github.com/chteuchteu/Munin-for-Android-GCM-Trigger.git

# ... OR download the ZIP archive
wget https://github.com/chteuchteu/Munin-for-Android-GCM-Trigger/archive/master.zip
unzip master.zip -d Munin-for-Android-GCM-Trigger
rm master.zip

# Update directory permissions
sudo chown -R "$USER":munin Munin-for-Android-GCM-Trigger
```
    

Don't forget to mark main file as executable:

```bash
cd Munin-for-Android-GCM-Trigger
chmod ug+x main.py
```
    
If not already done, request your unique device id for each device you'll use. Navigate to the notifications screen on
the app and hit the *Send me the instructions by mail* button.

Call this script using the `--add-device DEVICE_ID` argument or manually edit the `devices.json` file
and add the device id(s) in it. When interactively adding your device using `--add-device`, you'll
have the choice to send a test notification to the devices passed as argument.

```bash
./main.py --add-device VOPCG0LUaXWcnl56g2yp...
```

The `devices.json` file should look like this:
```json
{
    "devices": [
        "VOPCG0LUaXWcnl56g2yp...",
        "BLlWcH6Rh7Sb3t1S4bY1...",
        "dkOoc2qDCtaHvY5yJSg7..."
    ]
}
```

### 2. Test it
Once done, you can check if the script works by running the test command:

```bash
./main.py --test
```

A confirmation notification should appear on all of your devices:
![Test notification](README_testNotification.png)


### 3. Configure munin
We have to configure munin in order to make it call this script on each alert.
Open `/etc/munin/munin.conf`, and configure it as following. Replace `/home/munin/Munin-for-Android-GCM-Trigger/` with the script location.

```
# Munin for Android notifications
# Configure script location & args
contact.munin_for_android.command /home/munin/Munin-for-Android-GCM-Trigger/main.py

# Uncomment this if you want to be notified every 5 minutes about every alert
# contact.munin_for_android.always_send warning critical

# Set infos format
contact.munin_for_android.text  <alert group="${var:group}" host="${var:host}"\
  graph_category="${var:graph_category}" graph_title="${var:graph_title}" >\
  ${loop< >:wfields <warning label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
  ${loop< >:cfields <critical label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
  ${loop< >:ufields <unknown label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
  </alert>
```

Restart `munin-node` service to take configuration changes into account:

```bash
service munin-node restart
```

**That's it!**

## Update this script
If you're unsure about how you installed this script, check if a `.git/` directory
exists inside the script directory. If it does, follow the instructions under (1).

1. You installed it using `git clone https://github.com/chteuchteu/Munin-for-Android-GCM-Trigger.git`:
    You just have to run `git pull` to update it. If you encounter `error: Your local changes to the following
    files would be overwritten by merge:` error, check the
    files list below the error message:
    
    - if it contains `devices.py`, save it (`cp devices.py devices.py.bak`), pull (`git pull`) and restore it (`mv devices.py.bak devices.py`)
    - otherwise, undo these changes (`git checkout -- .`) and pull (`git pull`)
    
    You'll be more likely to encounter this error if you're updating from a version cloned before
    August 2016 - we fixed everything in the meantime.
 
2. You installed it by downloading the ZIP archive:
    
    ```bash
    # Move the script to another location
    mv Munin-for-Android-GCM-Trigger Munin-for-Android-GCM-Trigger.bak
    
    # Download its latest version
    wget https://github.com/chteuchteu/Munin-for-Android-GCM-Trigger/archive/master.zip
    unzip master.zip -d Munin-for-Android-GCM-Trigger
    rm master.zip
    
    # Copy back the devices list from the previous install
    cp Munin-for-Android-GCM-Trigger.bak/devices.py Munin-for-Android-GCM-Trigger/devices.py
    
    # If you customized values in const.py (GCM_PROXY_URL, HELP_DIAGNOSE or whatever), don't forget to update them in the new location
    ```

**In both cases, test the updated version**:

```bash
cd Munin-for-Android-GCM-Trigger && ./main.py --test
```
