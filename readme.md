# Inspiration for this project
As a React Native developer, connecting a real Android or iOS device to my local server is a daily need. Sure, I could mess around with my machine’s public IP, but that’s clunky and not exactly fun. I wanted something clean and human-readable.

That’s where ngrok came in, offering quick tunnels and a slick public URL. For $10/month, I even got a fixed domain. It worked... until it didn’t. Random errors, unstable connections, and no real answers from support left me frustrated.

So I built fuck-ngrok—a simple set of nginx configs you can toss into a cloud container to reverse tunnel back into your local machine. Best part? I get unlimited domains for the dirt-cheap price of a $5/month DigitalOcean droplet (plus whatever domain I use, which gives me endless subdomains too).

Goodbye ngrok. Hello freedom.

# Setting up new Droplet for reverse SSH tunnel

### Setting up droplet
[Follow this guide from digital ocean](https://docs.digitalocean.com/products/droplets/getting-started/recommended-droplet-setup/)

In the linked guide above, the only things that need to be changed is regarding disabling root login and changing the specified username (the guide also tells you to do this)
In the user setup script, change the line
```
sed --in-place 's/^PermitRootLogin.*/PermitRootLogin prohibit-password/g' /etc/ssh/sshd_config
```
To 
```
sed --in-place 's/^PermitRootLogin.*/PermitRootLogin no/g' /etc/ssh/sshd_config
```

### Install and set up NGINX
Run the below commands to update apt packages and install NGINX

```
sudo apt update
sudo apt install nginx
```

Then in the location `/etc/nginx/sites-enabled` create 2 new files called `server.conf` and `supabase.conf`.

Open `server.conf` using nano with the command
```
sudo nano server.conf
```
Then paste the contents of `server.nginx.conf` from this repo replacing the `SERVER_ZOPLO_DOMAIN` with the domain provided for the server.

Repeat the above step for `supabase.conf` and paste in the contents of `supabase.nginx.conf` replacing `SUPABASE_ZOPLO_DOMAIN` with the domain provided for the supabase webhooks.

### Certbot installation
Run the following command to install certbot
```
sudo apt install certbot python3-certbot-nginx
```

### SSL Certificate
Run the following command replacing DOMAIN with the server domain provided.
```
sudo certbot --nginx -d DOMAIN
```
Repeat the same command for the supabase webhook domain

Ensure the command output mentions the correct file that it has edited. This should be `server.conf` or `supabase.conf`

Run the following command to restart NGINX and apply our changes

```
sudo service nginx restart
```

### Connecting via SSH
Ensure the SSH key you have added is linked to your .ssh/config file with the corresponding ip/domain.

Then run the command replacing `LOCAL_SERVER_PORT` with the local server port (usually 8080), `LOCAL_WEBHOOK_PORT` with the supabase emulator webhook port (usually 54321), USERNAME with the username created during the droplet creation and DOMAIN_OR_IP with the domain or ip of the droplet.

```
ssh -R 3333:localhost:LOCAL_SERVER_PORT -R 3334:localhost:LOCAL_WEBHOOK_PORT USERNAME@DOMAIN_OR_IP
```

### Conclusion
You should now have a linux container running that will act as a reverse tunnel to direct traffic from the relevant web ports to your localhost meaning it can be connected to using physical devices on production or dev.


