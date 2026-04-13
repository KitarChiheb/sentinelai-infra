# Runbook: Hostname Change Procedure
## Purpose
* This runbook describes the safe procedure for changing the hostname on a Ubuntu 24.04 server running under WSL2,including preventing WSL2 from reverting changes on restart.
## Prerequisites
* You must have sudo access.
* You must have a backup of /etc/hosts and /etc/wsl.conf before starting,use these commands sudo cp /etc/hosts /etc/hosts.bak,sudo cp /etc/wsl.conf /etc/wsl.conf.bak
* You must know the new hostname you are setting.
* You must know that WSL2 will need a restart to confirm persistence of the generateHosts setting.
## Risk Assessment
* Sudo will throw name resolution warnings if /etc/hosts is not updated before the hostname changes.
* WSL2 may fail to restart systemd on next boot if wsl.conf is misconfigured.
* All systemd-managed services can stop working if the [boot] section is accidentally removed from wsl.conf.
## Procedure
###Step 1 — Verify current state
* Run the following commands and record the output before making any changes, this output becomes your rollback reference.
* hostname, cat /etc/hostname, cat /etc/hosts, cat /etc/wsl.conf
* Expected: hostname returns the current machine name,Both files reflect the same value,wsl.conf shows the existing configuration without [network] section.
* If wsl.conf does not exist, note this step2 will create it from scratch.
###Step 2 — Disable WSL2 auto-generation (wsl.conf)
* For editing i used nano (you ca use any editor of your choice)
* Open the wsl.conf file using sudo nano /etc/wsl.conf that will create the file (if it doesnt exists) and open it for edit.
* Expected: if the file exists you will find [boot] and [user] sections add [network] section with generateHosts = false 
* If the file does not exist make sure to add [boot] with systemd=true and [network] sections, preserve any existing sections — do not remove [boot] or [user] if they already exist.
###Step 3 — Edit /etc/hosts first
* Run the following commands to edit /etc/hosts  
* sudo nano /etc/hosts,change entry mapping 127.0.1.1 to both newhostname.localdomain and newhostname
* Expected: cat /etc/hosts shows updated /etc/hosts with correct entry mapping
* If the old mapping still exists that can cause errors make sure to remove it
###Step 4 — Apply hostname change via hostnamectl
* After editing /etc/hosts run this command to apply hostname change.
* sudo hostnamectl set-hostname newhostname
* Expected: hostnamectl shows the new static hostname
###Step 5 — Verify
* Run the following commands to verify
* hostname, hostname -f, hostnamectl, cat /etc/hostname, cat /etc/hosts
* if these commands dont refect reverify your steps, rollback if neccessary.
###Step 6 — Restart WSL2 to confirm persistence
* Open a Windows PowerShell or CMD terminal (not WSL2), run wsl --shutdown, then reopen the WSL2 terminal. After reopening, re-run the Step 5 verification commands to confirm the hostname persisted.
## Rollback Procedure
* restore the original hostname:
###Step1 - Restore the original /etc/hosts
* If you have backups as mentioned in prerequisites do sudo cp /etc/hosts.bak /etc/hosts.
* If not,Run sudo nano /etc/hosts to change the entry mapping 127.0.1.1 to the original mappings
###Step2 - Change(restore) the original hostname via hostnamectl
* Run the following commande to apply(restore) hostname change:
* hostnamectl set-hostname originalhostname
###Step3 - Restore the original wsl.conf file
* If you have backups as mentioned in prerequisites do sudo sudo cp /etc/wsl.conf.bak /etc/wsl.conf.
* If not open the wsl.conf file using sudo nano /etc/wsl.conf 
* remove the [network] block from wsl.conf and add comment in the top accordingly. 
###Step 4 — Verify and Restart WSL2
* Same as Procedure section above
## Notes & Known Issues
* change the /etc/hosts file before using hostenamectl command to avoid unable to resolve host warning
* change the top comments of /etc/hosts file according to the change
