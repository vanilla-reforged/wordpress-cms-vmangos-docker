### lazycms-vmangos-docker

A Docker setup for a simple VMaNGOS CMS
Using a Traefik reverse proxy for free SSL certificates.

### preview

[Vanilla Reforged](https://vanillareforged.org/)

Theme used: Twenty Seventeen

Plugins used: Options Twenty Seventeen, Updraftplus, WPCode


### dependencies

- Docker
- Docker compose 2

### security

Secure your system by understanding the following information: [ufw-docker](https://github.com/chaifeng/ufw-docker).

The ufw commands you will need to secure your installation:

Management:

```sh
ufw allow from [your client ip]
ufw route allow proto tcp from [your client ip] to any
```

Lazycms public access:

```sh
ufw route allow proto tcp from any to any port 80
ufw route allow proto tcp from any to any port 443
```

### docker setup

First, clone the repository and move into it.

```sh
git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker/
cd lazycms-vmangos-docker
```

At this point, you have to adjust the `./.env` file for your desired setup.

Then start your environment with:

```sh
docker compose up -d
```

### MANDATORY TODO BEFORE ANYTHING ELSE:

**Connect to your IP or website address to do the basic WordPress setup:**

- Use the SQL user and database name from your `.env` file.
- The database hostname is `wordpress_database`.

**Edit your wp-config.php file, so WordPress can be reached behind a reverse proxy:**

Open your `wp-config.php` file located in `var/www/html` in your `lazycms-vmangos-docker` directory.

Add this code at the beginning of the file right after `<?php`:

```sh
if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
    $_SERVER['HTTPS'] = 'on';
}
```

### THEN DO:

If you want to use Traefik to get free SSL certificates through Let's Encrypt and switch to HTTPS for your WordPress installation:

Switch the comments in your `docker-compose.yml` from:

```sh
labels:
 - "traefik.enable=true"
 - "traefik.http.routers.wordpress.entrypoints=web"
# - "traefik.http.routers.wordpress.entrypoints=websecure"
 - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
# - "traefik.http.routers.wordpress.tls=true"
# - "traefik.http.routers.wordpress.tls.certresolver=production"
```

to:

```sh
labels:
 - "traefik.enable=true"
# - "traefik.http.routers.wordpress.entrypoints=web"
 - "traefik.http.routers.wordpress.entrypoints=websecure"
 - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
 - "traefik.http.routers.wordpress.tls=true"
 - "traefik.http.routers.wordpress.tls.certresolver=production"
```

Then uncomment the following sections in your `/etc/traefik/traefik.yaml` file to enable the HTTPS redirection:

```sh
#    http:
#      redirections:
#        entryPoint:
#          to: websecure
#          scheme: https
```

```sh
#certificatesResolvers:
#  staging:
#    acme:
#      email: your-emaill@vmangos.com
#      storage: /etc/traefik/acme.json
#      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
#      httpChallenge:
#        entryPoint: web
```

```sh
#  production:
#     acme:
#       email: your-emaill@vmangos.com
#       storage: /etc/traefik/acme.json
#       caServer: "https://acme-v02.api.letsencrypt.org/directory"
#       httpChallenge:
#         entryPoint: web
```

Then restart your environment with:

```sh
docker compose down
docker compose up -d
```

### install WPCode plugin & create page to be used for registration:

Use official WordPress documentation if you need help with this.

### registration form using the wordpress WPCode plugin and php

Use the WPCode plugin to create the following code snippet and insert it into your Registration Page.
Taken from [WallRegistrationPage](https://github.com/vmangos/WallRegistrationPage/) and adjusted with the help of ChatGPT. Shoutout to [WallCraft](https://www.wallcraft.org/)!

Adjust these entries to fit your installation (without the {}):

- {enter your DB username here}
- {enter your DB password here}
- {Enter registration handler account name here}
- {Enter registration handler account pass here}
- {URL of your registration Page}

```sh
<?php
// Database credentials
define("DB_HOST_VMANGOS", "vmangos_database");
define('DB_USERNAME_VMANGOS', '{enter your DB username here}');
define('DB_PASSWORD_VMANGOS', '{enter your DB password here}');
define('DB_NAME_VMANGOS', 'realmd');

// Attempt to connect to MySQL database
$link = mysqli_connect(DB_HOST_VMANGOS, DB_USERNAME_VMANGOS, DB_PASSWORD_VMANGOS, DB_NAME_VMANGOS);

// Check connection
if($link === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
}

// Define variables and initialize with empty values
$username = $password = $email = "";
$username_err = $password_err = $email_err = $passver_err = ""; 
$regname = '{Enter registration handler account name here}';
$regpass = '{Enter registration handler account pass here}';
$host = "vmangos_mangos";
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

    // Validate email
    $input_email = trim($_POST["email"] ?? "");
    if(empty($input_email)){
        $email_err = "Please enter an email address.";
    } elseif(!filter_var($input_email, FILTER_VALIDATE_EMAIL)){
        $email_err = "Please enter a valid email address.";
    } else {
        $email = $input_email;
    }
    
    // Check input errors before inserting in database and making SOAP call
    if(empty($username_err) && empty($password_err) && empty($passver_err) && empty($email_err)){
        $command = str_replace(['{USERNAME}', '{PASSWORD}'], [strtoupper($username), strtoupper($password)], $command);
        try {
            $result = $client->__soapCall("executeCommand", [new SoapParam($command, "command")]);
            // Handle success or failure of SOAP call here
            // For example, checking if $result indicates success
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
                <input type="text" id="username" name="username" class="form-control" value="<?php echo $username; ?>">
                <span class="invalid-feedback"><?php echo $username_err; ?></span>
            </div>    
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" name="password" class="form-control" value="<?php echo $password; ?>">
                <span class="invalid-feedback"><?php echo $password_err; ?></span>
            </div>
            <div class="form-group">
                <label for="passver">Confirm Password</label>
                <input type="password" id="passver" name="passver" class="form-control" value="<?php echo $pass_ver; ?>">
                <span class="invalid-feedback"><?php echo $passver_err; ?></span>
            </div>
            <div class="form-group">
                <label for="email">Email</label>
                <input type="text" id="email" name="email" class="form-control" value="<?php echo $email; ?>">
                <span class="invalid-feedback"><?php echo $email_err; ?></span>
            </div>
            <div class="form-group">
                <input type="submit" class="custom-submit-button" value="Submit">
            </div>
        </form>
    </div>    
</body>
</html>
```

## vanilla reforged links

- [Vanilla Reforged Website](https://vanillareforged.org/)
- [Vanilla Reforged Discord](https://discord.gg/KkkDV5zmPb)
- [Vanilla Reforged Patreon](https://www.patreon.com/vanillareforged)

## other links
