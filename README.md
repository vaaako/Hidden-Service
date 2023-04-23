# About
In this tutorial I will be teaching:
- How to create a **Hidden Service** *(a `.onion` site)*
- Vanity *(Customizable link)*
- Basic security tips for the **Hidden Service**

This tutorial requires you to know **at least** the basics of:
- Linux
- Apache/Nginx

>Unfortunately this tutorial **still** doesn't have a version for `Nginx`
>This tutorial is focused on Ubuntu/Debian, but nothing prevents you from using equivalent commands (unless you are on windows :P)
>I recommend using `Nginx`, although `Apache` is easier to configure


#Configuration
Install tor:
```sh
sudo apt install tor
```

Use your text editor to open `/etc/tor/torrc`
>*(from now on instead of me saying "Open it with your text editor", I'll just say "Open the file", I recommend using `nano`. Just type `sudo nano path/to/file`)*

Uncomment the lines `71` and `72`
```
HiddenServiceDir /var/lib/tor/hidden_service # I recommend replacing "hidden_service" with the name of your Hidden Service
HiddenServicePort 80 127.0.0.1:8080
```

These lines basically mean: *"any incoming traffic to port of your Onion Service should be redirected to `127.0.0.1:80`"*<br>
In other words: *"Anything running on `127.0.0.1` will be your **Hidden Service**"*


<hr>

# Hosting
There are two main ways to host a **Hidden Service**
1. "Manually" *(I don't know what to call it)*
     - More workable if you only want to leave it on for a while
2. Webhosting *(`Nginx` or `Apache`)*
     - more dedicated
     - Recommended

Restart the tor:
```sh
sudo systemctl restart tor
```

>The `/var/lib/hidden_service` directory is created when `tor` restarts * (so instead of `hidden_service` it can be whatever name you want)*

>In this file are the "keys" to your **Hidden Service**. They are used to give you access to your **Hidden Service**, anyone who has access to them can use your **Hidden Service** URL

Don't forget that **Hidden Service** won't work if your **Firewall** blocks `Apache`, even if everything is ok

To allow `Apache` access on the **Firewall**, type:
```sh
sudo ufw allow 'apache'
```

Then you can verify:
```sh
sudo ufw status
```

## "Manually"
The easiest way to do it this way *(in my opinion)* is to use `php`
```sh
sudo apt install php
```

Create a directory for your **Hidden Service** *(can be anywhere)*<br>
Add an `index.html` or `index.php` file and write something inside

Now run `php` in the directory where the `index` file is:
```sh
sudo php -S 127.0.0.1:80
```

See if it's working by going to the browser and typing in the URL `127.0.0.1`<br>
As you may remember, in the `torrc` file we put that *"Everything that runs at 127.0.0.1:80" will be what will run on the **Hidden Service**"*
>To change the port just put the desired port in `/etc/tor/torrc` and run the `php` application with the chosen port

Enter the command:
```sh
sudo -u debian-tor tor
```

>This basically tells the `debian-tor` user *(a user automatically created once tor is installed)* to run tor *(to run tor, you need to have access to the tor folders and only that user has access by standard)*


Now just open Tor Browser and enter the link of your **Hidden Service**<br>
To know the link of your **Hidden Service** type the command:
```sh
sudo cat /var/lib/tor/hidden_service # Replace "hidden_service" with the name you chose earlier
```

## Webhosting
In short, Webhosting is a service that allows you to put your website on the internet for other people to see

### Apache
First of all you need to set an `Apache` server to run on **port 80** *(default port)*<br>
>You can see my [Nginx tutorial](https://github.com/vaaako/Apache-Tutorial)

>You can also just run the `bash script` to create an `Nginx` server automatically, if you don't want to see the tutorial

Turn on `Apache`:
```sh
sudo systemctl start apache2
```
Check if it's working by typing `127.0.0.1` in your browser

Now just open **Tor Browser** and enter the link of your **Hidden Service**<br>
To know the link of your **Hidden Service** type the command:
```sh
sudo cat /var/lib/tor/hidden_service # Replace "hidden_service" with the name you chose earlier
```


Add your **Hidden Service** link to `/etc/apache2/sites-avaiable/your_domain.conf`<br>
Here is an example of what the `.conf` file of your **Hidden Service** should look like:
```html
<VirtualHost *:80>
    ServerName youronionaddress.onion
    DocumentRoot /var/www/your_hidden_service_root_directory # what a big name
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```


## Nginx
`Nginx` is the recommended for an **Hidden Service**. `Nginx` is faster, simpler and accepts sockets
>And `Nginx` is considered much more secure than `Apache` server

First of all you need to set an `Nginx` server to run on **port 80** *(default port)*<br>
>You can see my [Nginx tutorial](https://github.com/vaaako/Nginx-Tutorial)

<!-- >You can also just run the `bash script` to create an `Nginx` server automatically, if you don't want to see the tutorial -->

Turn on `Nginx`:
```sh
sudo systemctl start nginx
```
Check if it's working by typing `127.0.0.1` in your browser

To know the link of your **Hidden Service** type the command:
```sh
sudo cat /var/lib/tor/hidden_service # Replace "hidden_service" with the name you chose earlier
```


Add your **Hidden Service** link to `/etc/nginx/sites-avaiable/your_domain.conf`<br>
Here is an example of what the `.conf` file of your **Hidden Service** should look like:
```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain/html; # you can type just "/var/www/your_domain"
    index index.html
  
    server_name localhost 127.0.0.1 yourhiddenserviceaddres.onion # "localhost" and "127.0.0.1" are not required, here just for testing purposes (your addres shoudn't have "http://" or a "/" at the end)
   
    access_log /var/log/nginx/my-website.log;
}
```

Now just open **Tor Browser** and enter the link of your **Hidden Service**

<hr>

# PRECAUTIONS
### WARNING: REMEMBER THAT EVERYTHING RUNNING AT 127.0.0.1:80 WILL BE APPEARING IN YOUR HIDDEN SERVICE

## Changing ports
I recommend running your **Hidden Service** on another port, because the **port 80** is a very common port to have something running, I will show you how to change port using the **port 8080** as an example

### Nginx
In your `/etc/nginx/sites-avaiable/your_domain.conf` file change to:
```nginx
server {
    listen 8080; # Change here
    listen [::]:80; # And here
    ...
}
```

### Apache
In your `/etc/apache2/sites-avaiable/your_domain.conf` file change to:
```html
<VirtualHost *:8080> # Change here
    ...
</VirtualHost>
```

### torrc
Now in the file `/etc/tor/torrc` change to:
```
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080 # Change here
```


Now restart `Apache`/`Nginx` and `tor`:
```sh
sudo systemctl restart apache2/nginx # change it durrh
sudo systemctl restart tor
```

## Shutting down Tor and Apache
You can choose to turn off `tor` or `Apache/Nginx` by typing:
```sh
sudo systemctl stop apache2/nginx # or tor
```
>Obviously instead of *"`apache/nginx`"* type `apache2` **or** `nginx` *(depends on wich one you are using :P)*

If you just turn off `Apache/Nginx` your **Hidden Service** will also stop, since *(in theory)* nothing is running on the port that would be your **Hidden Service**

To turn on type:
```sh
sudo systemctl start apache2/nginx # or tor
```

However, when you restart your computer, `Apache/Nginx` and `tor` will start automatically<br>
To prevent this use the command:
```sh
sudo systemctl disable apache2/nginx # or tor
```

>Technically just turning off `Apache/Nginx` would solve the problem, for the same explanation above, however I recommend turning off either just `tor` or both

To turn on each service type:
```sh
sudo systemctl start apache2/nginx # or... aaah you get it
```

Now when you restart your computer, none of the services will turn on automatically<br>
To enable them back type:
```sh
sudo systemctl enable apache2/nginx
```

You know which is on/off and which is enabled/disabled by typing:
```sh
sudo systemctl status apache2/nginx
```

>If you are using the "Manual" *(I still don't know how to call it)*, just stop `php`
<br>

**Important:** Make sure nothing is running on the port you configured for the **Hidden Service** in the `/etc/tor/torrc` file *(Port 80 is the default)*
>You can use `nmap` to check this by typing: `sudo nmap localhost`

>*(Install `nmap` by typing: `sudo apt install nmap`)*


## conf/snippets
> `confs` (`Apache`) and `snippets` (`Nginx`) are configuration files, but some of them, are accessed as pages, `phpmysql` for example: `127.0.0.1/phpmyadmin`.

>If you have one enabled, the `conf/snippets` can also be accessed in the **Hidden Service**

It is also extremely important to remember to disable the `confs/snippets` of `Apache/Nginx`<br>
Like `phpmyadmin` for example *(probably the most common)*

You can check for `confs` enabled by typing:
```sh
sudo ls /etc/apache2/conf-enabled/
```

And you can check for `snippets` enabled by typing:
```sh
sudo ls /etc/nginx/snippets/
```


>**WARNING:** not all `confs/snippets` are accessed as pages *(none of the defaults for example)*, but it is important to pay attention to this

>default `confs` are: `charset.conf`, `security.conf`, `localized-error-pages.conf`, `serve-cgi-bin.conf`, `other-vhosts-access-log.conf`

>default `snippets` are: `fastcgi-php.conf`, `snakeoil.conf`

To disable a `conf` type:
```sh
sudo a2disconf /etc/apache2/conf-enabled/phpmyadmin.conf # change phpmyadmin.conf
```

To disable a `snippet` type:
```sh
# idk help (maybe you will never need to)
```

To enable a `conf` back:
```sh
sudo a2enconf /etc/apache2/conf-avaiable/phpmyadmin.conf # change phpmyadmin.conf
```

To enable a `snippet` back:?
```sh
# idk either help (i still think you won't need to)
```

<hr>

# Vanity
`Vanity` is a way to have a custom `.onion` URL *(not all of it, just the first few characters)*

## Installation
```sh
git clone https://github.com/cathugger/mkp224o.git
cd mkp224o
./autogen.sh
./configure # On AMD64 add --enable-amd64-51-30k
make
```

## Generating
The command "template" is:
```sh
sudo ./mkp224o -d [folder] [filters]
```

- Replace `[folder]` with the directory where you want the keys of the generated links to be stored<br>
- Replace `[filters]` with the Vanities you want to generate

Example:
```sh
sudo ./mkp224o -d onionkeys neko nekostore nekostr
```

`mkp224o` will try to generate a link that starts with `neko`, `nekostore` or `nekostr`

## Using
When you manage to generate a website, just copy it to your *(or one of your)*  **Hidden Service** directory

For example:
```sh
sudo cp -r onionkeys/neko54as6d54....onion /var/lib/tor/neko
```

>Nothing prevents you from creating a new directory

>I recommend creating it inside the `/var/lib/tor/` directory

You may need to adjust ownership and permissions:
```sh
sudo chown -R debian-tor: /var/lib/tor/neko
sudo chmod -R u+rwX,og-rwx /var/lib/tor/neko

# You are obviously going to replace "neko" with the name of your Hidden Service directory, I don't even know why I explain this yet, it should be obvious
```

>Remember, if you created a new directory, you need to edit the `torrc` file in `/etc/tor/torrc`


Now just restart `tor`:
```sh
sudo systemctl restart tor
```

Learn more about the tool by accessing the [mkp224o](https://github.com/cathugger/mkp224o)'s github *(it's important)*

## Performance
Time to Generate a `.onion` with a Given Number of Initial Characters on a **1.5Ghz Processor**

| Characters | Time to generate (approx.) |
|------------|----------------------------|
|      1     | less than 1 second         |
|      2     | less than 1 second         |
|      3     | less than 1 second         |
|      4     | 2 seconds                  |
|      5     | 1 minute                   |
|      6     | 30 minutes                 |
|      7     | 25 days                    |
|      8     | 25 days                    |
|      9     | 2.5 years                  |
|     10     | 40 years                   |
|     11     | 640 years                  |
|     12     | 10 millenia                |
|     13     | 160 millenia               |
|     14     | 2.6 million years          |

>From [Shallot](https://github.com/dendisuhubdy/Shallot)

<hr>

# Security
## Unix Socket
>It took me a VERY long time *(call me an idiot if you like)* to figure out how to do this, but I finally did and can bring you this tip

A good practice to avoid leaking an Onion Service to a local network is to run Onion Services over `Unix sockets` instead of a TCP socket

>I'll only be showing how to use `Unix Socket files` in the `Webhost` method

>**What is a `Unix Socket File`?**: A `Unix socket file` is a type of endpoint for inter-process communication (IPC) between processes on a Unix-like operating system. Unlike a network socket, it is not accessible over the network and is only accessible by processes on the same system. This makes `Unix socket files` a more secure way for processes to communicate, as it reduces the attack surface that could be exploited by an attacker over the network. `Unix socket files` are commonly used for local communication between server and client processes, such as in the case of a web server communicating with a CGI script.

### Nginx
Add the following line to `/etc/nginx/sites-avaiable/your_domain.conf`:

```
server {
    # Comment this two:
    #listen 80;
    #listen [::]:80;

    listen unix:/var/run/tor-mywebsite.sock; # <- add this one
    ...
}
```
>it can be whatever name you want

Restart `Nginx`:
```sh
sudo systemctl restart nginx
```

Give the file access privilege
```sh
sudo chmod 775 tor-mywebsite.sock
```

Restart `tor`:
```sh
sudo systemctl restart tor
```

>If you start or restart `Nginx` and the file already exists, `Nginx` will give an error and won't start, to solve this just delete the socket file as it is created when `Nginx` starts

### Apache
>This topic has not been tested yet

```html
Use Nginx for this, sorry
```


## Vanguards
*"This addon is designed to make it as hard as possible for someone to determine the IP address of an onion service. Success of an attack depends on the adversary's ability to run large portions of the Tor network in addition to being able to compromise honest nodes."*


## Installation
This whole process needs to be done using the `debian-tor` user *(a user that is created once `tor` is installed, this user has permission to access the `tor` files)*

However, some commands need *sudo permission* to run efficiently

For your happiness and for my sadness, I spent **3 hours** until I discovered this and could bring you the solution

Add the `debian-tor` user to the **sudo users** list:
```sh
sudo adduser debian-tor sudo
```

However, as you may know *(or should know)*, in order to use `sudo` commands you need to extract the user's `sudo` password, but I couldn't find the `sudo` password for `debian-tor`, so my alternative is to reset `debian-tor`'s password:
```sh
sudo passwd debian-tor
```
>Choose a strong password *(and different from your PC's default user)*

To "log in" as the `debian-tor` user type:
```sh
sudo -u debian-tor bash
```

>**WARNING:** `Vanguards` will only work if `for` is running, to run it on type: `sudo systemctl start tor`

There are **3 methods** of installation, however, the one that worked best for me was the **third**, so I'll only be showing the **third method**, if you want to check out the other two installation methods yourself you can check the github page of [**Vanguards**](https://github.com/mikeperry-tor/vanguards)

### Pip
To install the latest release via pip type:
```sh
sudo torsocks pip install vanguards
```

But if you want:
```sh
sudo torsocks pypy3 -m pip install vanguards
```


## Configuration
Before configuring `Vanguards` let's create a `Hash Control Password`<br>
Why is it important? *explanation I need to add later*<br>
And also because `Vanguards` can give an error if we don't have a

Create a password by typing:
```sh
tor --hash-password your_password
```

Copy it, then go to `/etc/tor/torrc` and add this line:
```sh
HashedControlPassword your_password # yes, with the 16:

# Example:
HashedControlPassword 16:05B7B9E8F3D0AB3160E030928F9517EDA5348ECD1CDCE2D95F0D230016
```

Now continuing...

This addon is primarily intended for onion service operators. To use it, set up your onion service to expose a control port listener using the `ControlPort` or `ControlSocket` `torrc` directives

Open `/etc/tor/torrc` and add to:
```
ControlPort 9099 # or ControlSocket /path/to/socket
CookieAuthentication 1
DataDirectory /var/lib/hidden_service # change "hidden_service" to you actual hidden_service directory's name
```

## Using
Just run it:
```sh
sudo vanguards --control_port 9099 #(or --control_socket path/to/socket)
```

Now you are protected by `Vanguards`!

If `Vanguards` doesn't work, it could be that `tor` is running on another port<br>
Use `nmap` *(as shown before)* to see what port `tor` is running on, if it's different from `9050`, open `/etc/tor/torsocks.conf` and change ` TorPort 9050` with `TorPort tor_port`<br>
*(obviously replacing "tor_port" with the port `tor` is running on, I don't even know why I still need to explain these things)*

Learn more about `Vanguards` on their [github](https://github.com/mikeperry-tor/vanguards) page and [here](https://blog.torproject.org/announcing-vanguards-add-onion-services/) *(it's important)*

## Onionscan
>It took me a VERY long time to find a version of `onionscan` that was not outdated, until I found this version on [Hack Guru's channel](https://www.youtube.com/watch?v=vXcTcSwdW08&pp=ugMICgJwdBABGAHKBQ1vbmlvbnNjYW4gdG9y) *(thank you so much)*


*"OnionScan is a free and open source tool for investigating the Dark Web. For all the amazing technological innovations in the anonymity and privacy space, there is always a constant threat that has no effective technological"*

`Onionscan` in short, is a tool to find security flaws in an `.onion` website<br>
Why would this be useful for your **Hidden Service**? For you to find security flaws and fix them!


## Installation
This version I'll be showing you how to download is a **fork** of the original version of `onionscan`, updated to **links v3**

## Github Release

```sh
wget https://github.com/CypherpunkSamurai/onionscan/releases/download/release-0.3/linux_i386
mv linux_i386 onionscan
sudo chmod 775 onionscan
./onionscan -h
```

>Depending on your **system** and **processor** you may need to change `linux_i386` *(at the end of the `wget` command)* to one of the options: `darwin_amd64`, `linux_amd64`, `windows_amd64`, ` windows_i386`,

You can move `onionscan` file to `/bin/` if you want to run anywhere:
```sh
sudo mv onionscan /bin/
sudo chmod 775 /bin/onionscan
onionscan -h
```

## Go Method
You will need to have `go` installed, for that type:
```sh
sudo apt install golang
```

After that install `onionscan` by typing:
```sh
go get github.com/CypherpunkSamurai/onionscan
go install github.com/CypherpunkSamurai/onionscan
```

>It will probably give an error if you have a version of `go` above `1.7`, so I preferred to do it with another method *(besides being faster)*


## Some commands
### Basic scan
```sh
sudo ./onionscan realhiddenservice.onion
```
>It can took **20 minutes** to complete the scan

### verbose
Print out a verbose log output of the scan
```sh
sudo ./onionscan -verbose realhiddenservice.onion
```

### jsonReport
Same as `verbose` but in `json` format
```sh
sudo ./onionscan -jsonReport realhiddenservice.onion
```

### torProxyAddress
If you would like to use a proxy server listening on something other than `127.0.0.1:9050`, then you can use the `--torProxyAddress` flag
```sh
sudo ./onionscan --torProxyAddress=127.0.0.1:9150 realhiddenservice.onion
```

See more about `onionscan` at [forked github](https://github.com/CypherpunkSamurai/onionscan), [original github](https://github.com/s-rah/onionscan) and [docs](https://github.com/CypherpunkSamurai/onionscan/blob/master/doc/README.md)

## Onion Browser
As this is not the focus of this tutorial, I will just be giving you some quick security tips for the **Tor Browser**, without explaining too much

First of all, `Vanguards` can be used together with the **Tor Browser**, check the [Vanguards] github (https://github.com/mikeperry-tor/vanguards)

One very important thing is that the Tor Browser does NOT come with all secure settings by default, even if you enable the *"More secure"* option in the settings

So you can click on [this link]() to see a PDF with all the options you need to configure *(The PDF is in English, I would rewrite it here, but it is too big and the Tor Browser is not the focus of this tutorial... maybe later)*


## Changing access privileges
#### WARNING: This topic is not complete, untested and subject to change


This tip is not as important as the others, but if you have a dedicated server it might be a good idea to run it

This means: every directory that is for the exclusive use of your **Hidden Service** should only be accessible to the `debian-tor` user * (`debian-tor` is a user created when `tor` is installed and is a user that has access to the `tor` files)

## Root Dir
Where your site files are stored
to change access type:
```sh
sudo chown -R debian-tor:debian-tor /var/www/your_domain
```

## Hidden Service Keys
Where your **Hidden Service** keys are located

>Hidden Service Keys are what give you permission to use your Hidden Service URL, so if someone else has access to them, they can use your URL

```sh
sudo chown -R debian-tor:debian-tor /var/lib/tor/hidden_service
```

## Socket
Only if you have a `Unix Socket file` for your **Hidden Service** *(You should if you don't)*

```sh
sudo chown -R debian-tor:debian-tor /var/run/tor-mywebsite.sock
```





**Author:** *Vako*<br>
**Last edit:** *23/04/2023*
