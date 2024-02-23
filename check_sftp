#!/bin/bash

# Script version
SCRIPT_VERSION="check_sftp - John Gonzalez - v1.1.0"

# Default variables
PARTITION="/"
DISK_USAGE_THRESHOLD=85
CHECK_INTERVAL=5
SFTP_PROCESS_NAME="sftp-server"
SFTP_PORT=22

# Script usage
usage() {
    echo "Usage: $0 -P <partition> -d <disk_usage_threshold> -i <check_interval> [-n <sftp_process_name>] -p <sftp_port>"
    echo "Options:"
    echo "  -P <partition>                Partition to monitor (default: /)"
    echo "  -d <disk_usage_threshold>     Disk usage alert threshold in percent (default: 85)"
    echo "  -i <check_interval>           Interval between checks in seconds (default: 5)"
    echo "  -n <sftp_process_name>        Name of the SFTP process (default: sftp-server)"
    echo "  -p <sftp_port>                SFTP port (default: 22)"
    echo "  -h                            Show this help message"
    echo "  -v                            Show version"
    exit 1
}

# Options management
while getopts "P:d:i:n:p:hv" opt; do
    case $opt in
      P ) PARTITION=$OPTARG ;;
      d ) DISK_USAGE_THRESHOLD=$OPTARG ;;
      i ) CHECK_INTERVAL=$OPTARG ;;
      n ) SFTP_PROCESS_NAME=$OPTARG ;;
      p ) SFTP_PORT=$OPTARG ;;
      h ) usage ;;
      v ) echo "$SCRIPT_VERSION"; exit 0 ;;
      \? ) usage ;;
    esac
done

# Init output message and exit status
output_message=""
exit_status=0

# Function to check disk (if not separated partition) or partition usage for sftp
check_disk_usage() {
    local disk_usage_line=$(df -P "$PARTITION" | awk 'NR==2 {print $0}')
    local disk_usage=$(echo "$disk_usage_line" | awk '{print $5}' | sed 's/%//g')
    if [[ "$disk_usage" -lt "$DISK_USAGE_THRESHOLD" ]]; then
        output_message+="Disk Usage OK - $disk_usage% used on $PARTITION. "
    else
        output_message+="Disk Usage CRITICAL - $disk_usage% used on $PARTITION. "
        exit_status=2
    fi
}

# Function to check sftp service status
check_sftp_service() {
    if systemctl is-active sshd > /dev/null 2>&1; then
        output_message+="SFTP Service OK - SSHD is running. "
    else
        output_message+="SFTP Service CRITICAL - SSHD is not running. "
        exit_status=2
    fi
}

# Function to check active sftp connections status and connected users
check_sftp_connections() {
    local users=""
    local user_count=0

    if [[ "$SFTP_PROCESS_NAME" == "sftp-server" ]]; then
        # For default process name (sftp-server), use UID to identify active sessions
        local uids=$(ps -eo uid,comm | grep "$SFTP_PROCESS_NAME" | awk '{print $1}' | sort | uniq)

        for uid in $uids; do
            local user=$(getent passwd "$uid" | cut -d: -f1)
            if [[ -n "$user" && ! "$users" =~ "$user" ]]; then
                users+="$user, "
                ((user_count++))
            fi
        done
    else
        # Pour customized process, directly extract usernames
        local sftp_users=$(ps aux | grep "@${SFTP_PROCESS_NAME}" | grep -v grep | awk -F'sshd: ' '{print $2}' | awk -F'@' '{print $1}' | sort | uniq)

        for user in $sftp_users; do
            if [[ -n "$user" && "$user" != "|" ]]; then
                users+="$user, "
                ((user_count++))
            fi
        done
    fi

    users=${users%,} # Remove last comma

    if [[ $user_count -eq 0 ]]; then
        output_message+="SFTP Connections OK - No active connections. "
    else
        users=$(echo "$users" | sed 's/, $//')
        users="${users}."
        output_message+="SFTP Connections OK - $user_count active connection(s). Users: $users "
    fi
}

# Checks execution
check_disk_usage
sleep $CHECK_INTERVAL
check_sftp_service
check_sftp_connections

# Show final output and exit status
echo "$output_message"
exit $exit_status
