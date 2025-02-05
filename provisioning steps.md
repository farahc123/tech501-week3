# Steps to deploy 2-tier architecture in AWS

1. Create DB app instance (i.e. VM):
   - name
   - Ubuntu 22.04 LTS
   - t3.micro
   - create keypair: AWS will give the private key contents; download and securely store; use a bash command to work out the public key and store this too
   - Use default network (VPC) and subnet
   - Create security group, allowing SSH from anywhere for testing purpose
 2. Provision app VM:
    1. update and upgrade packages
    2. install gnupg curl, which we will use to download  gpg key
    3. create file list
    4. updates
    5. installs mongodb
    6. manually configure bindip in /etc/mongod.conf from localhost to 0.0.0.0 so it accepts connections from anywhere
    7. enable mongod
    8. check it's enabled
    9. start mongod
    10. restart mongod
![alt text](image-14.png)
![alt text](image-15.png)  


3. Create app VM:
   - Same as above, except add an inbound HTTP rule to the NSG as well as a custom TCP rule on port 3000

4. Provision app VM:

    1. Update package list and install them:
    `sudo apt update && sudo apt upgrade -y`
 
    2. Install nginx:
`sudo apt install nginx -y`
    3. Enabling and starting nginx
`sudo apt enable nginx`
`sudo systemctl start nginx`
 
    4. Installing *npm* and *nodejs*
`sudo DEBIAN_FRONTEND=noninteractive bash -c "curl -fsSL https://deb.nodesource.com/setup_20.x | bash -"`
`sudo DEBIAN_FRONTEND=noninteractive apt-get install -y nodejs` 
 
    5. Installing pm2
`sudo npm install -g pm2`
 
    6. Cloning the nodejs app repo (using Sameem's for now while I figure out issues with mine)
`git clone https://github.com/sameem97/tech501-sparta-app.git`
 
    7. Creating a backup of nginx configuration file:
`cd /etc/nginx/sites-available/`
`cp default default.backup` 

    8. Editing nginx configuration file to apply reverse proxy
`sudo nano /etc/nginx/sites-available/default`

- Removing *try_files* line  and replacing with:

`proxy_pass http://localhost:3000;}`

    9. Checking if *nginx* config file syntax is okay:

`sudo nginx -t`

    10. Reloading *nginx* to put above config file edit into action

`sudo systemctl reload nginx`
 
Steps to add:

# Connect to the mongodb server
export DB_HOST=mongodb://<db_private_ip>:27017/posts
 
# Change directory to the app
cd tech501-sparta-app/app
 
# Install npm packages, check db connection, clearing and reseeding (populating) the database
npm install
 
# Start the node app in backround with pm2
npm start app
 
# Check if all commands were successful
if [ $? -eq 0 ]; then
  echo "Deployment successful!"
else
  echo "Deployment failed!"
  exit 1

 
app not working now I sed-ed nginx config
