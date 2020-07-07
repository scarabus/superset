## Install Superset on a Google VM
First register for Google Cloud. It's free.
We will create a VM and install Superset on it. 
But in order to connect to the UI we need to permit traffic through the firewall.

### Set up firewall to allow traffic to port 8088
You cannot use port 80 unless you run the app via sudo, in which case the login screen will be presented but logging in silently fails. 
So we will use port 8088 and configure the firewall to allow access.

Go to VPC Network -> Firewall
Click on Create Firewall Rule
By default *Targets* is set to *Specified target tags*.
Either set this to *All instances in the network* (easier, less secure)
or specify a target tag (any name, eg "SupersetPort"), and apply that tag to the VM Instance in the networking section (see below)

Set source IP ranges to 0.0.0.0/0 (this allows traffic from anywhere; we don't really care at this point)

Under Protocols and Ports check tcp and either enter 8088 or, if you plan to experiment with other tools, set a range such as 8000-8100
Finally click Create.

### Create a Google virtual machine. 
Go to Compute Engine -> VM Instances
Click Create instance

You can always increase the machine type and disk if you need to, so start off small.

You can choose your preferred OS. 
I used Ubuntu 20:04 because it's the most recent version (as I type this), but also because it does NOT have python2 installed by default.
I find it frustrating when a system has two incompatible versions of Python installed.

Allow full access to all Cloud APIs
Allow HTTP/HTTPS traffic
If you chose *Specified target tags* for your Firewall rule then below Firewall select Networking and add the Network Tag that you created.

Click create. This takes a couple of minutes, so while that's happening get the JSON service account key from
https://console.cloud.google.com/apis/credentials/serviceaccountkey

### Log on to the VM (verify this)
You can SSH to the VM by clicking the button and a window will appear. 
You can also ssh directly from a terminal if you use the External IP address.
If you plan to do this it's a good idea to register your SSH public key with the server - 
run ssh-keygen -t rsa and then register the key with the VM - 
Compute Engine -> Metadata, select SSH Keys, click Edit, click Add Item

### Install Superset
    sudo apt-get update 
    sudo apt-get install python3-pip libssl-dev libffi-dev libsasl2-dev libldap2-dev python3-venv 
    pip3 install virtualenv  
    python3 -m venv virtual_env 
    . ~/virtual_env/bin/activate 
    pip3 install --upgrade setuptools pip  

Put the JSON key that you created earlier into a file and set **GOOGLE_APPLICATION_CREDENTIALS** to point to that file on startup.

    vim ~/cdp_key.json 
    echo 'export GOOGLE_APPLICATION_CREDENTIALS="$HOME/cdp_key.json"' >> ~/.bashrc 
    pip3 install apache-superset 
    superset db upgrade  
    export FLASK_APP=superset 
    flask fab create-admin 

Create an admin name and password (admin/admin is fine for testing)

    superset load_examples 
    superset init 
    pip3 install pybigquery 
    superset run --port 8088 --host 0.0.0.0 --with-threads --reload --debugger 

You should now be able to connect to the server from your desktop at the public IP address port 8088
If you can't, try running a simple web server on port 80 and see if you can connect to that -
Try connecting from the VM and from your deskktiop using wget

    sudo python3 -m http.server 80
