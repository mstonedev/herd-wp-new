# herd-wp-new-script

A macOS shell script for creating complete local WordPress sites in Laravel Herd with WP-CLI, DBngin MySQL or MariaDB, HTTPS, local development debugging, Query Monitor, and the Local Mail must-use plugin.

## What it does

Running one command creates a new site in `~/Herd/<site-name>` and performs the normal local setup automatically:

- Creates the WordPress project directory.
- Downloads WordPress core with WP-CLI.
- Creates `wp-config.php` using a TCP database host and the selected database port.
- Creates the database with `wp db create`.
- Installs WordPress with an administrator account.
- Sets the permalink structure to `/%postname%/`.
- Enables local development debugging:
  - `WP_DEBUG` = `true`
  - `WP_DEBUG_LOG` = `true`
  - `WP_DEBUG_DISPLAY` = `false`
- Installs and activates Query Monitor.
- Downloads Local Mail directly into the WordPress must-use plugins directory.
- Secures the Herd site and changes WordPress URLs to HTTPS.

## Requirements

This script is intended for a macOS local-development setup that includes:

- Laravel Herd
- WP-CLI
- DBngin or another local MySQL/MariaDB service
- MySQL command-line client
- `curl`
- Bash

The script assumes Herd serves projects from:

```text
~/Herd
```

It also assumes the database root account is `root` with an empty password, which matches a common local Herd/DBngin configuration. Change the variables in the script if your database setup differs.

## Installation

Store the executable script in your personal scripts directory:

```bash
mkdir -p ~/bin
nano ~/bin/herd-wp-new
```

Paste the script contents, save the file, then make it executable:

```bash
chmod +x ~/bin/herd-wp-new
```

Add `~/bin` to the `PATH` in `~/.zshrc` if it is not already present:

```zsh
# --- PERSONAL SCRIPTS ---
export PATH="$HOME/bin:$PATH"
```

Reload your shell configuration:

```bash
source ~/.zshrc
```

Verify that the command is available:

```bash
which herd-wp-new
```

Expected result:

```text
/Users/your-username/bin/herd-wp-new
```

## Usage

```bash
herd-wp-new <site-name> [database-port]
```

Examples:

```bash
# Uses port 3306 if no port is supplied.
herd-wp-new client-demo

# Create a site using MySQL on port 3306.
herd-wp-new client-demo 3306

# Create a site using MariaDB on port 3307.
herd-wp-new bricks-content-toolkit 3307
```

The database host is configured as `127.0.0.1:<port>`. Using `127.0.0.1` makes the database connection TCP-based and ensures the selected MySQL/MariaDB port is used.

## Generated site structure

For this command:

```bash
herd-wp-new bricks-content-toolkit 3307
```

the script creates a site similar to:

```text
~/Herd/bricks-content-toolkit/
├── wp-admin/
├── wp-content/
│   ├── debug.log                         # Created after WordPress logs a debug event
│   ├── mu-plugins/
│   │   └── local-mail.php
│   └── plugins/
│       └── query-monitor/
├── wp-includes/
└── wp-config.php
```

## Local Mail MU plugin

The script downloads Local Mail from:

```text
https://github.com/mstonedev/local-mail-wp/releases/download/v1.0/local-mail.php
```

It installs the file directly in the must-use plugins directory:

```text
wp-content/mu-plugins/local-mail.php
```

WordPress automatically loads PHP files that are directly inside `wp-content/mu-plugins/`. Local Mail therefore loads automatically for every request and does not require activation in the normal WordPress Plugins screen.

There is no `local-mail` subfolder and no `local-mail-loader.php` in this version of the installer.

## Default admin credentials

Unless you override them, the installer creates this WordPress administrator:

```text
Username: admin
Password: password
```

For a site called `bricks-content-toolkit`, the WordPress dashboard is available at:

```text
https://bricks-content-toolkit.test/wp-admin
```

Change the password after installation:

```bash
cd ~/Herd/bricks-content-toolkit
wp user update admin --user_pass='Choose-A-Strong-Local-Password'
```

## Override credentials

Set environment variables in front of the command to use different credentials for one installation:

```bash
WP_ADMIN_USER=mikestone \
WP_ADMIN_PASSWORD='Choose-A-Strong-Local-Password' \
WP_ADMIN_EMAIL='mikestone@example.test' \
herd-wp-new client-demo 3307
```

These values apply only to that one install and do not permanently modify the script.

## Debugging

The script writes these values to `wp-config.php`:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

This records WordPress and PHP warnings, notices, and errors in:

```text
wp-content/debug.log
```

It prevents debugging output from being printed directly in browser pages.

Watch the log while developing:

```bash
cd ~/Herd/bricks-content-toolkit
tail -f wp-content/debug.log
```

## Verification

From a generated site folder, verify WordPress and installed plugins:

```bash
cd ~/Herd/bricks-content-toolkit

wp core is-installed
wp option get siteurl
wp plugin list --status=active
wp plugin list --status=must-use
```

Verify the Local Mail file exists:

```bash
ls -la wp-content/mu-plugins/
```

Expected output includes:

```text
local-mail.php
```

## Troubleshooting

### Database server or port unavailable

The script checks the selected database server before installing WordPress. Confirm the required DBngin service is running and the port is correct.

For example, test MariaDB on port 3307:

```bash
mysql -h 127.0.0.1 -P 3307 -u root -e "SELECT VERSION();"
```

### `Undefined constant "truee"`

This means there is a typo in `wp-config.php`. PHP Boolean values must be `true` or `false`, without quotation marks and without extra letters.

Correct development settings:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

### `herd-wp-new: command not found`

Check that the script exists and is executable:

```bash
ls -l ~/bin/herd-wp-new
```

Reload zsh:

```bash
source ~/.zshrc
```

Confirm your personal scripts directory is in `PATH`:

```bash
echo "$PATH" | tr ':' '\n' | grep "$HOME/bin"
```

### HTTPS does not work

From inside the project directory, run:

```bash
herd secure
```

Then confirm the WordPress URLs:

```bash
wp option get home
wp option get siteurl
```

For a new site called `client-demo`, both should be:

```text
https://client-demo.test
```

## Notes

- This is a local-development tool. Never use `admin` / `password` on a public or production website.
- Keep database credentials and `.env` files out of Git repositories.
- Query Monitor is a conventional plugin and appears in the normal WordPress Plugins screen.
- Local Mail is a must-use plugin, loads automatically, and does not need normal plugin activation.
- For an existing WordPress site changed from HTTP to HTTPS, use `wp search-replace` carefully. Fresh sites do not typically need it because the script changes `home` and `siteurl` to the final HTTPS URL immediately.
