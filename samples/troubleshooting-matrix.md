# Troubleshooting Guide: Resolving WordPress LMS Server Crashes (HTTP 500)

## 📋 Context
* **Goal:** Provide a logical diagnostic pathway for IT infrastructure teams to isolate and fix server crashes during high-traffic student enrollment periods.
* **Target Audience:** Systems Administrators, DevOps Engineers, and Tier-2 Technical Support Specialists.
* **Tools Used:** Markdown, Linux Terminal, SSH, WP-CLI, PHP Error Logs.

---

### 📌 Overview
During peak enrollment hours, database connection spikes or corrupted plugin updates can trigger an `HTTP 500 Internal Server Error`. This guide outlines how to trace the error logs, isolate the root cause, and safely restore the learning management platform without losing user progress data.

### 🛑 Safety First: Before Modifying Production Files
Always take a manual database backup via SSH before executing command-line changes:
```bash
mysqldump -u [db_user] -p [db_name] > lms_backup_\$(date +%F).sql
```

---

### 🔍 Diagnostic Matrix

| Symptom | Probable Cause | Diagnostic Command / Step | Resolution Procedure |
| :--- | :--- | :--- | :--- |
| **"Error Establishing a Database Connection"** page. | MySQL service crashed or `wp-config.php` credentials are incorrect. | Run via SSH:<br>`systemctl status mysql` | 1. If service is down, restart it:<br>`sudo systemctl restart mysql`<br>2. Verify DB credentials match in `wp-config.php`. |
| **Blank White Screen** (White Screen of Death) on course checkout pages. | Fatal PHP script error caused by memory exhaustion or plugin conflicts. | Inspect the live error log:<br>`tail -n 50 /var/log/nginx/error.log` | 1. Increase PHP memory limit in `wp-config.php` by adding:<br>`define('WP_MEMORY_LIMIT', '512M');` |
| **HTTP 500** immediately after activating a new LMS sub-module plugin. | PHP fatal compilation error due to plugin-to-plugin incompatibility. | Use WP-CLI to check active list:<br>`wp plugin list --status=active` | 1. Force-deactivate the offending plugin via CLI:<br>`wp plugin deactivate [plugin-slug]` |
| **HTTP 500** occurs only when uploading heavy student video assignments. | Server execution timeouts or low post limits in PHP configuration. | Check variables in terminal:<br>`php -i | grep -E "upload_max|post_max"` | 1. Edit the server's `php.ini` file:<br>`upload_max_filesize = 128M`<br>`post_max_size = 128M`<br>2. Restart PHP-FPM service. |

---

### 🛠️ Advanced Isolation Workflow: Enabling Debug Mode

If the matrix above does not isolate the crash, you must force WordPress to output errors directly to a secure local log file instead of displaying generic 500 errors to students.

1. Connect to your server root directory via SSH or SFTP.
2. Open the `wp-config.php` file using a command-line editor:
   ```bash
   nano wp-config.php
   ```
3. Locate the line that says `define( 'WP_DEBUG', false );` and change it to `true`.
4. Append the following security directives immediately below it:
   ```php
   // Enable debug logging to /wp-content/debug.log
   define( 'WP_DEBUG_LOG', true );

   // Prevent public front-end display of errors to students
   define( 'WP_DEBUG_DISPLAY', false );
   @ini_set( 'display_errors', 0 );
   ```
5. Save the file (`Ctrl+O`, then `Enter`) and exit (`Ctrl+X`). 
6. Re-trigger the error on the browser and read the precise line of broken code using:
   ```bash
   tail -f wp-content/debug.log
   ```
