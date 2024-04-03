# Linux Assignment 3

# Install all Necessary Software
To Begin we need to install all the necessary software. This includes updating the system, installing Vim (if not already installed which it is on Arch Linux), and installing Nginx.

## Update the System using pacman
We need to make sure we have all the software up to date before we can install new software. Pacman is used to install, update, and remove software packages.

Run the following command to update all currently installed pacman packages:

```bash
sudo pacman -Syu
```
Note: -Syu tells pacman to upgrade installed packages and their dependencies

## Install Vim
Note: You can use any text editor
Vim is a text editor already installed on Arch Linux. If you don't have vim, you will need to install it. Run the following command to install Vim:
```bash
sudo pacman -S vim
```
-S indicates that we are installing a package with its dependencies

## Install nginx
Nginx is a commonly used web serverand reverse proxy that is used to serve web pages. 

To install Nginx, run the following command:
```bash
sudo pacman -S nginx
```

##  Start and enable nginx
To start the Nginx service, you need to use the systemctl command. This command is used to start, stop, and manage services in Linux.

 Services are programs that operate in the background to allow you to interact seamlessly with your device.

Run the following commands to start and enable the Nginx service:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
Then check if the service is running successfully.

Run the following command to check the status of the Nginx service:
```bash
sudo systemctl status nginx
```
This command will show you the status of the Nginx service. If the service is running, you will see an active status as well as enabled status.


# Configuring Nginx and Serving a Web Page

## Create the directories 

We will create directories where we will store our files for our project and be the base for our Git repo.

The directories which store the HTML files that Nginx will serve should be located in the root so make sure to cd to the root directory before creating the directories.

Once in the root, run the following command to create the directories:
Note: you can name the directories anything you want.
```bash
mkdir -p web/html/nginx-2420
```

### Check if the directories were created successfully
To ensure that the web directory and its subdirectories were created successfully, run the following command:
```bash
ls web/html/nginx-2420
```
if the directories were not created, you will see an error message otherwise it will return empty.

You can also ls the root directory to see if the web directory was created successfully by using grep:
```bash
ls | grep web
```

Next, in the created nginx-2420 directory, create the index.html file that will be served by Nginx. This html file **must** be called index.html.

Run the following command to create and edit the HTML file:
```bash
#use this command if you are in the root directory
vim web/html/nginx-2420/index.html
#else use this command
vim /web/html/nginx-2420/index.html
```

### Add HTML content to the file
Add the following HTML content to the file using Vim (provided by the instructor):
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        * {
            color: #db4b4b;
            background: #16161e;
        }
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
            font-family: sans-serif;
        }
    </style>
</head>
<body>
    <h1>All your base are belong to us</h1>
</body>
</html>
```

Make sure the file has the right permissions to be served by Nginx. Run the following command to change the permissions of the file:
```bash
sudo chmod 755  web/html/nginx-2420-tuesday-practise/index.html
```

## Create a new server block
Now you need to create a new conf file for the new server block. The server block is a configuration block that defines the settings for a specific domain. It is located in the /etc/nginx directory.

Note: we will not be using the default conf file. We will be creating a new one for this project. The name of the conf file is up to you. I recommend using the the same name as the directory you created earlier.

Since we are creating a new server block, we need to create two new directories in the /etc/nginx directory to store the new conf file.

/etc/nginx/sites-available: This directory stores the configuration files for all the server blocks that Nginx can use. The files in this directory are not active and need to be linked to the sites-enabled directory to be used by Nginx.
/etc/nginx/sites-enabled: This directory stores the configuration files that are currently active and being used by Nginx. The files in this directory are symbolic links to the files in the sites-available directory.

Run the following commands to create the directories:
```bash
sudo mkdir -p /etc/nginx/sites-available
sudo mkdir -p /etc/nginx/sites-enabled
```

Then in the sites-available directory, create a new conf file for the new server block. Run the following command to create and edit the conf file:
```bash
sudo vim /etc/nginx/sites-available/nginx-2420.conf
```
Note: you have to use sudo to edit the file because it is located in the /etc/nginx directory which requires root permissions to edit.

To create a new server block, enter the following information into the conf file:
```bash
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    root /web/html/nginx-2420;
    location / {
        index index.html;
    }
}
```

#### break down of the above code:
- listen: This directive specifies the port number that Nginx will listen on for incoming HTTP requests. In this case, we are using port 80.
- server_name: This directive specifies the domain name or IP address that should match the server block. In this case, we are using localhost.
- root: This directive specifies the root directory where Nginx should look for files to serve. In this case, we are using the path to the HTML directory created earlier.
- index: This directive specifies the default file that Nginx should serve when a directory is requested. In this case, we are using index.html which is the file created earlier.

## Enable the new server block
In the original Nginx configuration file, there is a directive called http, at the bottom of the directive, add the following line to include all the conf files in the sites-enabled directory:
```bash
include /etc/nginx/sites-enabled/*;
```
Reminder: Comment out the server block in the default conf file to avoid conflicts with the new server block.

To enable the new server block, you need to create a symbolic link from the sites-available directory to the sites-enabled directory. This will make the new server block active and available for Nginx to use.
Run the following command to create the symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/nginx-2420.conf /etc/nginx/sites-enabled/
```

## Reload/Restart  Nginx
use “sudo systemctl restart nginx” to restart the service to allow changes to take effect.
Note: you can also use “sudo systemctl reload nginx” to reload the service without restarting it. This is useful when you want to apply changes to the configuration file without interrupting the service.

```bash
sudo systemctl restart nginx
sudo systemctl reload nginx
```

### Check Status
To ensure that the reload was successful, run the following command to check the status of the Nginx service:
```bash
sudo systemctl status nginx
```
if all is good, you should see an active status as well as the following: 
'''
Apr 03 16:02:48 linuxArch nginx[1280559]: 2024/04/03 16:02:48 [notice] 1280559#1280559>
Apr 03 16:02:48 linuxArch systemd[1]: Reloaded A high performance web server and a rev>
'''
This means that the service is running and the changes have been applied successfully.

## Upload the HTML file to GitHub
Now we can upload the HTML file to GitHub. 
Note: you should be commiting to git regularly to keep track of changes and to have a backup of your files.

cd to the HTML directory you created earlier:
```bash
cd /web/html/nginx-2420
```

To ensure you're in the right directory, run the following command to list the files in the directory:
```bash
ls
```
if you see the index.html file, you are in the right directory.

Then, initialize a Git Repository in this directory with git init. Init is used to create a new empty repository for more information on this command, you can use “man git" to read the manual page for all git commands.

```bash
git init
```

Use git add to add the index.html to the staging area
Git holds all changes in this staging area until you are ready to commit them later.

```bash
git add index.html
```

To see if the file was added successfully, run the following command to check the status of the file:
```bash
git status
```
you should see the file in green text which means it was added successfully.

Then use git commit to upload and commit the changes to this local repository. :

```bash
git commit -m "Initial commit"
```
Note: the -m flag is used to add a message to the commit. This message should be a brief description of the changes you made in this commit.

Note: if you have not set your Git profile. Use “ git config --global user.email "you@example.com"” and “git config --global user.name "Your Name"” to set up and link your git profile. And then run git commit -m “initial commit”.

## Connect to GitHub:

This next step requires us to connect our local git repo on our droplet to the wider world of the internet and more specifically, GitHub.

Make sure you have an empty repo on GitHub already created and available to use. 

 Once, that is done, we need to back to our Droplet and assign which branch it will be pushed to with “git branch -M main”. Run the following commands to begin the process.

```bash
git branch -M main
```

Once that is done, you now need to the new origin of the GitHub repository. That being its new .git file.

Run the following commands to begin the process:

```bash
git remote add origin https://github.com/yourusername/your-repo.git
```

Replace the url to your new repo’s git file.

Finally, push files to the new repo:

```bash
git push -u origin main
```

Once you do this, GitHub will ask you to login. Enter your Username as Normal but instead of your password, you must use a personal access token. 

### Getting a personal access token
Go to your GitHub developer settings and generate a new classic access token and use that in place of your password.

Use this article for intructions: [https://dev.to/shafia/support-for-password-authentication-was-removed-please-use-a-personal-access-token-instead-4nbk](https://dev.to/shafia/support-for-password-authentication-was-removed-please-use-a-personal-access-token-instead-4nbk)

Once you have done this, use sudo systemctl reload nginx. Then open your browser and enter in your droplet’s IP address. If you did everything correct, you should see your webpage.