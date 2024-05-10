Postmortem

Date: 10 May
Duration: 1 Day
Issue Summary

At approximately 00:07 PST, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server during the release of Holberton School's System Engineering & DevOps project 0x19. GET requests to the server resulted in 500 Internal Server Error responses, instead of the expected HTML file for a simple Holberton WordPress site.
Debugging Process

Brennan (BDB), the assigned bug debugger, started investigating the issue at around 19:20 PST and followed these steps to debug the problem:

    Checked running processes using the ps aux command, confirming that both root and www-data Apache processes were running correctly.

    Examined the /etc/apache2/sites-available directory and determined that the web server was serving content from /var/www/html/.

    Ran strace on the PID of the root Apache process in one terminal and made a curl request to the server in another terminal. Unfortunately, the strace output did not provide useful information.

    Repeated step 3, but this time using the PID of the www-data process. This time, the strace output revealed an -1 ENOENT (No such file or directory) error when attempting to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

    Conducted a manual search through the files in the /var/www/html/ directory, using Vim pattern matching to identify the erroneous .phpp file extension. The issue was found in the wp-settings.php file at line 137 (require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).

    Corrected the typo by removing the trailing p from the file name.

    Tested another curl request on the server, and this time it returned a successful response with a status code of 200.

    Developed a Puppet manifest named 0-strace_is_your_friend.pp to automate the fix for this specific error.

Root Cause Analysis

The root cause of the outage was a typo in the WordPress application's wp-settings.php file. The file was attempting to load the non-existent file class-wp-locale.phpp instead of the correct file class-wp-locale.php. This error led to the critical failure of the WordPress application.
Resolution and Preventive Measures

To resolve the issue, the typo in the wp-settings.php file was corrected by removing the trailing p in the file name. Additionally, a Puppet manifest (0-strace_is_your_friend.pp) was created to automate the fix for any similar errors that might occur in the future.

To prevent similar outages in the future, the following preventive measures are recommended:

    Conduct thorough testing of the application before deployment to catch errors like this early on.

    Implement status monitoring using tools such as UptimeRobot to receive immediate alerts in case of website outages.

By following these recommendations, future occurrences of such application errors can be minimized, ensuring smoother operations.