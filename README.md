# Project-3-Deploy-Matomo-Web-Analytics-on-Cloud
**[Matomo](https://matomo.org/faq/new-to-piwik/faq_13/)**, formerly known as Piwik, is an open-source web analytics software platform that is an alternative to Google Analytics.  It provides detailed reports on a website and its visitors, including the search engines and keywords they used, the language they speak, which pages they like, the files they download and so much more. 
Matomo is PHP MySQL software which you download and install on your own webserver. 

**Maria DB** is a drop-in replacement for the MySQL databases. It is one of the most popular open source relational databases. It’s made by the original developers of MySQL and guaranteed to stay open source. It is part of most cloud offerings and the default in most Linux distributions. It is built upon the values of performance, stability, and openness, and MariaDB Foundation ensures contributions will be accepted on technical merit. https://mariadb.org

**Docker Compose**  is a tool for defining and running multi-container Docker applications; It, also, uses YAML file to configure application services, using a single command to build, start, and stop containers.
Certbot https://certbot.eff.org

## Installing and enabling UFW firewall on Ubuntu 20.04 ##
Like we’ve done in previous projects, we will login to our AWS account and launch an EC2 instance with Ubuntu 20.04 server.
Detailed description of how to do that is available here https://github.com/skidodo2018/Web-Stack-Implementation and this Youtube video https://www.youtube.com/watch?v=jpOM7UC_p8I will also come in handy as it explains how to link your public IP to an Elastic IP.

```
If you plan on stopping and restarting your instance, it is important to link it to an Elastic IP. You don't need this step until you get to the part where you purchase a domain. I say this because Elastic IPs are only free if they are attached to a running AWS instance. Once your instance is stopped or terminated, the Elastic IP must be de-associated from the instance and released to avoid incurring costs.
```

Also, we will edit our security settings to allow HTTP traffic once the instance is up and running.

![](/images3/alltraffic.png)

### Step 1 — Logging in as root
To log into your server, you will need to know your server’s public IP address. You will also need the password or — if you installed an SSH key for authentication — the private key for the root  user’s account.
If you are not already connected to your server, log in now as the root user using the following command (substitute the highlighted portion of the command with your server’s public IP address):

`$ sudo su`

Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.
The root user is the administrative user in a Linux environment that has very broad privileges. Because of the heightened privileges of the root account, you are discouraged from using it on a regular basis. This is because the root account is able to make very destructive changes, even by accident.
The next step is setting up a new user account with reduced privileges for day-to-day use. Later, we’ll temporarily gain increased privileges for the times when they are needed.

### Step 2 — Creating a New User
Once you are logged in as **root**, you’ll be able to add the new user account. In the future, we’ll log in with this new account instead of **root**.

This example creates a new user called **betty**, but you should replace that with a username that you like:

`# adduser betty`

You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit `ENTER` in any field you wish to skip.

### Step 3 — Granting Administrative Privileges
Now we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the **root** account, we can set up what is known as *superuser* or **root** privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word `sudo` before the command.

To add these privileges to our new user, we need to add the user to the **sudo** group. By default, on Ubuntu 20.04, users who are members of the **sudo** group are allowed to use the `sudo` command.

As **root**, run this command to add your new user to the **sudo** group (substitute the highlighted username with your new user):

`usermod -aG sudo betty`

Now, when logged in as your regular user, you can type sudo before commands to run them with superuser privileges.

### Step 4 — Setting Up a Basic Firewall

Ubuntu 20.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall using this application.

Applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW.

You can see this by typing:

`# ufw app list`

```
Output
Available applications:
  OpenSSH
```
We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:

`# ufw allow OpenSSH`

Afterwards, we can enable the firewall by typing:

```bash
# ufw enable
```

Type `y`and press `ENTER`to proceed. You can see that SSH connections are still allowed by typing:

As **the firewall is currently blocking all connections except for SSH**, if you install and configure additional services, you will need to adjust the firewall settings to allow traffic in. 

![](/images3/adduser.png)

**Step 5 — Enabling External Access for Your Regular User**

Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

The process for configuring SSH access for the new user depends on whether our server’s **root**
 account uses a password or SSH keys for authentication.

**If the root Account Uses SSH Keys for Authentication**

If the **root** account *uses SSH keys*, as in the case of AWS EC2 instance, then we will have an error when we try to login as below. 

```bash
$ ssh betty@your_server_public_ip
```
![](/images3/ssherror.png)

**RESOLVING THE PERMISSION DENIED (publickey) ERROR**
To resolve the above error, we will modify the config file by using the following command in our Ubuntu server.

`$ sudo nano /etc/ssh/sshd_config`
Next, scroll through the file and make the following changes:

**Change `# PermitRootLogin prohibit-password` to the following:**

`# PermitRootLogin yes`

**Next, change `# PasswordAuthentication no` to the following:**

`# PasswordAuthentication yes`
![](/images3/sshd-config.png)

Once the changes are made, save the file, exit Nano and restart the server by running the following command:

`$ sudo service ssh restart`

Now you should be able to SSH with your new username. Try this command again:

`$ ssh betty@<your_server_public_ip>`

You will be prompted for your regular user password when using `sudo` for the first time each session (and periodically afterwards).

![](/images3/sshpwd.png)

To enhance our server’s security, **we strongly recommend setting up SSH keys instead of using password authentication**.

****How to Set Up SSH Keys on your PC****

SSH, or secure shell, is an encrypted protocol used to administer and communicate with servers. SSH keys provide a secure way of logging into your server and are recommended for all users.

**Step 1 — Creating the Key Pair**

The first step is to create a key pair on the client machine (usually your computer):

`$ ssh-keygen`

By default recent versions of `ssh-keygen` will create a 3072-bit RSA key pair, which is secure enough for most use cases (you may optionally pass in the `-b 4096` flag to create a larger 4096-bit key).

After entering the command, you should see the following output

```jsx
Output
Generating public/private rsa key pair.
Enter file in which to save the key (/your_home/.ssh/id_rsa):
```

Press enter to save the key pair into the `.ssh/` subdirectory in your home directory, or specify an alternate path.

If you had previously generated an SSH key pair, you may see the following prompt:

```jsx
Output
/home/your_home/.ssh/id_rsa already exists.
Overwrite (y/n)?
```

If you choose to overwrite the key on disk, you will **not** be able to authenticate using the previous key anymore. Be very careful when selecting yes, as this is a destructive process that cannot be reversed.

You should then see the following prompt:

```
Output
Enter passphrase (empty for no passphrase):

```
Here you optionally may enter a secure passphrase, which is highly recommended. A passphrase adds an additional layer of security to prevent unauthorized users from logging in. To learn more about security, 

You should then see the output similar to the following:

![](/images3/ssh-keygen.png)

You now have a public and private key that you can use to authenticate. The next step is to place the public key on your server so that you can use SSH-key-based authentication to log in.

**Step 2 — Copying the Public Key to Your Ubuntu Server**

The quickest way to copy your public key to the Ubuntu host is to use a utility called `ssh-copy-id`. Due to its simplicity, this method is highly recommended if available. If you do not have `ssh-copy-id` available on your client machine, you may use an alternate method (copying via password-based SSH, or manually copying the key).

### Copying the Public Key Using `ssh-copy-id`

The `ssh-copy-id` tool is included by default in many operating systems, so you may have it available on your local system. For this method to work, you must already have password-based SSH access to your server.

To use the utility, you specify the remote host that you would like to connect to, and the user account that you have password-based SSH access to. This is the account to which your public SSH key will be copied.

The syntax is:

`$ ssh-copy-id username@remote_host`

The utility will scan your local account for the id_rsa.pub key that we created earlier. When it finds the key, it will prompt you for the password of the remote user’s account:

Type in the password (your typing will not be displayed, for security purposes) and press `ENTER`. The utility will connect to the account on the remote host using the password you provided. It will then copy the contents of your `~/.ssh/id_rsa.pub` key into a file in the remote account’s home `~/.ssh` directory called `authorized_keys`.

You should see the following output:

![](/images3/sshkeyadd.png)

At this point, your `id_rsa.pub` key has been uploaded to the remote account. Y

### Step 3 — Authenticating to Your Ubuntu Server Using SSH Keys

If you have successfully copied the SSH key to your Ubuntu server, you should be able to log into the remote host *without* providing the remote account’s password.

The basic process is the same:

```
$ ssh username@remote_host
```

If this is your first time connecting to this host (if you used the last method above), you may see something like this:

```
Output
The authenticity of host '203.0.113.1 (203.0.113.1)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)?yes
```

This means that your local computer does not recognize the remote host. Type “yes” and then press `ENTER` to continue.

If you did not supply a passphrase for your private key, you will be logged in immediately. If you supplied a passphrase for the private key when you created the key, you will be prompted to enter it now (note that your keystrokes will not display in the terminal session for security). After authenticating, a new shell session should open for you with the configured account on the Ubuntu server.

![](/images3/sshkeylogin.png)

If key-based authentication was successful, continue on to learn how to further secure your system by disabling password authentication.

### Step 4 — Disabling Password Authentication on Your Server

If you were able to log into your account using SSH without a password, you have successfully configured SSH-key-based authentication to your account. However, your password-based authentication mechanism is still active, meaning that your server is still exposed to brute-force attacks.

Before completing the steps in this section, make sure that you either have SSH-key-based authentication configured for the **root** account on this server, or preferably, that you have SSH-key-based authentication configured for a non-root account on this server with `sudo` privileges. This step will lock down password-based logins, so ensuring that you will still be able to get administrative access is crucial.

Once you’ve confirmed that your remote account has administrative privileges, log into your remote server with SSH keys, either as **root** or with an account with `sudo` privileges. Then, open up the SSH daemon’s configuration file:

```
$ sudo nano /etc/ssh/sshd_config

```

Inside the file, search for a directive called `PasswordAuthentication`. This line may be commented out with a `#` at the beginning of the line. Uncomment the line by removing the `#`, and set the value to `no`. This will disable your ability to log in via SSH using account passwords:

/etc/ssh/sshd_config

```
. . .
PasswordAuthentication no
. . .

```

![](/images3/disablepwd.png)

Save and close the file when you are finished by pressing `CTRL+X`, then `Y` to confirm saving the file, and finally `ENTER` to exit nano. To actually activate these changes, we need to restart the `sshd` service:

```
$ sudo systemctl restart ssh

```

As a precaution, open up a new terminal window and test that the SSH service is functioning correctly before closing your current session:

Once you have verified your SSH service is functioning properly, you can safely close all current server sessions.

The SSH daemon on your Ubuntu server now only responds to SSH-key-based authentication. Password-based logins have been disabled.

You should now have SSH-key-based authentication configured on your server, allowing you to sign in without providing an account password.

## 2. Docker installed.

[Docker](https://www.docker.com/) is an application that simplifies the process of managing application processes in *containers*. Containers let you run your applications in resource-isolated processes. They’re similar to virtual machines, but containers are more portable, more resource-friendly, and more dependent on the host operating system.

In this section, you’ll install and use Docker Community Edition (CE) on your PC. You’ll install Docker itself, work with containers and images, and push an image to a Docker Repository.

### Prerequisites

To follow this tutorial, you will need the following:

- One Ubuntu 20.04 server set up  including a sudo non-root user and a firewall.
- An account on [Docker Hub](https://hub.docker.com/) if you wish to create your own images and push them to Docker Hub

### Step 1 — Installing Docker

The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:

```bash
$ sudo apt update

```

Next, install a few prerequisite packages which let `apt` use packages over HTTPS:

```bash
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common

```

Then add the GPG key for the official Docker repository to your system:

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

```

Add the Docker repository to APT sources:

```bash
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

```

This will also update our package database with the Docker packages from the newly added repo.

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:

```bash
$ apt-cache policy docker-ce

```

You’ll see output like this, although the version number for Docker may be different:

Output of apt-cache policy docker-ce

![](/images3/dockerrepo.png)

Notice that `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 20.04 (`focal`).

N:B: If nothing shows up, run `$sudo apt update` to update the system.

Finally, install Docker:

```bash
$ sudo apt install docker-ce

```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```bash
$ sudo systemctl status docker

```

The output should be similar to the following, showing that the service is active and running:

![](/images3/docker.png)

Installing Docker now gives you not just the Docker service (daemon) but also the `docker` command line utility, or the Docker client. We’ll explore how to use the `docker` command later in this tutorial.

### Step 2 — Executing the Docker Command Without Sudo (Optional)

By default, the `docker` command can only be run the **root** user or by a user in the **docker** group, which is automatically created during Docker’s installation process. If you attempt to run the `docker` command without prefixing it with `sudo` or without being in the **docker** group, you’ll get an output like this:

```bash
Output
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.

```

If you want to avoid typing `sudo` whenever you run the `docker` command, add your username to the `docker` group:

```bash
$ sudo usermod -aG docker ${USER}

```

To apply the new group membership, log out of the server and back in, or type the following:

```bash
$ su - ${USER}

```

You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the **docker** group by typing:

```bash
$ groups

```

![](/images3/dockergrp.png)

If you need to add a user to the `docker` group that you’re not logged in as, declare that username explicitly using:

```bash
$ sudo usermod -aG dockerusername
```

The rest of this article assumes you are running the `docker` command as a user in the **docker** group. If you choose not to, please prepend the commands with `sudo`.

Let’s explore the `docker` command next.

### Step 3 — Using the Docker Command

Using `docker` consists of passing it a chain of options and commands followed by arguments. The syntax takes this form:

```bash
$ docker [option] [command] [arguments]

```

To view all available subcommands, type:

To view the options available to a specific command, type:

```bash
$ dockerdocker-subcommand --help

```

To view system-wide information about Docker, use:

```bash
$ docker info

```

Let’s explore some of these commands. We’ll start by working with images.

### Step 4 — Working with Docker Images

Docker containers are built from Docker images. By default, Docker pulls these images from [Docker Hub](https://hub.docker.com/), a Docker registry managed by Docker, the company behind the Docker project. Anyone can host their Docker images on Docker Hub, so most applications and Linux distributions you’ll need will have images hosted there.

To check whether you can access and download images from Docker Hub, type:

```bash
$ docker run hello-world

```

The output will indicate that Docker is working correctly:

![](/images3/helloworld.png)

Docker was initially unable to find the `hello-world` image locally, so it downloaded the image from Docker Hub, which is the default repository. Once the image downloaded, Docker created a container from the image and the application within the container executed, displaying the message.

You can search for images available on Docker Hub by using the `docker` command with the `search` subcommand. For example, to search for the Ubuntu image, type:

```bash
$ docker search ubuntu

```

The script will crawl Docker Hub and return a listing of all images whose name match the search string. In this case, the output will be similar to this:

![](/images3/dockersearch.png)

In the OFFICIAL column, OK indicates an image built and supported by the company behind the project. Once you’ve identified the image that you would like to use, you can download it to your computer using the pull subcommand.

Execute the following command to download the official ubuntu image to your computer:

`$ docker pull ubuntu`

You’ll see the following output:

![](/images3/dockerpull.png)

After an image has been downloaded, you can then run a container using the downloaded image with the `run` subcommand. As you saw with the `hello-world` example, if an image has not been downloaded when `docker` is executed with the `run` subcommand, the Docker client will first download the image, then run a container using it.

To see the images that have been downloaded to your computer, type:

```bash
$ docker images

```

The output will look similar to the following:

![](/images3/dockerimages.png)

As you’ll see later in this project, images that you use to run containers can be modified and used to generate new images, which may then be uploaded (*pushed* is the technical term) to Docker Hub or other Docker registries.

Let’s look at how to run containers in more detail.

### Step 5 — Running a Docker Container

The `hello-world` container you ran in the previous step is an example of a container that runs and exits after emitting a test message. Containers can be much more useful than that, and they can be interactive. After all, they are similar to virtual machines, only more resource-friendly.

As an example, let’s run a container using the latest image of Ubuntu. The combination of the **-i** and **-t** switches gives you interactive shell access into the container:

```bash
$ docker run -it ubuntu

```

Your command prompt should change to reflect the fact that you’re now working inside the container and should take this form:

```bash
Output
root@dad60d1b9308:/#

```

Note the container id in the command prompt. In this example, it is `dad60d1b9308`. You’ll need that container ID later to identify the container when you want to remove it.

Now you can run any command inside the container. For example, let’s update the package database inside the container. You don’t need to prefix any command with `sudo`, because you’re operating inside the container as the **root** user:

```bash
$ apt update

```

Then install any application in it. Let’s install Node.js:

```bash
$ apt install nodejs

```

This installs Node.js in the container from the official Ubuntu repository. When the installation finishes, verify that Node.js is installed:

```bash
$ node -v

```

You’ll see the version number displayed in your terminal:

```bash
Output
v10.19.0

```

Any changes you make inside the container only apply to that container.

To exit the container, type `exit` at the prompt.

Let’s look at managing the containers on our system next.

### Step 6 — Managing Docker Containers

After using Docker for a while, you’ll have many active (running) and inactive containers on your computer. To view the **active ones**, use:

```bash
$ docker ps

```

You will see output similar to the following:

![](/images3/ps.png)

In this project, you started two containers; one from the `hello-world` image and another from the `ubuntu` image. Both containers are no longer running, but they still exist on your system.

To view all containers — active and inactive, run `docker ps` with the `-a` switch:

```bash
$ docker ps -a

```

You’ll see output similar to this:

![](/images3/ps-a.png)

To view the latest container you created, pass it the `-l` switch:

```bash
$ docker ps -l
```

![](/images3/ps-l.png)

To start a stopped container, use `docker start`, followed by the container ID or the container’s name. Let’s start the Ubuntu-based container with the ID of `dad60d1b9308`:

```bash
$ docker start dad60d1b9308
```

The container will start, and you can use `docker ps` to see its status:

![](/images3/dockerps.png)

To stop a running container, use `docker stop`, followed by the container ID or name. This time, we’ll use the name that Docker assigned the container, which is `quizzical_mcnulty`:

```bash
$ docker stop optimistic_moore
```

Once you’ve decided you no longer need a container anymore, remove it with the `docker rm` command, again using either the container ID or the name. Use the `docker ps -a` command to find the container ID or name for the container associated with the `hello-world` image and remove it.

```bash
$ docker rm thirsty_euclid
```
![](/images3/thirsty.png)

You can start a new container and give it a name using the `--name` switch. You can also use the `--rm` switch to create a container that removes itself when it’s stopped. See the `docker run help` command for more information on these options and others.

Containers can be turned into images which you can use to build new containers. Let’s look at how that works.

### Step 7 — Committing Changes in a Container to a Docker Image

When you start up a Docker image, you can create, modify, and delete files just like you can with a virtual machine. The changes that you make will only apply to that container. You can start and stop it, but once you destroy it with the `docker rm` command, the changes will be lost for good.

This section shows you how to save the state of a container as a new Docker image.

After installing Node.js inside the Ubuntu container, you now have a container running off an image, but the container is different from the image you used to create it. But you might want to reuse this Node.js container as the basis for new images later.

Then commit the changes to a new Docker image instance using the following command.

```bash
$ docker commit -m "What you did to the image" -a "Author Name"container_idrepository/new_image_name
```

The **-m** switch is for the commit message that helps you and others know what changes you made, while **-a** is used to specify the author. The `container_id` is the one you noted earlier in the tutorial when you started the interactive Docker session. Unless you created additional repositories on Docker Hub, the `repository` is usually your Docker Hub username.

For example, for the user **betty**, with the container ID of `dad60d1b9308`, the command would be:

```bash
$ docker commit -m "added Node.js" -a "betty" dad60d1b9308 betty/ubuntu-nodejs

```

When you *commit* an image, the new image is saved locally on your computer. Later in this tutorial, you’ll learn how to push an image to a Docker registry like Docker Hub so others can access it.

Listing the Docker images again will show the new image, as well as the old one that it was derived from:

```bash
$ docker images

```

You’ll see output like this:

![](/images3/docker-commit.png)

In this example, `ubuntu-nodejs` is the new image, which was derived from the existing `ubuntu` image from Docker Hub. The size difference reflects the changes that were made. And in this example, the change was that NodeJS was installed. So next time you need to run a container using Ubuntu with NodeJS pre-installed, you can just use the new image.

You can also build Images from a `Dockerfile`, which lets you automate the installation of software in a new image. However, that’s outside the scope of this tutorial.

Now let’s share the new image with others so they can create containers from it.

## Step 8 — Pushing Docker Images to a Docker Repository

The next logical step after creating a new image from an existing image is to share it with a select few of your friends, the whole world on Docker Hub, or other Docker registry that you have access to. To push an image to Docker Hub or any other Docker registry, you must have an account there.

This section shows you how to push a Docker image to Docker Hub. To learn how to create your own private Docker registry, **GOOGLE!**

To push your image, first log into Docker Hub.

```bash
docker login -u docker-registry-username
```
where docker-registry-username = the username you registered with on hub.docker.com

You’ll be prompted to authenticate using your Docker Hub password. If you specified the correct password, authentication should succeed.

**Note:** If your Docker registry username is different from the local username you used to create the image, you will have to tag your image with your registry username. For the example given in the last step, you would type:

```bash
$ docker tag betty/ubuntu-nodejs skidodo2018/ubuntu-nodejs

```

Then you may push your own image using:

```bash
$ docker push docker-registry-username/docker-image-name
```

To push the `ubuntu-nodejs` image to the **betty** repository, the command would be:

```bash
docker push skidodo2018/ubuntu-nodejs
```

The process may take some time to complete as it uploads the images, but when completed, the output will look like this:

![](/images3/dockerpush.png)

After pushing an image to a registry, it should be listed on your account’s dashboard, like that show in the image below.

![](/images3/dockerhub.JPG)

Log in with `docker login` and repeat the push attempt. Then verify that it exists on your Docker Hub repository page.

You can now use `docker pull betty/ubuntu-nodejs` to pull the image to a new machine and use it to run a new container.

## 3. Docker Compose installed.

Docker simplifies the process of managing application processes in containers. While containers are similar to virtual machines in certain ways, they are more lightweight and resource-friendly. This allows developers to break down an application environment into multiple isolated services.

For applications depending on several services, orchestrating all the containers to start up, communicate, and shut down together can quickly become unwieldy. [Docker Compose](https://docs.docker.com/compose/) is a tool that allows you to run multi-container application environments based on definitions set in a YAML file. It uses service definitions to build fully customizable environments with multiple containers that can share networks and data volumes.

In this guide, we’ll demonstrate how to install Docker Compose on an Ubuntu 20.04 server and how to get started using this tool.

### Prerequisites

You will need:

- Access to an Ubuntu 20.04 local machine or development server as a non-root user with sudo privileges. If you’re using a remote server, it’s advisable to have an active firewall installed.
- Docker installed on your server or local machine

To make sure we obtain the most updated stable version of Docker Compose, we’ll download this software from its [official Github repository](https://github.com/docker/compose).

First, confirm the latest version available in their [releases page](https://github.com/docker/compose/releases). At the time of this writing, the most current stable version is `1.27.4`.

The following command will download the `1.27.4` release and save the executable file at `/usr/local/bin/docker-compose`, which will make this software globally accessible as `docker-compose`:

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

Next, set the correct permissions so that the `docker-compose` command is executable:

```bash
$ sudo chmod +x /usr/local/bin/docker-compose

```

To verify that the installation was successful, you can run:

```bash
$ docker-compose --version

```

You’ll see output similar to this:

```bash
Output
docker-compose version1.27.4, build 40524192

```

Docker Compose is now successfully installed on your system. In the next section, we’ll see how to set up a `docker-compose.yml` file and get a containerized environment up and running with this tool.

### Step 2 — Setting Up a `docker-compose.yml` File

To demonstrate how to set up a `docker-compose.yml` file and work with Docker Compose, we’ll create a web server environment using the official [Nginx image](https://hub.docker.com/_/nginx) from [Docker Hub](https://hub.docker.com/), the public Docker registry. This containerized environment will serve a single static HTML file.

Start off by creating a new directory in your home folder, and then moving into it:

```bash
$ mkdir ~/compose-demo
cd ~/compose-demo
```

In this directory, set up an application folder to serve as the document root for your Nginx environment:

```bash
$ mkdir app
```

Using your preferred text editor, create a new `index.html` file within the `app` folder:

```bash
$ nano app/index.html

```

Place the following content into this file:

~/compose-demo/app/index.html

```bash
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Docker Compose Demo</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

	<h1>This is a Docker Compose Demo Page.</h1>
	<p>This content is being served by an Nginx container.</p>

</body>
</html>

```

Save and close the file when you’re done. If you are using `nano`, you can do that by typing `CTRL+X`, then `Y` and `ENTER` to confirm.

Next, create the `docker-compose.yml` file:

```bash
$ nano docker-compose.yml

```

Insert the following content on your `docker-compose.yml` file:

docker-compose.yml

```bash
version: '3.7'
services:
  web:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./app:/usr/share/nginx/html

```

The `docker-compose.yml` file typically starts off with the `version` definition. This will tell Docker Compose which configuration version we’re using.

We then have the `services` block, where we set up the services that are part of this environment. In our case, we have a single service called `web`. This service uses the `nginx:alpine` image and sets up a port redirection with the `ports` directive. All requests on port `8000` of the **host** machine (the system from where you’re running Docker Compose) will be redirected to the `web` container on port `80`, where Nginx will be running.

The `volumes` directive will create a [shared volume](https://docs.docker.com/compose/compose-file/#volumes) between the host machine and the container. This will share the local `app` folder with the container, and the volume will be located at `/usr/share/nginx/html` inside the container, which will then overwrite the default document root for Nginx.

Save and close the file.

We have set up a demo page and a `docker-compose.yml` file to create a containerized web server environment that will serve it. In the next step, we’ll bring this environment up with Docker Compose.

### Step 3 — Running Docker Compose

With the `docker-compose.yml` file in place, we can now execute Docker Compose to bring our environment up. The following command will download the necessary Docker images, create a container for the `web` service, and run the containerized environment in background mode:

```bash
$ docker-compose up -d

```

Docker Compose will first look for the defined image on your local system, and if it can’t locate the image it will download the image from Docker Hub. You’ll see output like this:

![](/images3/dk-compose.png)

Your environment is now up and running in the background. To verify that the container is active, you can run:

```bash
$ docker-compose ps

```

This command will show you information about the running containers and their state, as well as any port redirections currently in place:

![](/images3/dk-composeps.png)

You can now access the demo application by pointing your browser to either `localhost:8000` if you are running this demo on your local machine, or `your_server_domain_or_IP:8000` if you are running this demo on a remote server.

You’ll see a page like this:

![](/images3/dk-compose-pg.png)

Because the shared volume you’ve set up within the `docker-compose.yml` file keeps your `app` folder files in sync with the container’s document root. If you make any changes to the `index.html` file, they will be automatically picked up by the container and thus reflected on your browser when you reload the page.

In the next step, you’ll see how to manage your containerized environment with Docker Compose commands.

### Step 4 — Getting Familiar with Docker Compose Commands

You’ve seen how to set up a `docker-compose.yml` file and bring your environment up with `docker-compose up`. You’ll now see how to use Docker Compose commands to manage and interact with your containerized environment.

To check the logs produced by your Nginx container, you can use the `logs` command:

```bash
$ docker-compose logs

```

You’ll see output similar to this:

![](/images3/dk-logs.png)

If you want to pause the environment execution without changing the current state of your containers, you can use:

```bash
$ docker-compose pause

```

```bash
Output
Pausingcompose-demo_web_1 ... done

```

To resume execution after issuing a pause:

```bash
$ docker-compose unpause

```

```bash
Output
Unpausing compose-demo_web_1 ... done

```

The `stop` command will terminate the container execution, but it won’t destroy any data associated with your containers:

```bash
$ docker-compose stop

```

```bash
Output
Stopping compose-demo_web_1 ... done

```

If you want to remove the containers, networks, and volumes associated with this containerized environment, use the `down` command:

```bash
$ docker-compose down

```

```bash
Output
Removingcompose-demo_web_1 ... done
Removing networkcompose-demo_default
```

Notice that this won’t remove the base image used by Docker Compose to spin up your environment (in our case, `nginx:alpine`). This way, whenever you bring your environment up again with a `docker-compose up`, the process will be much faster since the image is already on your system.

In case you want to also remove the base image from your system, you can use:

```bash
$ docker image rm nginx:alpine
```

![](/images3/dk-img-rm.png)

We’ve seen how to install Docker Compose and set up a containerized environment based on an Nginx web server image. We’ve also seen how to manage this environment using Compose commands.

Finally, to enable SSL you’ll need a domain name pointed at your server’s public IP address. This should be something like `example.com` or `matomo.example.com.`

Create a new domain from [https://freenom.com](https://freenom.com) 

Adding a domain you own to your cloud account lets you manage the domain’s DNS records with the control panel and API. Domains you manage on your cloud account also integrate with Load Balancers and Spaces to streamline automatic SSL certificate management.

To set up a domain with your cloud account, you need to add it (and any of its existing DNS records) to the control panel, then delegate the domain by updating your registrar to use your domain name servers.

1. From the control panel, click **Create** in the top right, then click **Domains/DNS**.
2. In the **Enter Domain** section, enter the domain name.
    
    This is typically the apex domain, such as `example.com`. To add subdomains, like `www.example.com` or `images.example.com`, create DNS records for them after you add the apex domain.
    
3. Click **Add Domain**. This takes you to the **Create new record** page and adds NS records for the domain on cloud name servers.
4. On the **Create new record** page, add any DNS records for the domain. For each record, select the record type, fill in the necessary data, and click **Create Record**.
5. Delegate your domain by pointing your domain name to cloud name servers through your registrar.

When you’ve satisfied all the prerequisites, proceed to **Step 1**, where you’ll download and launch the Matomo software.

## Step 1 — Running Matomo and MariaDB with Docker Compose

Your first step will be to create the Docker Compose configuration that will launch containers for both the Matomo app and a MariaDB database.

This section will put your configuration inside a `matomo` directory in your home directory. You could also choose to work in an `/opt/matomo` directory or some other directory of your choosing.

First ensure you’re in your home directory:

```bash
cd ~

```

Then create the `matomo` directory and `cd` into it:

```bash
mkdir matomo
cd matomo

```

Now open a new blank YAML file called `docker-compose.yml`:

```bash
nano docker-compose.yml

```

This is the configuration file that the `docker-compose` software will read when bringing up your containers. Paste the following into the file:

docker-compose.yml

```bash
version: "3"

services:
  db:
    image: mariadb
    command: --max-allowed-packet=64MB
    restart: always
    environment:
      - MARIADB_DATABASE=matomo
      - MARIADB_USER
      - MARIADB_PASSWORD
      - MARIADB_ROOT_PASSWORD
    volumes:
      - ./db:/var/lib/mysql

  app:
    image: matomo
    restart: always
    volumes:
      - ./matomo:/var/www/html
    ports:
      - 127.0.0.1:8080:80

```

The file defines two `services`, one `db` service which is the MariaDB container, and an `app` service which runs the Matomo software. Both services also reference a named volume where they store some data, and the `app` service also opens up port `8080` on the loopback (`127.0.0.1`) interface, which we’ll connect to via `localhost`.

Save the file and exit your text editor to continue. In `nano`, press `CTRL+O` then `ENTER` to save, then `CTRL+X` to exit.

The MariaDB container needs some configuration to be passed to it through environment variables in order to function. The `docker-compose.yml` file lists these environment variables, but not all of them have associated values. That’s because it’s good practice to keep passwords out of your `docker-compose.yml` file, especially if you’ll be committing it to a Git repository or other source control system.

Instead, we’ll put the necessary information in a `.env` file in the same directory, which the `docker-compose` command will automatically load when we start our containers.

Open a new `.env` file with `nano`:

```bash
nano .env

```

You’ll need to fill in a user name and password, as well as a strong password for the MariaDB **root** superuser account:

.env

```bash
MARIADB_USER=matomo
MARIADB_PASSWORD=a_strong_password_for_user
MARIADB_ROOT_PASSWORD=a_strong_password_for_root
```

One way of generating a strong password is to use the `openssl` command, which should be available on most any operating system. The following command will print out a random 30 character hash that you can use as a password:

```bash
openssl rand 30 | base64 -w 0 ; echo

```

When you’re done filling out the information in your `.env` file, save it and exit your text editor.

You’re now ready to bring up the two containers with `docker-compose`:

```bash
sudo docker-compose up -d

```

The `up` subcommand tells `docker-compose` to start the containers (and volumes and networks) defined in the `docker-compose.yml` file, and the `-d` flag tells it to do so in the background (“daemonize”) so the command doesn’t take over your terminal. `docker-compose` will print some brief output as it starts the containers:

```bash
Output
Creating matomo_db_1  ... done
Creating matomo_app_1 ... done

```

When that’s done, Matomo should be running. You can test that a webserver is running at `localhost:8080` by fetching the homepage using the `curl` command:

```bash
curl --head http://localhost:8080

```

This will print out only the HTTP headers from the response:

![](/images3/http-resp.png)

The `200 OK` response means the Matomo server is up and running, but it’s only available on `localhost`. The highlighted `X-Matomo-Request-Id` header indicates that the server is Matomo and not something else that might be configured to listen on port 8080. Next we’ll set up Nginx to proxy public traffic to the Matomo container.

## Step 2 — Installing and Configuring Nginx

Putting a web server such as Nginx in front of your Matomo server can improve performance by offloading caching, compression, and static file serving to a more efficient process. We’re going to install Nginx and configure it to *[reverse proxy](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)* requests to Matomo, meaning it will take care of handing requests from your users to Matomo and back again. Using a non-containerized Nginx will also make it easier to add Let’s Encrypt SSL certificates in the next step.

First, refresh your package list, then install Nginx using `apt`:

```bash
sudo apt update
sudo apt install nginx

```

Allow public traffic to ports `80` and `443` (HTTP and HTTPS) using the “Nginx Full” UFW application profile:

```bash
sudo ufw allow "Nginx Full"

```

```bash
Output
Rule added
Rule added (v6)

```

Next, open up a new Nginx configuration file in the `/etc/nginx/sites-available` directory. We’ll call ours `matomo.conf` but you could use a different name:

```bash
sudo nano /etc/nginx/sites-available/matomo.conf

```

Paste the following into the new configuration file, being sure to replace `your_domain_here` with the domain that you’ve configured to point to your Matomo server. This should be something like `matomo.example.com`, for instance:

/etc/nginx/sites-available/matomo.conf

```bash
server {
    listen       80;
    listen       [::]:80;
    server_nameyour_domain_here;

    access_log  /var/log/nginx/matomo.access.log;
    error_log   /var/log/nginx/matomo.error.log;

    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Proto https;
      proxy_pass http://localhost:8080;
  }
}

```

This configuration is HTTP-only for now, as we’ll let Certbot take care of configuring SSL in the next step. The rest of the config sets up logging locations and then passes all traffic, as well as some important proxy headers, along to `http://localhost:8080`, the Matomo container we started up in the previous step.

Save and close the file, then enable the configuration by linking it into `/etc/nginx/sites-enabled/`:

```bash
sudo ln -s /etc/nginx/sites-available/matomo.conf /etc/nginx/sites-enabled/

```

Use `nginx -t` to verify that the configuration file syntax is correct:

```bash
sudo nginx -t

```

```bash
Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```

And finally, reload the `nginx` service to pick up the new configuration:

```bash
sudo systemctl reload nginx

```

Your Matomo site should now be available on plain HTTP. Load `http://your_domain_here` (you may have to click through a security warning) and it will look like this:

![](/images3/matomo-site.png)

Now that you have your site up and running over HTTP, it’s time to secure the connection with Certbot and Let’s Encrypt certificates. You should do this *before* going through Matomo’s web-based setup procedure.

## Step 3 — Installing Certbot and Setting Up SSL Certificates

Thanks to Certbot and the Let’s Encrypt free certificate authority, adding SSL encryption to our Matomo app will take only two commands.

First, install Certbot and its Nginx plugin:

```bash
sudo apt install certbot python3-certbot-nginx

```

Next, run `certbot` in `--nginx` mode, and specify the same domain you used in the Nginx `server_name` config:

```bash
sudo certbot --nginx -dyour_domain_here
```

You’ll be prompted to agree to the Let’s Encrypt terms of service, and to enter an email address.

Afterwards, you’ll be asked if you want to redirect all HTTP traffic to HTTPS. It’s up to you, but this is generally recommended and safe to do.

After that, Let’s Encrypt will confirm your request and Certbot will download your certificate:

![](/images3/SSLcert.png)

Certbot will automatically reload Nginx to pick up the new configuration and certificates. Reload your site and it should switch you over to HTTPS automatically if you chose the redirect option.

Your site is now secure and it’s safe to continue with the web-based setup steps.

## Step 4 — Setting Up Matomo

Back in your web browser you should now have Matomo’s **Welcome!** page open via a secure `https://` connection. Now you can enter usernames and passwords safely to complete the installation process.

Click the **Next** button. You’ll be taken to the **System Check** step:

![](/images3/syscheck.png)

This is a summary of the system Matomo is running on, and everything should be green checkmarks indicating there are no problems. Scroll all the way to the bottom and click the **Next** button.

Now you’ll be on the **Database Setup** page.

Continue the setup and click the Next button until you reach the Congratulations page:

![](/images3/congrats.png)

The information you fill in on this page will tell the Matomo application how to connect to the MariaDB database. You’ll need the `MARIADB_USER` and `MARIADB_PASSWORD` that you chose in Step 1. You can copy them out of your `.env` file if you need to.

Fill out the first four fields:

- **Database Server:** db
- **Login:** the username you set in the `MARIADB_USER` environment variable
- **Password:** the password you set in the `MARIADB_PASSWORD` environment variable
- **Database Name:** matomo

The defaults are fine for the remaining two fields.

Click **Next** once more. You’ll get a confirmation that the database was set up correctly. Click **Next** again. You’ll then need to set up an admin user, and finally you’ll set up information about the first website you want to collect analytics for.

After all that, you should end up on step 8, a **Congratulations** page. You’re almost all done. Scroll down to the bottom and click the **Continue to Matomo** button, and you’ll be taken to the homepage:

![](/images3/homepage-warning.png)

There will be a large warning at the top of the page. There’s a small update you’ll need to do to Matomo’s configuration file to finish up this process.

Back on the command line, open up the configuration file with a text editor:

```bash
sudo nano matomo/config/config.ini.php

```

Near the top you should have a `[General]` section. Add the last three lines, highlighted below, to the end of that section:

config.ini.php

```bash
[General]
proxy_client_headers[] = "HTTP_X_FORWARDED_FOR"
proxy_host_headers[] = "HTTP_X_FORWARDED_HOST"
salt = "e0a81d6e54d6d2200efd0f0ef6ef8563"
trusted_hosts[] = "localhost"
trusted_hosts[] = "example.com"
trusted_hosts[] = "localhost:8080"
assume_secure_protocol = 1
force_ssl = 1
```

These options let Matomo know that it’s safe to use port `8080`, and that it should assume it’s always being accessed over a secure connection.

Save and close the configuration file, then switch back to your browser and reload the page. The error should be gone, and you’ll be presented with a login prompt:

![](/images3/sign-in.png)

Log in with the admin account you created during setup, and you should be taken to the dashboard:

![](/images3/pagesetup.png)

Because you’ve probably not set up your tracking code yet, the dashboard will indicate that no data has been recorded. Follow the instructions to finish setting up the JavaScript code on your website to start receiving analytics data.

## Conclusion

In this project, we launched the Matamo analytics app and a MariaDB database using Docker Compose, then set up an Nginx reverse proxy and secured it using Let’s Encrypt SSL certificates.

