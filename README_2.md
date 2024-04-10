# Linux Assignment 3 Part 2

### Continue from Assignment 3 Part 1, read README.md for part 1

## Purpose:
Add a fireware using ufw and a reverse proxy server to a backend. 

>A reverse proxy server is used to "distribute the load among several servers, seamlessly show content from different websites, or pass requests for processing to application servers over protocols other than HTTP" (https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

# Adding a Firewall using ufw
1. Ensure ufw is installed. To install ufw, run the following command:
```bash
sudo pacman -S ufw
```
 > -S is used to install a package with pacman
2. Before enabling the ufw serivce, **ensure that ssh is allowed** or you will be locked out of the server.
> Note: by default all incoming connections are denied and all outgoing connections are allowed.
 To allow ssh, run the following command:
```bash
sudo ufw allow ssh
# or
sudo ufw allow 22
# 22 is the port number for ssh
```
3. Enable the ufw service by running the following command:
```bash
sudo systemctl enable --now ufw.service
```
>Note: --now is used to start the service immediately after enabling it.

4. Also allow http connections and port 8080 as we will be using them to connect to the backend server. To allow these connections, run the following command:
```bash
sudo ufw allow http
# or
sudo ufw allow 80
# 80 is the port number for http
sudo ufw allow 8080
```
5. To check the status of the ufw service and all the rules(connections) that are allowed, run the following command:
```bash
sudo ufw status verbose
```

Should look something like this:
```bash
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         LIMIT IN    Anywhere
22                         ALLOW IN    203.0.113.0/24
8080                       ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
22 (v6)                    LIMIT IN    Anywhere (v6)
8080 (v6)                  ALLOW IN    Anywhere (v6)
80 (v6)                    ALLOW IN    Anywhere (v6)
"
```
> Note: for port 22(ssh) I have limited the incoming connection. Which is done by using LIMIT. This is done to limit the number of incoming connections to the server. 
>To limit the incoming connections, run the following command:
>```bash
># example limiting ssh connections
>sudo ufw limit ssh
>```

# Reverse Proxy Server
## Adding the Backend
1. We need to add the provided backend binary file(hello-server) to the server. For this you will need to enter your droplet using sftp. To do this, run the following command:
> **NOTE: cd into the directory where the binary file is located before running the following command.**
```bash
sftp root@<ip_address>
# or sftp droplet_name if you have added the droplet to your ssh config file
```
2. Once you are in the droplet, run the following command to upload the binary file:
```bash
put hello-server
```
>Note: put will upload files from your host to the droplet.

exit sftp and return to your ssh droplet to begin the next steps.

## Creating a Backend Service
> Note: you can name the files and directories whatever you like just remember them. I will be using the names provided in the assignment.
1. Move the hello-server file into the user local bin directory to store the binary file. To do this, run the following command:
```bash
sudo mv hello-server /usr/local/bin/hello-server
```
> Note: because hello-server is a binary file, it should be stored in the bin directory, specifically for the local user.

2. Create a service file for the hello-server in the /etc/systemd/system directory. To do this, run the following command:
```bash
sudo vim /etc/systemd/system/hello-server.service
```
> **Remember to add the .service extension to the file name.**

3. Add the following to the hello-server.service file:
```bash
[Unit]
Description=Run Backend hello-server

[Service]
Type=simple
ExecStart=/usr/local/bin/hello-server
restart=always

[Install]
WantedBy=multi-user.target
```
> Explanation:
> - Description: a basic description of the service
> - Type: the type of service, in this case, it is a simple service, which is the default type so it is not necessary to add this line.
> - ExecStart: the command to run the service which is located in the /usr/local/bin directory that we defined in step 1.
> - restart: this indicates the circumstances under which systemd will attempt to automatically restart the service. In this case, it is always.
> - WantedBy: the target that the service should be started by. In this case, it is the multi-user target which means the service can accept connections from multiple users.

4. Reload the systemd daemon to apply the changes made to the service file. To do this, run the following command:
```bash
sudo systemctl daemon-reload
```
> You need to reload the daemon whenever you make changes to the service file to let systemd know that changes have been made.

5. Start the hello-server service by running the following command:
```bash
sudo systemctl start hello-server
```
Also enable the service to start on boot by running the following command:
```bash
sudo systemctl enable hello-server
```
> Systemctl is a systemd utility that is used to manage services. 

6. Check the status of the service by running the following command:
```bash
sudo systemctl status hello-server
```
> If all is good, you should see a green active status.

## Creating a Reverse Proxy Server
1. Return to the nginx configuration file created in part 1. Run the following command to edit the file:
```bash
#replace nginx-2420.conf with the name of the file you created in part 1
sudo vim /etc/nginx/sites-available/nginx-2420.conf
```

2. Add the following to the configuration file:
```
server {
    #you should have the listen 80 and server_name localhost from part 1
        listen 80;
        server_name localhost;
        location / {
                root /web/html/nginx-2420;
                index index.html;
        }

        location /hey {
            proxy_pass http://127.0.0.1:8080/hey;
        }

        location /echo {
            proxy_pass http://127.0.0.1:8080/echo;
        }
}
```
> Explanation:
> We have added two locations/routes to the server block. The first location is /hey and the second location is /echo. The inccluded backend routes are running on port 8080. 
>proxy_pass is used to pass the request
> When nginx proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client

3. Test the configuration file for syntax errors by running the following command:
```bash
sudo nginx -t
```
> -t is used to test the configuration file for syntax errors.

4. Once everything looks good, restart the nginx service by running the following command:
```bash
sudo systemctl restart nginx
```
and check the status of the service by running the following command:
```bash
sudo systemctl status nginx
```

## Testing the Reverse Proxy Server
1. To test the reverse proxy server, run the following curl command in your local terminal, I will be using powershell 7:
```
curl http://<ip_address>
```
> This should return the index.html file that was created in part 1 if everything is working correctly.

2. To test the /hey route, run the following curl command:
```
curl http://<ip_address>/hey
```
> This should return the response from the /hey route in the backend server.

3. To test the /echo route, run the following curl command:
```
 curl -X POST -H "Content-Type: application/json" -d '{"message": "Hello from your server"}' http://<ip_address>/echo
```
> This should return the response from the /echo route in the backend server which should be the message you sent.