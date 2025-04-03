# Wordpress CMS VMaNGOS Docker

A Docker setup for a simple VMaNGOS WordPress CMS, utilizing a Traefik reverse proxy for free SSL certificates.

## Todo

- Implement Docker Swarm for managing Docker secrets.

## Preview

Check out [Vanilla Reforged](https://vanillareforged.org/) for a live preview.

- **Theme used**: Eldritch

## Dependencies

- **Docker**
- **Docker Compose 2.x**

Make sure Docker and Docker Compose are installed on your system.

## Security Considerations

To secure your system, refer to the [ufw-docker guide](https://github.com/chaifeng/ufw-docker) for essential firewall configurations.

### Essential UFW Commands

- **Allow management access from a specific IP**:
    ```bash
    ufw allow from [your client ip] to any
    ufw route allow proto tcp from [your client ip] to any
    ```

- **Allow public access to specific ports**:
    ```bash
    ufw route allow proto tcp from any to any port 80
    ufw route allow proto tcp from any to any port 443
    ```

## Docker Setup

### Step 1: Clone the Repository

Use a User with UID:GUID 1000:1000 for this step (default user on ubuntu).:

    git clone https://github.com/vanilla-reforged/wordpress-cms-vmangos-docker/
    cd wordpress-cms-vmangos-docker

### Step 2: Configure Environment Variables

Edit the `.env`, `.env-wordpress`, and `.env-wordpress-database` files according to your setup.

### Step 3: Start Docker Environment

Once configured, bring up the Docker environment with:

    sudo docker compose up -d

## Mandatory Changes Before Proceeding

### WordPress Reverse Proxy Configuration

You must edit your `wp-config.php` file to enable WordPress to work behind a reverse proxy.

1. Open `wp-config.php` located in `var/www/html` inside the `lazycms-vmangos-docker` directory.
2. Add this snippet directly after `<?php`:

    ```php
    if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
        $_SERVER['HTTPS'] = 'on';
    }
    ```

## Switching to HTTPS with Traefik

To use Traefik for obtaining free SSL certificates through Let's Encrypt, switch the comments in your `docker-compose.yml` from:

    labels:
     - "traefik.enable=true"
     - "traefik.http.routers.wordpress.entrypoints=web"
    # - "traefik.http.routers.wordpress.entrypoints=websecure"
     - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
    # - "traefik.http.routers.wordpress.tls=true"
    # - "traefik.http.routers.wordpress.tls.certresolver=production"

to:

    labels:
     - "traefik.enable=true"
    # - "traefik.http.routers.wordpress.entrypoints=web"
     - "traefik.http.routers.wordpress.entrypoints=websecure"
     - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
     - "traefik.http.routers.wordpress.tls=true"
     - "traefik.http.routers.wordpress.tls.certresolver=production"

### Update Traefik Configuration

Uncomment the following sections in `/etc/traefik/traefik.yaml` to enable HTTPS redirection:

    #    http:
    #      redirections:
    #        entryPoint:
    #          to: websecure
    #          scheme: https

    #certificatesResolvers:
    #  staging:
    #    acme:
    #      email: your-email@vmangos.com
    #      storage: /etc/traefik/acme.json
    #      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
    #      httpChallenge:
    #        entryPoint: web

    #  production:
    #    acme:
    #      email: your-email@vmangos.com
    #      storage: /etc/traefik/acme.json
    #      caServer: "https://acme-v02.api.letsencrypt.org/directory"
    #      httpChallenge:
    #        entryPoint: web

### Restart Docker Environment

Apply the changes by restarting your Docker environment:

    sudo docker compose down
    sudo docker compose up -d

## Installing WPCode Plugin & Creating a Registration Page

Refer to the official WordPress documentation for instructions on installing the WPCode plugin and creating a registration page.

---

### PHP Snippet for Registration Form

Edit these to fit your installation:

{enter your DB username here}/
{enter your DB password here}/
{enter registration handler account username here}/
{enter registration handler account password here}/
{URL of your registration Page}


    <?php
    // Database credentials
    define("DB_HOST_VMANGOS", "vmangos-database");
    define('DB_USERNAME_VMANGOS', '{enter your DB username here}');
    define('DB_PASSWORD_VMANGOS', '{enter your DB password here}');
    define('DB_NAME_VMANGOS', 'realmd');

    // Attempt to connect to MySQL database
    $link = mysqli_connect(DB_HOST_VMANGOS, DB_USERNAME_VMANGOS, DB_PASSWORD_VMANGOS, DB_NAME_VMANGOS);

    // Check connection
    if($link === false){
        die("ERROR: Could not connect. Please try again later.");
    }

    // Define variables and initialize with empty values
    $username = $password = "";
    $username_err = $password_err = $passver_err = ""; 
    //$email = ""; // Initialize email variable (commented out)
    $regname = '{enter registration handler account username here}';
    $regpass = '{enter registration handler account password here}';
    $host = "vmangos-mangos";
    $soapport = 7878;
    $command = "account create {USERNAME} {PASSWORD}";
    $result = "";

    // Define SOAP client
    $client = new SoapClient(null, [
        "location" => "http://$host:$soapport",
        "uri" => "urn:MaNGOS",
        "style" => SOAP_RPC,
        'login' => $regname,
        'password' => $regpass
    ]);

    // Processing form data when form is submitted
    if($_SERVER["REQUEST_METHOD"] == "POST"){
    
        // Validate username
        $input_username = trim($_POST["username"] ?? "");
        if(empty($input_username)){
            $username_err = "Please enter a username.";
        } elseif(!ctype_alnum($input_username)) {
            $username_err = "Username must be letters and numbers only.";
        } elseif(strlen($input_username) > 16){
            $username_err = "Username too long.";
        } elseif(strlen($input_username) < 4){
            $username_err = "Username too short.";
        } else {
            $sql = "SELECT id FROM account WHERE username = ?";
            
            if($stmt = mysqli_prepare($link, $sql)){
                mysqli_stmt_bind_param($stmt, "s", $param_username);
                $param_username = $input_username;
                
                if(mysqli_stmt_execute($stmt)){
                    mysqli_stmt_store_result($stmt);
                    
                    if(mysqli_stmt_num_rows($stmt) == 1){
                        $username_err = "This username is already taken.";
                    } else{
                        $username = $input_username;
                    }
                } else{
                    echo "Oops! Something went wrong. Please try again later.";
                }
                mysqli_stmt_close($stmt);
            }
        }
        
        // Validate password
        $input_password = trim($_POST["password"] ?? "");
        if(empty($input_password)){
            $password_err = "Please enter a password.";
        } elseif(strlen($input_password) < 6){
            $password_err = "Password must have at least 6 characters.";
        } elseif(strlen($input_password) > 16){
            $password_err = "Password must not exceed 16 characters.";
        } elseif(!preg_match('/^[a-zA-Z0-9!@#$%^&*_\-+=.,;:~]+$/', $input_password)) {
            $password_err = "Password contains invalid characters. Please use only letters, numbers, and these special characters: !@#$%^&*_-+=.,;:~";
        } else {
            $password = $input_password;
        }

        // Confirm password
        $input_passver = trim($_POST["passver"] ?? "");
        if(empty($input_passver)){
            $passver_err = "Please confirm the password.";
        } elseif($password !== $input_passver){
            $passver_err = "Password did not match.";
        }

        // Validate email (commented out)
        /*
        $input_email = trim($_POST["email"] ?? "");
        if(empty($input_email)){
            $email_err = "Please enter an email address.";
        } elseif(!filter_var($input_email, FILTER_VALIDATE_EMAIL)){
            $email_err = "Please enter a valid email address.";
        } else {
            $email = $input_email;
        }
        */

        // Use the dummy email address
        $email = "dummy@vanillareforged.org";
        
        // Check input errors before inserting in database and making SOAP call
        if(empty($username_err) && empty($password_err) && empty($passver_err)){

        $username = htmlspecialchars(trim($input_username));
        $password = htmlspecialchars(trim($input_password));

            $command = str_replace(['{USERNAME}', '{PASSWORD}'], [strtoupper($username), strtoupper($password)], $command);
            try {
                $result = $client->__soapCall("executeCommand", [new SoapParam($command, "command")]);
                // Handle success or failure of SOAP call here
                $success_message = "Account successfully created!";
            } catch (Exception $e) {
                echo "Registration failed: " . $e->getMessage();
            }
        }
    }
    ?>

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <!-- Ensure proper rendering and touch zooming on mobile devices -->
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <style>
            body { 
                font: 14px sans-serif; 
                text-align: center; 
                margin: 0; /* Removes default margin */
                padding: 0; /* Removes default padding */
                display: flex; /* Use flexbox for body to center wrapper vertically */
                justify-content: center; /* Center horizontally in the flex container */
                align-items: center; /* Center vertically in the flex container */
                min-height: 100vh; /* Minimum height of 100% of the viewport height */
            }
            .wrapper { 
                width: 90%; /* Adjust width to be responsive */
                max-width: 360px; /* Maximum width of the wrapper */
                padding: 20px; 
                margin: auto; 
                box-shadow: 0 4px 8px rgba(0,0,0,0.1); /* Optional: Adds shadow for better visibility */
            }
            .form-group { 
                margin-bottom: 20px; 
                text-align: left; 
            }
            .form-group label { 
                display: block;
                text-align: center;
                margin-bottom: 5px; 
            }
            .form-control {
                width: 100%; /* Full width */
                padding: 10px; /* Adjust padding */
                margin-bottom: 10px; /* Adds bottom margin */
            }
            .invalid-feedback { 
                color: red; 
                display: block; 
                text-align: center; /* Centering the error messages */
            }
            input[type="submit"].custom-submit-button {
                background-color: #4b1912;
                color: #a8522d;
                border: none;
                padding: 10px 20px;
                border-radius: 5px;
                cursor: pointer;
                font-size: 16px;
                width: 100%; /* Full width */
                margin: 20px 0; /* Adjust margin */
            }
            input[type="submit"].custom-submit-button:hover {
                background-color: #3a1410;
            }
        </style>
    </head>
    <body>
        <div class="wrapper">
            <form action="{URL of your registration Page}" method="post">
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" id="username" name="username" class="form-control" value="<?php echo htmlspecialchars($username); ?>">
                    <span class="invalid-feedback"><?php echo $username_err; ?></span>
                </div>    
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" id="password" name="password" class="form-control" value="<?php echo htmlspecialchars($password); ?>">
                    <span class="invalid-feedback"><?php echo $password_err; ?></span>
                </div>
                <div class="form-group">
                    <label for="passver">Confirm Password</label>
                    <input type="password" id="passver" name="passver" class="form-control" value="<?php echo htmlspecialchars(isset($input_passver) ? $input_passver : ''); ?>">
                    <span class="invalid-feedback"><?php echo $passver_err; ?></span>
                </div>
                <!-- Email field commented out -->
                <!--
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="text" id="email" name="email" class="form-control" value="<?php echo htmlspecialchars($email); ?>">
                    <span class="invalid-feedback"><?php echo $email_err; ?></span>
                </div>
                -->
                <div class="form-group">
                    <input type="submit" class="custom-submit-button" value="Submit">
                </div>
            </form>

	        <?php if (!empty($success_message)) : ?>
                <div><?php echo $success_message; ?></div>
            <?php endif; ?>
        </div>    
    </body>
    </html>

    ?>

## Vanilla Reforged Links
- [Vanilla Reforged Website](https://vanillareforged.org/)
- [Vanilla Reforged Discord](https://discord.gg/KkkDV5zmPb)
- [My Patreon](https://www.patreon.com/vanillareforged)
- [Buy Me a Coffee](https://buymeacoffee.com/vanillareforged)
