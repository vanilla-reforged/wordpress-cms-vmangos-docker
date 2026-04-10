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

### Using Tailscale

[Tailscale](https://tailscale.com/)

Use Tailscale and keep in mind that any port you bind to a container may bypass UFW and become exposed to the public internet.

This is fine as long as you only expose the ports that actually need internet access. For example, you can keep the Mangos database private and access it over Tailscale instead.

To secure your system, refer to the [ufw-docker guide](https://github.com/chaifeng/ufw-docker) for essential firewall configurations.

### Using UFW

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

Use a User with UID:GID 1000:1000 for this step (default user on Ubuntu):

    git clone https://github.com/vanilla-reforged/wordpress-cms-vmangos-docker/
    cd wordpress-cms-vmangos-docker

### Step 2: Configure Environment Variables

Edit the `.env`, `.env-wordpress`, and `.env-wordpress-database` files according to your setup.

Example `.env`:

    WEBSITE_URL=example.com
    WEBSITE_URL_WWW=www.example.com

### Step 3: Create Docker Network

This setup uses an external Docker network named `cms-network`.

    docker network create cms-network

### Step 4: Start Docker Environment

Once configured, bring up the Docker environment with:

    sudo docker compose up -d


## Mandatory Changes Before Proceeding

### WordPress Reverse Proxy Configuration

You must edit your `wp-config.php` file to enable WordPress to work behind a reverse proxy.

1. Open `wp-config.php` located in `var/www/html` inside the project directory.
2. Add this snippet directly after `<?php`:

    ```php
    if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '', 'https') !== false) {
    $_SERVER['HTTPS'] = 'on';
    }
    ```

## HTTPS with Caddy

Caddy automatically provisions and renews TLS certificates (Let's Encrypt).

### Requirements

- `WEBSITE_URL` and `WEBSITE_URL_WWW` must point to your server
- Ports **80** and **443** must be reachable from the internet

### How it works

- Caddy reads your domain from `.env`
- Automatically enables HTTPS
- Proxies traffic to the `wordpress` container

No additional configuration is required.

## Restart Docker Environment

If you change configuration, apply it with:

    sudo docker compose down
    sudo docker compose up -d

## Installing WPCode Plugin & Creating a Registration Page

Refer to the official WordPress documentation for instructions on installing the WPCode plugin and creating a registration page.

---

### PHP Snippet for Registration Form

Edit these to fit your installation:

{YOUR_API_KEY}

```php
<?php
$username = $password = $passver = "";
$username_err = $password_err = $passver_err = "";
$success_message = "";

// API settings
$api_url = 'http://vmangos-api:8080'; // vmangos-api container in vmangos-network
$api_key = '{YOUR_API_KEY}';

// Process form submission
if ($_SERVER["REQUEST_METHOD"] === "POST") {

    // Local input validation (format/length)
    $input_username = trim($_POST["username"] ?? "");
    if (empty($input_username)) {
        $username_err = "Please enter a username.";
    } elseif (!ctype_alnum($input_username)) {
        $username_err = "Username must be letters and numbers only.";
    } elseif (strlen($input_username) < 4 || strlen($input_username) > 16) {
        $username_err = "Username must be 4-16 characters long.";
    } else {
        $username = $input_username;
    }

    $input_password = trim($_POST["password"] ?? "");
    if (empty($input_password)) {
        $password_err = "Please enter a password.";
    } elseif (strlen($input_password) < 6 || strlen($input_password) > 16) {
        $password_err = "Password must be 6-16 characters long.";
    } elseif (!preg_match('/^[a-zA-Z0-9!@#$%^&*()_\-+={}[\]?.,;:~]+$/', $input_password)) {
        $password_err = "Password contains invalid characters.";
    } else {
        $password = $input_password;
    }

    $input_passver = trim($_POST["passver"] ?? "");
    if (empty($input_passver)) {
        $passver_err = "Please confirm the password.";
    } elseif ($password !== $input_passver) {
        $passver_err = "Passwords do not match.";
    }

    // Only call API if local validation passed
    if (empty($username_err) && empty($password_err) && empty($passver_err)) {

        $payload = ['username' => $username, 'password' => $password];

        $ch = curl_init($api_url);
        curl_setopt_array($ch, [
            CURLOPT_POST           => true,
            CURLOPT_POSTFIELDS     => json_encode($payload),
            CURLOPT_HTTPHEADER     => [
                'Content-Type: application/json',
                'X-API-KEY: ' . $api_key
            ],
            CURLOPT_RETURNTRANSFER => true
        ]);

        $response = curl_exec($ch);
        $curl_error = curl_error($ch);
        curl_close($ch);

        if ($curl_error) {
            echo "Registration failed: " . htmlspecialchars($curl_error);
        } else {
            $result = json_decode($response, true);

            if ($result['success'] ?? false) {
                $success_message = $result['message'] ?? "Account created successfully!";
                // Reset fields after successful creation
                $username = $password = $passver = "";
            } else {
                // Map API errors to form fields
                $message = $result['message'] ?? "Registration failed";
                if (stripos($message, 'username') !== false) {
                    $username_err = $message;
                } else {
                    echo "Registration failed: " . htmlspecialchars($message);
                }
            }
        }
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body { font:14px sans-serif; text-align:center; margin:0; padding:0; display:flex; justify-content:center; align-items:center; min-height:100vh; }
.wrapper { width:90%; max-width:360px; padding:20px; margin:auto; box-shadow:0 4px 8px rgba(0,0,0,0.1); }
.form-group { margin-bottom:20px; text-align:left; }
.form-group label { display:block; text-align:center; margin-bottom:5px; }
.form-control { width:100%; padding:10px; margin-bottom:10px; }
.invalid-feedback { color:red; display:block; text-align:center; }
input[type="submit"].custom-submit-button { background-color:#4b1912; color:#a8522d; border:none; padding:10px 20px; border-radius:5px; cursor:pointer; font-size:16px; width:100%; margin:20px 0; }
input[type="submit"].custom-submit-button:hover { background-color:#3a1410; }
</style>
</head>
<body>
<div class="wrapper">
    <form action="" method="post">
        <div class="form-group">
            <label for="username">Username</label>
            <input type="text" id="username" name="username" class="form-control" value="<?php echo htmlspecialchars($username); ?>">
            <span class="invalid-feedback"><?php echo $username_err; ?></span>
        </div>
        <div class="form-group">
            <label for="password">Password</label>
            <input type="password" id="password" name="password" class="form-control">
            <span class="invalid-feedback"><?php echo $password_err; ?></span>
        </div>
        <div class="form-group">
            <label for="passver">Confirm Password</label>
            <input type="password" id="passver" name="passver" class="form-control">
            <span class="invalid-feedback"><?php echo $passver_err; ?></span>
        </div>
        <div class="form-group">
            <input type="submit" class="custom-submit-button" value="Submit">
        </div>
    </form>

    <?php if (!empty($success_message)) : ?>
        <div><?php echo htmlspecialchars($success_message); ?></div>
    <?php endif; ?>
</div>
</body>
</html>
```

## Vanilla Reforged Links
- [Vanilla Reforged Website](https://vanillareforged.org/)
- [Vanilla Reforged Discord](https://discord.gg/KkkDV5zmPb)
- [My Patreon](https://www.patreon.com/vanillareforged)
- [Buy Me a Coffee](https://buymeacoffee.com/vanillareforged)
