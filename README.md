# Acuity WordPress User Import
Connects a Backdrop CMS installation to a legacy WordPress database to migrate user accounts and metadata.

This module is designed to be an idempotent migration tool: it can be run multiple times to sync changes from WordPress to Backdrop, updating existing users and creating new ones as needed.

When all users have been imported and/or your new backdrop site is live, you can safely uninstall this module.

To manipulate the user accounts we recommend using the **Views Bulk Operations** module, for example, to change the user role from the standard *Authenticated* to a custom role you may have created, or update an email or newsletter subscription status.

We recommend deleteing any unneeded fields we created for the import process once you are happy with the imported data.

## Initial version

This is an beta release for code review and testing.

### Compatibility notes:
This module is a utility intended for migration purposes. It is designed to work in tandem with Acuity Auth Handler (abms_auth) to allow migrated users to log in using their legacy passwords.

### Requirements:

- Backdrop CMS 1.x
- A secondary database connection defined in settings.php
- abms_auth (Recommended for password compatibility)

## Installation:

Install this module using the official Backdrop CMS instructions at https://docs.backdropcms.org/documentation/extend-with-modules

Add the external database configuration to your settings.php file (see Documentation below).

Enable the module.

## Documentation:

1. **Database Configuration**

 - Before running the migration, you must define the connection to the WordPress database in your site's settings.php. The prefix must be empty to query wp_users directly.
 - 'host' could be 127.0.0.1, localhost or a URL to the database.

```
$databases['default']['default'] = array(
  'database' => 'your_backdrop_db_name',
  'username' => 'db_user',
  'password' => 'db_pass',
  'host' => '127.0.0.1',
  'prefix' => '',
);

$databases['wordpress']['default'] = array(
  'driver' => 'mysql',
  'database' => 'your_wordpress_db_name',
  'username' => 'db_user',
  'password' => 'db_pass',
  'host' => '127.0.0.1',
  'prefix' => '',
);
```

2. **Running the Migration**

 - Navigate to Configuration > Acuity Utilities > WP User Migration (admin/config/acuity/wpuser).

 - Enter the database key defined in settings.php (e.g., wordpress).

 - Click Test Connection & Count Users to verify connectivity.

 - Click Run Migration to begin the batch import.


3. **Data Mapping**

The module automatically creates hidden fields to store legacy data.

| WordPress Field | Backdrop Field | Visibility | Description |
| --- | --- | --- | --- |
| ID | field_wp_guid | Hidden | Integer. Used for syncing. |
| user_login | name | Public | Unique username. |
| user_email | mail | Private | Unique email. |
| user_pass | pass | Private | Raw injection of legacy hash. |
| display_name | field_display_name | Public | Preferred public name. |
| first_name | field_first_name | Public | Meta field. |
| last_name | field_last_name | Public | Meta field. |
| description | field_wp_bio | Public | Biographical info. |
| wp_user_level | field_wp_user_level | Hidden | Legacy permission level. |
| wp_capabilities | field_wp_capabilities | Hidden | Serialized roles array. |

## Security Implications & Safeguards

### Password Handling

**Injection:** Passwords are injected as raw WordPress hashes (```$P$```... or ```$H$```...). If you have enabled our ***Acuity WP Auth Handler*** module, the user's password will be converted to a Backdrop password hash when they first log in.

**Safety Check:** If a user has already logged into Backdrop and upgraded to a secure hash (```$S$```...), re-running this migration will not overwrite their password.

**User 1:** The Backdrop Root Administrator (User 1) is never touched by the password update logic.

### Field Access

Technical fields (field_wp_guid, field_wp_capabilities, field_wp_user_level) are restricted via *hook_field_access()*. Only administrators with administer users permission can view or edit these fields.

### Silent Import

The module explicitly suppresses "Account Created" emails ($account->notify = FALSE) to prevent mass spamming during import.

### Content Risks (XSS)

Bio data (field_wp_bio) is imported raw from WordPress. If the source site contained malicious scripts in user bios, they are now in your database. Ensure the Text Format used to display this field filters HTML tags appropriately.

## Issues:

Bugs and Feature requests should be reported in the Issue Queue: https://github.com/backdrop-contrib/abms_wpuser/issues

## Current Maintainer(s):
- Steve Moorhouse (albanycomputers) (https://github.com/albanycomputers)
- Seeking additional maintainers and contributors.

## Credits:

- Google Gemini 3.0 assisted with the coding of this module.

## Sponsorship:
- Albany Computer Services (https://www.albany-computers.co.uk)
- Albany Web Design (https://www.albanywebdesign.co.uk)
- Albany Hosting (https://www.albany-hosting.co.uk)

## License
This project is GPL v2 software. See the LICENSE.txt file in this directory for complete text.
