# X-Domain password reset - workstation script - v1.0 - Simon Burbery - October 2022

# : For those odd occasions where you cannot install anything and need to syncrhonise passwords between source and destination domains
# : Can also be used [with modification of target] to update 3rd party application or SaaS credentials usng APIs

# ** Admin server share must be accessible; if not the script will exit assuming the machine is offline.

# There are two components:
# 1. this workstation script; runs as a scheduled task which is deployed via GPO to run on workstations in the source domain
# 2. the server script; a scheduled task running on any 2016+ member server in the source domain that can contact source and target DC; requires AD PowerShell module

# Workstation:
# - set the variable values in the 'UPDATE THESE VALUES' section below
# - task runs during workstation logon and unlock
# - user is prompted when expiry is less than "$pswddays" away
# - current password is verified against source domain before allowing the change
# - new password is verified then encrypted using a key and saved to a randomly named text file on "$fileshare"
# - monitor server share until file is deleted, confirm access to both domains before reporting success

# Server:
# - available at https://github.com/simonburbery/x-domain-pswd-reset-server
# - task runs as a dedicated service account with permission to reset passwords in both domains (no other privileges required)
 # ideally a 1-way trust is in place, but in testing was found to work as long as the user was created in both domains with the same username & pswd
# - runs every 5 seconds to process new file(s) created in "$fileshare"
# - username and encrypted password are read from file then converted to a secure credential using a random key (the key is rolled after a pswd reset occurs)
# - password reset is performed against domain controllers (set as variables in the server script) in source and target domains
# - if successful, the processed csv file(s) are deleted
# - if not successful, write $username.err file (to notify workstation) then delete the csv

# * Both workstation and server scripts log actions and errors to the eventlog files in "$fileshare"
# * There are also PowerShell transcripts (generally empty but there for troubleshooting).

