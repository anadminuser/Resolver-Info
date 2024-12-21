# DNS Resolver Information Script

## Overview

This script enhances common terminal commands like `curl` and `ping` by displaying the DNS resolver used to resolve the hostname. It checks whether the hostname was resolved using:

1. `/etc/hosts`
2. A specific DNS server

### Key Features
- Automatically detects if `/etc/hosts` was used for resolution.
- Queries configured DNS servers to determine which was used.
- Works seamlessly as a wrapper for `curl` and `ping` commands.

---

## Setup

### Prerequisites
- macOS or a Unix-like operating system with the following tools installed:
  - `scutil` (default on macOS)
  - `dig`
  - `awk`

### Installation
1. Copy the script into your shell configuration file (e.g., `~/.zshrc` or `~/.bashrc`).

   ```bash
   resolve_source() {
       local hostname="$1"

       # Check if the hostname is in /etc/hosts
       if grep -q "$hostname" /etc/hosts; then
           echo "Resolved using /etc/hosts"
           return
       fi

       # Use scutil to determine DNS server and query resolution
       local dns_servers
       dns_servers=$(scutil --dns | grep "nameserver\[[0-9]*\]" | awk '{print $3}')

       # Use dig to query the hostname
       for server in $dns_servers; do
           if dig @"$server" +short "$hostname" > /dev/null 2>&1; then
               echo "Resolved using DNS server: $server"
               return
           fi
       done

       echo "Resolver source unknown"
   }

   resolver_info() {
       local hostname="$1"
       resolve_source "$hostname"

       # Run the actual command
       shift
       "$@"
   }

   curl_resolver() {
       local host="$(echo "$1" | sed 's|http[s]*://||' | cut -d/ -f1)"
       resolver_info "$host" curl "$@"
   }

   ping_resolver() {
       resolver_info "$1" ping "$@"
   }

   alias curlr="curl_resolver"
   alias pingr="ping_resolver"
   ```

2. Save the file.
3. Reload your shell configuration:
   ```bash
   source ~/.zshrc  # or ~/.bashrc depending on your shell
   ```

---

## Usage

### Wrapped Commands
The script provides the following aliases:
- `curlr`: Enhanced `curl` command with resolver information.
- `pingr`: Enhanced `ping` command with resolver information.

#### Examples

**Curl Example:**
```bash
curlr http://google.com
```
Output:
```
Resolved using DNS server: 8.8.8.8
[Standard curl output]
```


```bash
curlr http://domaininmyhostfile.com
```
Output:
```
Resolved using /etc/hosts
[Standard curl output]
```


**Ping Example:**
```bash
pingr google.com
```
Output:
```
Resolved using DNS server: 8.8.8.8
PING google.com (127.0.0.1): 56 data bytes
[Standard ping output]
```

```bash
pingr http://domaininmyhostfile.com
```
Output:
```
Resolved using /etc/hosts
[Standard ping output]
```

---

## Notes

1. The script uses `scutil --dns` to retrieve DNS server information on macOS. On other systems, you may need to replace this with an equivalent command (e.g., `systemd-resolve --status` or `resolvectl`).
2. Ensure `dig` is installed and available in your `PATH`. On macOS, you can install it via Homebrew:
   ```bash
   brew install bind
   ```

---

## Customization

Feel free to modify the aliases or add new functions for other commands that involve hostname resolution. For example, you could extend it to `wget` or `ssh`.

---

## Troubleshooting

- **DNS resolution issues:** Ensure your system's DNS settings are configured correctly.
- **`dig` not found:** Install `dig` using your system's package manager (e.g., Homebrew, apt, or yum).

---

## License
This script is open source and provided "as is" without warranty. Feel free to modify and distribute it under the MIT License.

