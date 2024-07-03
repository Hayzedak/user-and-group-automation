# Automating User and Group Management with a Bash Script

This is an [HNG](https://hng.tech/internship) internship task for a DevOps engineer. Managing user accounts and groups is a crucial job for system administrators, especially in environments where many new users are frequently added. Automating this process can save significant time and reduce the risk of human error. 

In this article, we will demonstrate how to automate user and group management using Bash script. This script will read a text file containing usernames and group names, create the users and groups as specified, set up home directories, generate random passwords, and log all actions.

## Script Overview

The script **`create_users.sh`** performs the following tasks:

1. Reads a text file where each line contains a username and a list of groups, separated by a semicolon (**`;`**).

2. Creates a personal group for each user.

3. Creates user accounts with their respective personal groups.

4. Adds users to additional groups as specified.

5. Sets up home directories with appropriate permissions.

6. Generates random passwords for each user.

7. Logs all actions to **`/var/log/user_management.log`**.

8. Stores the generated passwords securely in **`/var/secure/user_passwords.txt`**.


## Prerequisites

Ensure you have the necessary permissions to create users, groups, and modify system files. The script needs to be executed with superuser privileges.

## The Bash Script: **`create_users.sh`**

```
#!/bin/bash

# Log file location
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.txt"

# Create secure directory if it doesn't exist
mkdir -p /var/secure
chmod 700 /var/secure

# Function to log messages
log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Function to generate a random password
generate_password() {
    openssl rand -base64 12
}

# Function to create user and groups
create_user_and_groups() {
    local user="$1"
    local groups="$2"

    # Create personal group with the same name as the user
    if ! getent group "$user" > /dev/null; then
        groupadd "$user"
        log_message "Group $user created."
    else
        log_message "Group $user already exists."
    fi

    # Create the user with their personal group
    if ! id -u "$user" > /dev/null 2>&1; then
        useradd -m -g "$user" -s /bin/bash "$user"
        log_message "User $user created."
    else
        log_message "User $user already exists."
    fi

    # Set home directory permissions
    chmod 700 /home/"$user"
    chown "$user":"$user" /home/"$user"
    log_message "Home directory permissions set for $user."

    # Add user to additional groups
    if [ -n "$groups" ]; then
        IFS=',' read -ra group_array <<< "$groups"
        for group in "${group_array[@]}"; do
            group=$(echo "$group" | xargs) # Trim whitespace
            if ! getent group "$group" > /dev/null; then
                groupadd "$group"
                log_message "Group $group created."
            fi
            usermod -aG "$group" "$user"
            log_message "User $user added to group $group."
        done
    fi

    # Generate and set password
    local password=$(generate_password)
    echo "$user:$password" | chpasswd
    log_message "Password set for user $user."

    # Save the password securely
    echo "$user:$password" >> $PASSWORD_FILE
    chmod 600 $PASSWORD_FILE
}

# Check if the input file is provided
if [ -z "$1" ]; then
    echo "Usage: $0 <user_file>"
    exit 1
fi

# Read the input file
while IFS=';' read -r user groups || [ -n "$user" ]; do
    user=$(echo "$user" | xargs) # Trim whitespace
    groups=$(echo "$groups" | xargs) # Trim whitespace
    [ -z "$user" ] && continue # Skip empty lines
    create_user_and_groups "$user" "$groups"
done < "$1"

log_message "User creation script completed."
```

## Preparing the Input File

Create a text file named **`user_list.txt`** with the following format:

```
azeez;developers,admins
hng;developers
nora;admins
```

Each line contains a username and a list of groups separated by a semicolon (**`;`**). Multiple groups are separated by commas (**`,`**).

## Running the Script

## Make the Script Executable:

`sudo chmod +x create_users.sh`

## Execute the Script:

`sudo ./create_users.sh user_list.txt`

## Verifying the Script Execution

## Check the Log File:

`sudo cat /var/log/user_management.log`

## Check the Password File:

`sudo cat /var/secure/user_passwords.txt`

## Verify User Accounts:

`cut -d: -f1 /etc/passwd | grep -E 'azeez|hng|nora'`

## Verify Group Membership:

```
groups azeez
groups hng
groups nora
```

## Conclusion

Automating user and group management with a Bash script can enhance the efficiency and accuracy of administrative tasks. This script provides solution for creating users, managing group memberships, setting up home directories, and ensuring secure password handling. By following this guide, system administrators can save time and reduce errors, particularly in environments with frequent user account changes.

You can also join [HNG](https://hng.tech/premium) next cohort to have access to real life tasks and connecting with experienced professionals.