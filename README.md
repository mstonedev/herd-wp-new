# herd-wp-new-script

A macOS shell script for creating complete local WordPress sites in Laravel Herd with WP-CLI, DBngin MySQL or MariaDB, HTTPS, local development debugging, Query Monitor, and a Local Mail must-use plugin.

## What it does

Running one command creates a new site in `~/Herd/<site-name>` and performs the usual local setup automatically:

- Creates the WordPress project directory.
- Downloads WordPress core using WP-CLI.
- Creates `wp-config.php` using a TCP database host and the selected database port.
- Creates the database through `wp db create`.
- Installs WordPress with an administrator account.
- Sets the permalink format to `/%postname%/`.
- Enables local development debugging:
  - `WP_DEBUG` = `true`
  - `WP_DEBUG_LOG` = `true`
  - `WP_DEBUG_DISPLAY` = `false`
- Installs and activates Query Monitor.
- Downloads Local Mail as a must-use plugin.
- Creates an MU-plugin loader so Local Mail can stay organized in its own folder.
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

It also assumes the database root account is `root` with an empty password, which matches the common local Herd/DBngin configuration. Change the values in the script if your database setup is different.

## Installation

Store the executable script in your personal scripts directory:

```bash
mkdir -p ~/bin
nano ~/bin/herd-wp-new
```

Paste the script contents, save the file, and make it executable:

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
# Uses port 3306 when no port is supplied.
herd-wp-new client-demo

# Create a site using MySQL on port 3306.
herd-wp-new client-demo 3306

# Create a site using MariaDB on port 3307.
herd-wp-new bricks-content-toolkit 3307
```

The database host is configured as `127.0.0.1:<port>`. Using `127.0.0.1` forces a TCP connection and makes the selected MySQL/MariaDB port explicit.

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
│   ├── debug.log                         # Created when WordPress writes a debug event
│   ├── mu-plugins/
│   │   ├── local-mail-loader.php
│   │   └── local-mail/
│   │       └── local-mail.php
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

The actual plugin is stored here:

```text
wp-content/mu-plugins/local-mail/local-mail.php
```

WordPress only automatically discovers PHP files directly in `wp-content/mu-plugins/`; it does not recursively load plugin files in nested folders. The script therefore creates this loader file:

```text
wp-content/mu-plugins/local-mail-loader.php
```

The loader contains:

```php
<?php
/**
 * Plugin Name: Local Mail Loader
 * Description: Loads the Local Mail must-use plugin.
 */

require_once WPMU_PLUGIN_DIR . '/local-mail/local-mail.php';
```

WordPress loads the root-level loader automatically, and the loader includes the real Local Mail plugin from its `local-mail` directory.

## Default admin credentials

Unless overridden, the installer creates this WordPress administrator:

```text
Username: admin
Password: password
```

For a site called `bricks-content-toolkit`, the dashboard is available at:

```text
https://bricks-content-toolkit.test/wp-admin
```

Change the password after installation:

```bash
cd ~/Herd/bricks-content-toolkit
wp user update admin --user_pass='Choose-A-Strong-Local-Password'
```

## Override credentials for one site

Set environment variables in front of the command:

```bash
WP_ADMIN_USER=mikestone \
WP_ADMIN_PASSWORD='Choose-A-Strong-Local-Password' \
WP_ADMIN_EMAIL='mikestone@example.test' \
herd-wp-new client-demo 3307
```

These values apply only to that one command. They do not permanently change the script.

## Debugging

The script writes the following values to `wp-config.php`:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

This records WordPress and PHP debugging information in:

```text
wp-content/debug.log
```

while preventing notices and warnings from being displayed directly on pages.

Watch the debug log during development:

```bash
cd ~/Herd/bricks-content-toolkit
tail -f wp-content/debug.log
```

## Verification

From a generated site folder, verify WordPress and the installed plugins:

```bash
cd ~/Herd/bricks-content-toolkit

wp core is-installed
wp option get siteurl
wp plugin list --status=active
wp plugin list --status=must-use
```

Verify the Local Mail files:

```bash
find wp-content/mu-plugins -maxdepth 2 -type f
```

Expected files:

```text
wp-content/mu-plugins/local-mail-loader.php
wp-content/mu-plugins/local-mail/local-mail.php
```

## Troubleshooting

### Database server or port is unavailable

The script checks the requested server before beginning installation. Confirm the selected service is running in DBngin and check the port.

For example, test MariaDB on port 3307:

```bash
mysql -h 127.0.0.1 -P 3307 -u root -e "SELECT VERSION();"
```

### `Undefined constant "truee"`

This is a typo in `wp-config.php`. The valid Boolean values are `true` and `false`, with no quotes and no extra letters.

Correct values:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
```

### Command not found: `herd-wp-new`

Check that the file exists and is executable:

```bash
ls -l ~/bin/herd-wp-new
```

Then reload zsh:

```bash
source ~/.zshrc
```

Confirm your path includes the personal scripts folder:

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

- This is a local-development tool. Do not use `admin` / `password` for a public or production website.
- Keep database credentials and `.env` files out of Git repositories.
- Query Monitor is a conventional WordPress plugin and appears in the normal Plugins screen.
- Local Mail is a must-use plugin and loads automatically; it does not need activation in the Plugins screen.
- If you create an existing site over HTTP and later move it to HTTPS, use `wp search-replace` carefully. Fresh sites do not normally require a search-and-replace because the script sets the final HTTPS `home` and `siteurl` values immediately.
