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

### Secure Container Access with Tailscale

When using Tailscale, be aware that any port you bind to a container will bypass UFW without the modifcation below and become exposed to the public internet.

Only expose ports that require internet access.

To enable SSH access via Tailscale:

    sudo tailscale up --ssh

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
    define( 'FORCE_SSL_ADMIN', true );

    if ( strpos( $_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '', 'https' ) !== false ) {
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
$username = "";
$username_err = $password_err = $passver_err = $form_err = "";
$success_message = "";

$api_url = getenv('VMANGOS_API_URL') ?: 'http://vmangos-api:8090';
$api_key = getenv('VMANGOS_API_KEY') ?: '';

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $input_username = trim($_POST["username"] ?? "");
    $input_password = trim($_POST["password"] ?? "");
    $input_passver  = trim($_POST["passver"] ?? "");

    if ($input_username === "") {
        $username_err = "Please enter a username.";
    } elseif (!ctype_alnum($input_username)) {
        $username_err = "Username must be letters and numbers only.";
    } elseif (strlen($input_username) < 4 || strlen($input_username) > 16) {
        $username_err = "Username must be 4-16 characters long.";
    } else {
        $username = $input_username;
    }

    if ($input_password === "") {
        $password_err = "Please enter a password.";
    } elseif (strlen($input_password) < 6 || strlen($input_password) > 16) {
        $password_err = "Password must be 6-16 characters long.";
    } elseif (!preg_match('/^[a-zA-Z0-9!@#$%^&*()_\-+={}[\]?.,;:~]+$/', $input_password)) {
        $password_err = "Password contains invalid characters.";
    }

    if ($input_passver === "") {
        $passver_err = "Please confirm the password.";
    } elseif ($input_password !== $input_passver) {
        $passver_err = "Passwords do not match.";
    }

    if ($api_key === "") {
        $form_err = "Registration service misconfigured.";
    }

    if ($username_err === "" && $password_err === "" && $passver_err === "" && $form_err === "") {
        $payload = json_encode([
            'username' => $input_username,
            'password' => $input_password
        ]);

        $ch = curl_init($api_url);
        curl_setopt_array($ch, [
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => $payload,
            CURLOPT_HTTPHEADER => [
                'Content-Type: application/json',
                'X-API-KEY: ' . $api_key
            ],
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_CONNECTTIMEOUT => 3,
            CURLOPT_TIMEOUT => 10,
        ]);

        $response = curl_exec($ch);
        $curl_error = curl_error($ch);
        $http_code = curl_getinfo($ch, CURLINFO_RESPONSE_CODE);
        curl_close($ch);

        if ($curl_error) {
            error_log("Registration API curl error: " . $curl_error);
            $form_err = "Registration service unavailable.";
        } else {
            $result = json_decode($response, true);

            if ($http_code === 200 && is_array($result) && ($result['success'] ?? false)) {
                $success_message = $result['message'] ?? "Account created successfully!";
                $username = "";
            } else {
                $message = is_array($result) ? ($result['message'] ?? '') : '';
                if (stripos($message, 'username') !== false) {
                    $username_err = "This username is unavailable.";
                } else {
                    error_log("Registration API failure: HTTP $http_code, body: " . $response);
                    $form_err = "Registration failed. Please try again.";
                }
            }
        }
    }
}
?>
```

## Vanilla Reforged Links
- [Vanilla Reforged Website](https://vanillareforged.org/)
- [Vanilla Reforged Discord](https://discord.gg/KkkDV5zmPb)
- [My Patreon](https://www.patreon.com/vanillareforged)
- [Buy Me a Coffee](https://buymeacoffee.com/vanillareforged)
