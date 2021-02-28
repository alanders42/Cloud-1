# Cloud-1

Introduction to Cloud infrastructure using the Google Cloud Platform.

> The site will be hosted at http://34.96.109.216//. The site will only be up during evalutaion.

## Requirements:
- 2 instances have to be running at all times.
- Traffic pikes will trigger the launch of other servers with perfectly synced data
- Instance group has to be scaled depending on traffic.
- Logged-in users will stay identified for the length of a normal session
- CDN is used to serve static content to all the users.
- Any failure should be handled so that your website is always available.
- Hosting cost should reflect your actual usage

## Setup with Google Cloud Platform:
1.  Go to marketplace and deploy a WordPress VM:
    1. Navigate to storage and create a bucket.
    2. Navigate to IAM & Admin > Service Accounts.
    3. Create a service account and download the keyfile.
    4. SSH into the Wordpress instance.
    5. Once you're in type Sudo su for super user editing priviledges.
    6. cd to /var/www/html and add the following to your `wp-config.php`.
       ```
       define( 'AS3CF_SETTINGS', serialize( array('provider' => 
        'gcp', 'key-file-path' => '/etc/keyFile.json',) ) );
       ```
    7. Upload the keyfile you downloaded when creating the service account rename it to keyFile.json and move it to /etc.
    8. In Deployment manager go to wordpress admin page and login using the password that is provided.
        1. Install and activate WP Offload Media Lite plugin.
        2. In the plugin settings enter the name of the bucket you created.
    9. Navigate to external-ip/phpmyadmin and login using the details from deployment manager.
    10. Create a new user and only set the username, pass and check all priviledges.
    11. Navigate to SQL and create a new instance(Enable private IP).
    12. Under the Users tab add a new user with the details from the new user you just created on phpmyadmin.
    13. Under the database tab create a new database called wordpress, set charset to 'utf8mb4' and collate to 'utfmb48_general_ci'.
    14. SSH into the Wordpress VM and add the following to `wp-config.php`:
        ```
        define( 'DB_NAME', '<wordpress>');
        define( 'DB_USER', '<Username you just created in phpmyadmin>');
        define( 'DB_PASSWORD', '<Password of that user>');
        define( 'DB_HOST', '<Private IP of your SQL instance>');
        define( 'DB_CHARSET', 'utf8mb4');
        define( 'DB_COLLATE', 'utfmb48_general_ci');
        ```
    4. Install and activate W3 Total Cache plugin:
        1. Enable CDN, select Generic Mirror. Save and purge cache.
        2. At the top of the page it might ask you to add code to the `.htaccess` file.
        2. If so copy the code that was given.
        3. SSH into the wordpress vm, navigate to /var/www/html and create the file `.htaccess` and paste the code.
        
2. Creating an image of your WordPress VM:
    1. Stop the WordPress VM you created in the previous step.
    2. Navigate to Images in the compute engine menu. Select create image.
    3. Change the source disk to your wordpress vm.
   
3. Creating an instance template from your image:
    1. Nagivate to instance template in the compute engine menu. Select create instance template.
    2. Select custom image as boot disk, selecting the image you created.
    3. Allow HTTP and HTTPS traffic.
    
4. Creating an instance group from your instance template:
    1. Navigate to Instance groups in the compute engine menu. Select create instance group.
    2. Set instance template as the template you created.
    3. Make sure Autoscaling is turned on to allow automatic resizing of the instance group.
    4. You need to have a minimum of 2 instances running and a max of 6.
    5. Create new health check on tcp port 80.
    
5. Creating a firewall rule for external access:
    1. Type Firewall in the search bar and select create firewall rule.
    2. Set Direction of traffic to 'Ingress'.
    3. Set Action on Match to allow.
    4. Set targets to all instances in network.
    5. Set Source Filter to IP Ranges.
    6. Add `130.211.0.0/22` and `35.191.0.0/16` under Source IP ranges.
    7. Select TCP and set the port to 80.
    
6. Creating a Static External-IP
    1. Navigate to VPC networks > External IP addresses.
    2. Select Reserve static address.
    3. Call it loadbalancer-static.
    4. Select Global as the type.
    
7) ##### **Create an HTTP(S) Load balancer:**
    * Navigate to Load balancing (from search bar - it's easiest this way).
    * Select `Create a load balancer` and start configuration of `HTTP(S) Load Balancing`.
    * Select `From Internet to my VMs` when prompted.
    * Under Backend configuration, `create a backend service`:
        * Under New backend, select the instance group previously created.
        * Enable `Cloud CDN` and select `Cache static content`.
        * Under Health check, select the health check previously created.
        * Then click create.
    * Under Frontend configuration, `create a frontend service`:
        * Under IP address, select the static ip previously reserved.
    * Then click create.
