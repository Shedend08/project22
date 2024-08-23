## Documentation

## Create An Ubuntu Server

- Locate and click on **EC2** within the AWS management console.

- Click on **Launch Instance**

- **Name** your instance and select the **Ubuntu** AMI.

- Click the **Create new key pair** button to generate a key pair for secure connection to your instance.

- Enter a **Key pair name** and click on **Create key pair**.

- Enable **SSH**, **HTTP**, and **HTTPS** access, then proceed to click **Launch instance**.

- Click on **View all instances**.

- Click on the **created instance**.

- Click on the **Connect** button.

- Copy the command provided under **`SSH client`**.

- Open a terminal in the directory where your **`.pem`** file was downloaded, paste the command and press Enter.

### Create And Assign an Elastic IP

- Return to your AWS console and click on the **menu icon** to open the dashboard menu.

- Select **Elastic IPs** under **Network & Security**.

- Keep the settings unchanged and proceed to click **Allocate**.

- **Associate this Elastic IP address** with your running instance.

- Select the instance you wish to associate with the elastic IP address, then click on **Associate**.

- Paste the **command** into your terminal and then press Enter. When prompted, type **"yes"** and press Enter to connect.

### Install Nginx and Setup Your Website

- Execute the following commands.

`sudo apt update`

`sudo apt upgrade`

`sudo apt install nginx`

- Start your Nginx server by running the **`sudo systemctl start nginx`** command, enable it to start on boot by executing **`sudo systemctl enable nginx`**, and then confirm if it's running with the **`sudo systemctl status nginx`** command.

- Go back to your EC2 dashboard and copy your **Public IPv4 address**.

- Visit your instances **Public IPv4 address** in a web browser to view the default Nginx startup page.

- Download your website template from your preferred website by navigating to the website, locating the template you want, and obtaining the download URL for the website.

- Right click and select **Inspect** from the drop down menu.

- Click on the **Network** tab and then click **Download** button.

- Right click on the website name, select **Copy** and click on **Copy link address**.

To install the unzip tool, run the following command: **`sudo apt install unzip`**.

Execute the command to download and unzip your website files `sudo curl -o /var/www/html/2110_character.zip https://www.tooplate.com/zip-templates/2110_character.zip && sudo unzip -d /var/www/html/ /var/www/html/2110_character.zip && sudo rm -f /var/www/html/2110_character.zip`.

- Download the 2nd website template by running the following command:

```
sudo curl -o /var/www/html/2108_dashboard.zip https://www.tooplate.com/zip-templates/2108_dashboard.zip && sudo unzip -d /var/www/html/ /var/www/html/2108_dashboard.zip && sudo rm -f /var/www/html/2108_dashboard.zip
```
- To set up your website's configuration, start by creating a new file in the Nginx sites-available directory. Use the following command to open a blank file in a text editor: **`sudo nano /etc/nginx/sites-available/character`**

- Copy and paste the following code into the open text editor.

```
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/html/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- Edit the `root` directive within your server block to point to the directory where your downloaded website content is stored.

- Configure your second website by creating a new file in the Nginx sites-available directory with the following command: `sudo nano /etc/nginx/sites-available/dashboard`.

- Copy and paste the following code into the open text editor.

```
server {
    listen 80;
    server_name placeholder.com www.placeholder.com;

    root /var/www/html/placeholder.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- Edit the **`root`** directive within your server block to point to the directory where your downloaded website content is stored.

- Create a symbolic link for both websites by running the following command.
`sudo ln -s /etc/nginx/sites-available/character /etc/nginx/sites-enabled/`
`sudo ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/`


- Run the **`sudo nginx -t`** command to check the syntax of the Nginx configuration file.

- Delete the default files in the sites-available and sites-enabled directories by executing the following commands:

```
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

- Restart the Nginx server by executing the following command: **`sudo systemctl restart nginx`**.

### Create An A Record

I have a domain name where I purchased from NameCheap and then moving hosting to AWS Route 53, where I set up an A record.

- On the website click on **Domain List**.

- Click on the **Manage** button.

- Go back to your AWS console, search for **Route 53①**, and then choose **Route 53②** from the list of services shown.

- Click on **Get started**.

- Select **Create hosted zones①** and click on **Get started②**.

- Enter your **Domain name①**, choose **Public hosted zone②** and then click on **Create hosted zone③**.

- Select the **created hosted zone①** and copy the assigned **Values②**.

- Go back to your domain registrar and select **Custom DNS** within the **NAMESERVERS** section.

- Paste the values you copied from Route 53 into the appropriate fields, then click the **checkmark symbol** to save the changes.

- Head back to your AWS console and click on **Create record**.

- Paste your **IP address①** and then click on **Create records②**.

- Your A record has been successfully created.

- Click on **Create record** again, to create the record for your sub domain.

- Input the **Record name①**, paste your **IP address②** and then click on **Create records③**.

- Repeat the same process while creating your second subdomain record, and confirm that they both exist in the records list.

- Open your terminal and run **`sudo nano /etc/nginx/sites-available/character`** to edit your settings. Enter the name of your domain and then save your settings.

- Run **`sudo nano /etc/nginx/sites-available/dashboard`** to edit your settings. Enter the name of your domain and then save your settings.

- Restart your nginx server by running the **`sudo systemctl restart nginx`** command.

- Go to your domain name in a web browser to verify that your website is accessible.

> [!NOTE]
You may notice the sign that says **Not secure**. Next, you'll use certbot to obtain the SSL certificate necessary to enable HTTPS on your site.

---

#### Install certbot and Request For an SSL/TLS Certificate

- Install certbot by executing the following commands:
`sudo apt update`
`sudo apt install python3-certbot-nginx`
`sudo certbot --nginx`

- Execute the **`sudo certbot --nginx`** command to request your certificate. Follow the instructions provided by certbot and select the domain name for which you would like to activate HTTPS.

> [NOTE]
In this case, SSL certificates were only created for **`character.cloudghoul.online`** and **`dashboard.cloudghoul.online`**. So, when prompted, I entered the numbers 1 and 3 (corresponding to those two domains) to select them for certificate generation. If we had also created records for **`www.character.cloudghoul.online`** and **`www.dashboard.cloudghoul.online`**, I could have simply pressed Enter to accept the default selection (**all available records**). If you try to create a certificate without having created an A record first, you will receive an error message.

- Verify the website's SSL using the OpenSSL utility with the command: **`openssl s_client -connect character.cloudghoul.online:443`**

- Verify the 2nd website's SSL using the OpenSSL utility with the command: **`openssl s_client -connect dashboard.cloudghoul.online:443`**

- Visit **`https://<domain name>`** to view your websites.

![1](img/Screenshot%202024-08-22%20at%209.05.48 PM.png)

![2](img/Screenshot%202024-08-22%20at%209.06.38 PM.png)

---
---

#### The End Of Project 2
