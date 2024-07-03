## Automation of User and Group Management using Bash Script

## Objective
Automate the process of onboarding new developers by efficiently managing user and group creation on a Unix-based system. This script, create_users.sh, reads a text file containing employee usernames and their respective group names, creating users and groups as specified. It ensures proper setup of home directories with appropriate permissions and ownership, generates random passwords for the users, and securely stores these passwords. 

## Step 1 - Write the script
```
#!/bin/bash

# Log file and password storage
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.txt"

# Ensure the /var/secure directory exists
if [ ! -d "/var/secure" ]; then
    mkdir -p /var/secure
    chmod 700 /var/secure
fi
# Ensure the log file and password file exist and have correct permissions
touch $LOG_FILE
touch $PASSWORD_FILE
chmod 600 $PASSWORD_FILE

# Function to log messages
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Check if the file is supplied
if [ $# -ne 1 ]; then
    log_message "ERROR: No input file supplied"
    echo "Usage: $0 <name-of-text-file>"
    exit 1
fi

INPUT_FILE=$1

# Check if input file exists
if [ ! -f $INPUT_FILE ]; then
    log_message "ERROR: Input file does not exist"
    echo "ERROR: Input file does not exist"
    exit 1
fi

# Read the input file line by line
while IFS=';' read -r username groups; do
    # Trim whitespace
    username=$(echo $username | xargs)
    groups=$(echo $groups | xargs)

    # Skip empty lines
    if [ -z "$username" ]; then
        continue
    fi

    # Create a personal group for the user
    if ! getent group $username >/dev/null; then
        groupadd $username
        log_message "Group $username created."
    else
        log_message "Group $username already exists."
    fi

    # Create the user with the personal group
    if ! id -u $username >/dev/null 2>&1; then
        useradd -m -g $username $username
        log_message "User $username created."
    else
        log_message "User $username already exists."
    fi

    # Assign the user to additional groups
    IFS=',' read -ra ADDR <<< "$groups"
    for group in "${ADDR[@]}"; do
        group=$(echo $group | xargs)
        if [ ! -z "$group" ]; then
            if ! getent group $group >/dev/null; then
                groupadd $group
                log_message "Group $group created."
            fi
            usermod -aG $group $username
            log_message "User $username added to group $group."
        fi
    done

    # Generate a random password
    password=$(openssl rand -base64 12)

    # Set the user's password
    echo "$username:$password" | chpasswd
    log_message "Password set for user $username."

    # Store the password securely
    echo "$username,$password" >> $PASSWORD_FILE
done < "$INPUT_FILE"

# Set the correct permissions on the password file
chmod 600 $PASSWORD_FILE

log_message "User creation process completed."

echo "User creation process completed. Check $LOG_FILE for details."

```
## Step 2 - Make the Script executable

`chmod +x create_users.sh`

## Step 3 - Test the script
To test this script, create a .txt file and add names of users as well as their roles/groups.

`nano users.txt`

Enter the names of users and their groups
```
wendy; engineering,webteam
florenceokoli; admins, dev_team
chi; support
```

## Step 4 - Run the Script with the Text File
To confirm that this script is running, run this command on your terminal:

`sudo ./create_users.sh users.txt`

This will be the outcome if it was successful
```

```
# Conclusion
The provided script facilitates the automated creation and management of user accounts in Unix/Linux systems, streamlining the process of assigning users to specific groups based on operational roles. 



