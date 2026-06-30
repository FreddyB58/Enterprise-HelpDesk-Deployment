# Enterprise IT Help Desk Deployment & Lifecycle Management (osTicket)

## Objective
The purpose of this project is to demonstrate the deployment, configuration, and administration of a full-scale IT Help Desk ticketing system using osTicket. This lab highlights hands-on experience with Linux command-line administration, database configuration, web server management (LAMP Stack), Role-Based Access Control (RBAC), and the end-to-end lifecycle of IT service requests—culminating in a Tier 2 physical network hardware escalation.

### Skills Demonstrated
* **Systems Administration:** Deployment of web applications via Ubuntu Linux CLI, dependency management (`apt`), and managing file permissions (`chmod`, `chown`).
* **LAMP Stack Troubleshooting:** Resolving Apache Multi-Processing Module (MPM) conflicts and diagnosing PHP/MySQL communication errors via raw system logs.
* **Service Management:** Creation of Help Topics, SLAs, and routing rules to automate enterprise workflows.
* **Access Control:** Configuration of Primary and Extended Access roles to ensure proper segregation of duties across IT departments.
* **Ticket Lifecycle:** Managing the flow of a ticket from user creation to agent resolution, and documenting hardware-level escalations for networking equipment.

### Tools & Technologies
* **Operating System:** Ubuntu Linux (Virtual Machine)
* **Web Stack:** Apache2, MySQL, PHP 8.5 (LAMP)
* **Ticketing Software:** osTicket v1.18.1

---

## Phase 1: LAMP Stack Provisioning & Web Server Setup
The deployment began by utilizing the Linux terminal to update the local package repositories and provision the core web server infrastructure. 

![Updating Ubuntu and Installing Apache](1_Ticketing%20system%20update.png)
*Updating Ubuntu and installing the Apache2 web server.*

![Apache Default Page](2_Active%20Server.png)
*Verifying Apache2 is active and successfully serving the default HTML page via localhost.*

![Creating MySQL Database](3_SQL%20Code_1.png)
*Accessing the MySQL monitor via CLI to create the `osticket_db` database and provision a dedicated administrative user with full privileges.*

![Downloading osTicket](3_SQL%20Code_2.png)
*Fetching the osTicket release package directly from GitHub via `wget`.*

![Modifying Permissions](3_SQL%20Code_3.png)
*Applying proper web ownership (`www-data`) and modifying read/write permissions (`chmod 0666`) to the core configuration file prior to web installation.*

![Extracting Files](3_SQL%20Code_4.png)
*Unzipping the application files into the `/var/www/html/osticket` directory.*

---

## Phase 2: Resolving PHP & Apache MPM Conflicts
During the initial LAMP stack configuration, the Apache web server failed to process PHP correctly due to a Multi-Processing Module (MPM) conflict. Ubuntu defaults to `mpm_event`, which is incompatible with the standard `mod_php`. 

Manual intervention in the CLI was required to disable the conflicting modules, enable `mpm_prefork`, and install the correct PHP 8.5 dependencies.

![Installing PHP Packages](Code%20for%20PHP%20for%20osTicket%201.png)
*Initial attempts to install PHP dependencies.*

![Locating libapache2-mod-php](Code%20for%20PHP%20for%20osTicket%202.png)
*Successfully locating and installing the `libapache2-mod-php` package.*

![Unpacking PHP Dependencies](Code%20for%20PHP%20for%20osTicket%203.png)
*Unpacking PHP 8.5 dependencies and watching the system automatically attempt to switch Apache to `prefork`.*

![Disabling MPM Event](Code%20for%20PHP%20for%20osTicket%204.png)
*Disabling `mpm_event` and `mpm_worker`.*

![Enabling Prefork](Code%20for%20PHP%20for%20osTicket%205.png)
*Executing the final configuration: Enabling `mpm_prefork`, explicitly enabling `php8.5`, and restarting the `apache2` service.*

![Missing MySQLi Extension 1](osTicket%20working%20via%20firefox%20before%20SQL%20fixed.png)
![Missing MySQLi Extension 2](osTicket%20working%20via%20firefox%20before%20SQL%20fixed%202.png)
*Initial browser check revealing the web server was missing the critical MySQLi extension required to communicate with the database.*

![Installing PHP MySQL Driver](Installing%20php8.5mysql.png)
*Installing the missing `php8.5-mysql` driver via the terminal.*

![Prerequisites Passed](osTicket%20fixed%20with%20MySQLi%20extension%20working%203.png)
*Validating all server prerequisites are successfully met.*

---

## Phase 3: GUI Installation & Database Configuration
With the web server stabilized, the setup transitioned to the front-end graphical installer to bind the application to the MySQL database.

![Installer Setup](osTicket%20Installer%201.png)
*Beginning the basic installation.*

![Admin Configuration](osTicket%20Installer%202.png)
*Configuring the Administrator account and help desk URLs.*

![Database Connection](osTicket%20Installer%203%20COMPLETE.png)
*Defining the MySQL database connection parameters.*

### Troubleshooting: Database Authentication Failure
Immediately after executing the installer, the installation failed. Instead of relying on the GUI, raw Apache error logs were queried via the terminal to diagnose the root cause.

![Reading Apache Error Logs](Read%20error%20log%20after%20failed%20osTicket%20install.png)
*Diagnosing a PHP Fatal Error: `Uncaught mysqli_sql_exception: Access denied`. The MySQL password/privileges were corrected, and the installation proceeded successfully.*

<img width="1211" height="762" alt="osTicket Congratulations screen" src="https://github.com/user-attachments/assets/ddcf7fd4-85a2-48c6-bc02-5b38c0bab5b5" />
*Successful installation confirmation.*

Following setup, the environment was locked down for production. The `setup` directory was removed, and `ost-config.php` file permissions were restricted to `0644`.

---

## Phase 4: Help Desk UI & Access Control (RBAC)
The system was configured for enterprise use, establishing separate portals for end-users and IT support staff, alongside strict Role-Based Access Control (RBAC).

![End User Portal](Ticketing%20system%20for%20employees.png)
*The public-facing Support Center portal where end-users submit requests.*

![Staff Login](Successful%20login%20to%20osTicket.png)
*The secure Agent authentication portal.*

![Agent Panel Dashboard](osTicketing%20Management%20system%20for%20IT.jpg)
*The backend Staff Control Panel dashboard.*

![Agent Settings](image_084404.png)
*Configuring IT Staff profiles and assigning "Extended Access" to ensure specific agents have visibility across multiple departmental queues.*

---

## Phase 5: Ticket Lifecycle - Tier 1 Support
To validate the system's routing capabilities, an end-user was simulated reporting an Active Directory authentication failure.

![Submitting AD Lockout](Create%20a%20ticket%20Test%201.png)
*Simulating an employee submitting a High-Priority Password Reset ticket.*

![Ticket in Queue](New%20Password%20reset%20ticket%20created%20by%20Greg%20Smith.png)
*Validating the ticket successfully bypassed automated spam filters and landed directly in the primary IT Support queue.*

![Agent Reply](Replying%20to%20Gregs%20ticket_Closing%20ticket_1.png)
*Acting as the IT Support Agent: Resetting the Active Directory password, forcing a domain sync, and providing a temporary password to the user.*

![Ticket Closed](Closed%20Gregs%20ticket_Closing%20ticket_2.png)
*Validating the ticket was successfully resolved and removed from the Open queue.*

---

## Phase 6: Advanced Routing & Tier 2 Network Escalation
To demonstrate advanced enterprise workflows, a specialized `Tier 2 - Network Engineering` department was created to handle physical infrastructure failures.

![Sales Floor Offline Ticket](Create%20a%20ticket%20Test%202.png)
*An end-user submits a critical ticket reporting a total loss of network connectivity on the sales floor, noting an amber system LED on the IDF closet switch.*

![Agent Viewing Ticket](Escalate%20Ticket%20submitted_1.png)
*The Tier 1 agent reviews the ticket and identifies a hardware-level failure on a Cisco Catalyst 2960-C switch.*

![Transferring Ticket](Escalating%20Ticket_2.png)
*Executing the escalation: The ticket is officially transferred out of the standard Support queue and into the Tier 2 Network Engineering department, complete with internal engineering handoff notes for hardware diagnostics.*
