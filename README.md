Noia Node setup guide on Ubuntu using terminal only
==================

* This article is a branch of of a guid by "Surf-N-Code", modified by myself to make it a little
more user friendly for those new to the command line.
Those more comfortable with the command line may prefer the original
guide: https://github.com/Surf-N-Code/noia-node-terminal *

This is a quick tutorial for those wanting to host a NOIA node on a virtual box for example. I have used Digital Ocean for this purpose and setup a Ubuntu 18.04 droplet on the smallest configuration, which is more than adequate for this stage.

If you are interested in signing up on Vultr too, you could use my [ref link](https://m.do.co/c/a2f92f54df25) to sign up. Otherwise just browse to [DigitalOcean](https://www.digitalocean.com) and sign up.

If you need help setting up a VM, contact me here.

Register and KYC on the NOIA platform
-------------
[Register here](https://dashboard.noia.network/r/0d304917)

Initial system checks and requirements
-------------

If you are using an existing setup, it's best to use a normal user account to run the node. It may be OK to run as root but I had problems with the setup like that.

So first thing is to create a new user. For this example we will create the user "noia":

Log in to your server as the root user.

ssh root@server_ip_address
Use the adduser command to add a new user to your system.

Be sure to replace username with the user that you want to create.

adduser username
Set and confirm the new user's password at the prompt. A strong password is highly recommended!

Set password prompts:
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Follow the prompts to set the new user's information. It is fine to accept the defaults to leave all of this information blank.

User information prompts:
Changing the user information for username
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n]
Use the usermod command to add the user to the sudo group.

usermod -aG sudo username
By default, on Ubuntu, members of the sudo group have sudo privileges.

Test sudo access on new user account

Use the su command to switch to the new user account.

su - username
As the new user, verify that you can use sudo by prepending "sudo" to the command that you want to run with superuser privileges.

sudo command_to_run
For example, you can list the contents of the /root directory, which is normally only accessible to the root user.

sudo ls -la /root
The first time you use sudo in a session, you will be prompted for the password of the user account. Enter the password to proceed.

Output:
[sudo] password for username:





--------

We need to make sure that systemd is being used for managing deamons. Run the following command. If your system is using upstart, please refer to this great [tutorial]( https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it)

    ps -p 1 -o comm=

If you haven't got one already, create an ERC20 wallet here: [MEW](https://www.myetherwallet.com/)

Node JS and NPM are required for the noia node to run. Install using the commands below

    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
    sudo apt-get install -y nodejs
    sudo apt-get install build-essential

Noia node installation
------------
I created a new directory below my home directory. You can create it anywhere you like though. I will be using the path: /home/noia in this tutorial.

    git clone https://github.com/noia-network/noia-node-terminal.git
    cd /home/noia/noia-node-terminal

This is enough to test the installation using:

    npm install

    npm start

A couple of things will happen, in the end you should get an error though. Something like this:

    [NODE]: initialized.
    [NODE]: starting...
    /home/noia/noia-node-terminal/node_modules/noia-node/dist/lib/master.js:66
                throw new Error("master address null or undefined");
                ^

    Error: master address null or undefined
        at Master.connect (/home/noia/noia-node-terminal/node_modules/noia-node/dist/lib/master.js:66:19)
        at Node.start (/home/noia/noia-node-terminal/node_modules/noia-node/dist/index.js:74:25)
        at Object.<anonymous> (/home/noia/noia-node-terminal/index.js:74:6)
        at Module._compile (internal/modules/cjs/loader.js:702:30)
        at Object.Module._extensions..js (internal/modules/cjs/loader.js:713:10)
        at Module.load (internal/modules/cjs/loader.js:612:32)
        at tryModuleLoad (internal/modules/cjs/loader.js:551:12)
        at Function.Module._load (internal/modules/cjs/loader.js:543:3)
        at Function.Module.runMain (internal/modules/cjs/loader.js:744:10)
        at startup (internal/bootstrap/node.js:238:19)

If you get a different error, you need to debug - make sure latest npm and nodejs versions are installed

Noia node configuration
-------------

Having run the command above, should have generated a settings.json file within your noia-node-terminal folder. Open that file and adjust a couple of settings as described below:

    nano settings.json

Adjust the line for masterAddress to match this:

    "masterAddress": "ws://csl-masters.noia.network:5565"

Adjust the line for wallet.address to match this:

    "wallet.address": "Your ERC20 wallet address"

Test the node again by running
    npm start

Now you should some logging of [info] message about your configuration such as this line:

    info: [noia-node] Creating wire... interface=gui, node_ip=yourip, node_ws_port=7676, node_domain=, node_wallet_address=your ETH address

You should not see any error messages at this point. If you do, please make sure you have executed all steps from above correctly. Theoretically you can now let this run, but we want the node to be running as a service in the background and startup automatically when the machine reboots for example.

Adding noia node as a service
-------------
Open the noia-node.service file provided within your installation path

    nano noia-node.service

Adjust the ExecStart and WorkinDirectory paths to your node and noia installation paths. See an example of my service file witin this repo.

Once done, copy the service file to your /etc/systemd/system directory, adjust its execution rights and restart the systemctl daemon.

    cp /home/noia/noia-node-terminal/noia-node.service /etc/systemd/system/
    chmod 664 /etc/systemd/system/noia-node.service
    systemctl daemon-reload
    systemctl enable noia-node.service
    systemctl start noia-node.service

Check if everything works
------------

To check whether the node is running type

    systemctl | grep noia

You should see something like this:

    noia-node.service  loaded  active  running NOIA Node
