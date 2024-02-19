# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker compose 2

### Preface

This a simple CMS solution for vanilla wow servers using docker, wordpress and a traefik reverse proxy for free SSL certificates.

Result: https://vanillareforged.org/

### Instructions lazycms
### Docker Setup

First, clone the repository and move into it.

```sh
git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker/
cd lazycms-vmangos-docker
```

At this point, you have to adjust the `./.env` for your desired setup.

Then start your environment with:

```sh
docker compose up -d
```

#### Connect to your IP or website address to do the basic wordpress setup:

- Use the sql user and database name from your .env file.
- The database hostname is wordpress_database.

#### Edit your wp-config.php file, so wordpress can be reached behind a reverse proxy:

Open your wp-config.php file located in var/www/html in your lazycms-vmangos-docker directory.

Add this code at the beginning of the file right after "<?php":

```sh
if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
    $_SERVER['HTTPS'] = 'on';
}
```

#### If you want to use traefik to get free SSL certificates through letsencrypt and switch to HTTPS for you Wordpress installation:

Switch the comments in your docker-compose.yml from

```sh
    labels:
     - "traefik.enable=true"
     - "traefik.http.routers.wordpress.entrypoints=web"
#     - "traefik.http.routers.wordpress.entrypoints=websecure"
     - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
#     - "traefik.http.routers.wordpress.tls=true"
#     - "traefik.http.routers.wordpress.tls.certresolver=production"
```

to

```sh
    labels:
     - "traefik.enable=true"
#     - "traefik.http.routers.wordpress.entrypoints=web"
     - "traefik.http.routers.wordpress.entrypoints=websecure"
     - "traefik.http.routers.wordpress.rule=Host(`${WEBSITE_URL}`,`${WEBSITE_URL_WWW}`)"
     - "traefik.http.routers.wordpress.tls=true"
     - "traefik.http.routers.wordpress.tls.certresolver=production"
```

then restart your environment with:

```sh
docker compose down
docker compose up -d
```

#### Install WPCode Plugin & create Page to be used for Registration:

Use official Wordpress documentation if you need help with this.

### PHP CODE REGISTRATION FORM

Taken from https://github.com/vmangos/WallRegistrationPage/ and edited with hostnames as they are expected in the vmangos-docker configuration.

Edit as necessary to fit your purpose.

Shoutout to WallCraft (https://www.wallcraft.org/)!

Use the WPCode plugin to create following code snippet (adjust fields to fit your installation).

```sh
/* Database credentials. Assuming you are running mariadb
server with default setting vmangos-docker (user 'mangos' with defined password) */
define("DB_HOST_VMANGOS", "vmangos_database");
define('DB_USERNAME_VMANGOS', 'enter your DB username here');
define('DB_PASSWORD_VMANGOS', 'enter your DB password here');
define('DB_NAME_VMANGOS', 'enter your realmd DB name here');
 
/* Attempt to connect to MySQL database */
$link = mysqli_connect(DB_HOST_VMANGOS, DB_USERNAME_VMANGOS, DB_PASSWORD_VMANGOS, DB_NAME_VMANGOS);
 
// Check connection
if($link === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
}

// Define variables and initialize with empty values
$username = $password = $email = "";
$username_err = $password_err = $email_err = ""; 
$regname = 'Enter registration handler account name here'; //  (create gmlvl 6 account, enable soap in mangosd.conf - soap uses a dummy gm account to handle registration)
$regpass = 'Enter registration handler account pass here';
$host = "vmangos_mangos";
$soapport = 7878;
$command = "account create {USERNAME} {PASSWORD}";
$result = "";

// Define SOAP client
$client = new SoapClient(NULL,
array(
    "location" => "http://$host:$soapport",
    "uri" => "urn:MaNGOS",
    "style" => SOAP_RPC,
    'login' => $regname,
    'password' => $regpass
));

// Processing form data when form is submitted
if($_SERVER["REQUEST_METHOD"] == "POST"){
 
    // Validate username
    if(empty(trim($_POST["username"]))){
        $username_err = "Required";
	} elseif(!ctype_alnum($_POST["username"])) {
		$username_err = "Must be letters and numbers only";
    } elseif(strlen(trim($_POST["username"])) > 16){
        $username_err = "Too long";
    } elseif(strlen(trim($_POST["username"])) < 4){
        $username_err = "Too short";
    } else{
        // Prepare a select statement
        $sql = "SELECT id FROM account WHERE username = ?";
        
        if($stmt = mysqli_prepare($link, $sql)){
            // Bind variables to the prepared statement as parameters
            mysqli_stmt_bind_param($stmt, "s", $param_username);
            
            // Set parameters
            $param_username = htmlspecialchars(trim($_POST["username"]));
            
            // Attempt to execute the prepared statement
            if(mysqli_stmt_execute($stmt)){
                /* store result */
                mysqli_stmt_store_result($stmt);
                
                if(mysqli_stmt_num_rows($stmt) == 1){
                    $username_err = "Taken";
                } else{
                    $username = htmlspecialchars(trim($_POST["username"]));
                }
            } else{
                echo "Error";
            }

            // Close statement
            mysqli_stmt_close($stmt);
        }
    }
    
    // Validate password
    if(empty(trim($_POST["password"])))
	{
        $password_err = "Required";
    }
	if(strlen(trim($_POST["password"])) > 16)
	{
        $password_err = "Too long";
    }
	elseif(strlen(trim($_POST["password"])) < 6)
	{
        $password_err = "Too short";
    }
	else
	{
        $password = htmlspecialchars(trim($_POST["password"]));
    }
	
	// Validate passver
    if(empty(trim($_POST["passver"])))
	{
        $passver_err = "Required";
    }
	if(strlen(trim($_POST["passver"])) != strlen(trim($_POST["password"])))
	{
        $password_err = "Mismatch";
    }
    
    // Validate email
    if(empty(trim($_POST["email"]))){
        $email_err = "Invalid";     
    } else{
        $email = htmlspecialchars(trim($_POST["email"]));
    }
   
    // Check input errors before inserting in database
    if(empty($username_err) && empty($password_err) && empty($passver_err) && empty($email_err)){
		$command = str_replace('{USERNAME}', strtoupper($_POST["username"]), $command);
		$command = str_replace('{PASSWORD}', strtoupper($_POST["password"]), $command);
        try {
        $result = $client->executeCommand(new SoapParam($command, "command"));
		}
		catch (Exception $e) {
		echo $e;
		}
	}
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sign Up</title>
    <link rel="stylesheet" href="style.css">
    <style>
        body{ font: 14px sans-serif; }
        .wrapper{ width: 350px; padding: 20px; }
    </style>
</head>
<body>
    <div class="wrapper">
        <h2><center>Sign Up</center></h2>
        <p>Please fill this form to create an account.</p>
        <form action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]); ?>" method="post">
			<table width="100%">
				<col style="width:25%">
				<col style="width:50%">
				<col style="width:25%">
			<tr><td>
				<label>Username:</label>
			</td><td>
                <input type="text" name="username" class="form-control <?php echo (!empty($username_err)) ? 'is-invalid' : ''; ?>" value="<?php echo $username; ?>">
            </td><td>
				<span class="invalid-feedback"><?php echo $username_err; ?></span>
            </td></tr>		
			<tr><td>
                <label>Password:</label>
			</td><td>
                <input type="password" name="password" class="form-control <?php echo (!empty($password_err)) ? 'is-invalid' : ''; ?>" value="<?php echo $password; ?>">
            </td><td>
				<span class="invalid-feedback"><?php echo $password_err; ?></span>
            </td></tr>	
			<tr><td>
                <label>Verify:</label>
				
				
			</td><td>
                <input type="password" name="passver" class="form-control <?php echo (!empty($passver_err)) ? 'is-invalid' : ''; ?>" value="<?php echo $password; ?>">
            </td><td>
				<span class="invalid-feedback"><?php echo $password_err; ?></span>
            </td></tr>				
			<tr><td>
                <label>Email:</label>
			</td><td>
                <input type="text" name="email" class="form-control <?php echo (!empty($email_err)) ? 'is-invalid' : ''; ?>" value="<?php echo $email; ?>">
            </td><td>
				<span class="invalid-feedback"><?php echo $email_err; ?></span>
			</td></tr>
			</table>
            <div class="form-group">
				<input type="submit" class="clicker" value="Submit">
			<table>
            <h3>Already have an account?</h3>
	<tr><p><h4><input type="button" class="clicker" onclick="location.href='login.php';" value="Login here" /></p></tr></h4></p></tr>
			<tr><span class="invalid-feedback"><?php echo $result; ?></span></tr>
			</table>
			</div>
        </form>
    </div>    
</body>
</html>
```

#### Used themes and plugins for https://www.vanillareforged.org/:
Theme: Twenty Seventeen

Plugins: Options Twenty Seventeen, Updraftplus, WPCode

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/

Support our efforts on Patreon https://www.patreon.com/vanillareforged
