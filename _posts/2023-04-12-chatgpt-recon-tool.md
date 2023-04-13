---
title: "ChatGPT: Build me a Recon Tool!"
categories:
  - AI
---

Using ChatGPT to build a simple hacking recon tool

In Chapter 5 of [Bug Bounty Bootcamp](https://nostarch.com/bug-bounty-bootcamp), I talked about how you can write a simple bash script to automate recon tasks before hacking and bug bounty hunting. Then just a year later, ChatGPT came around.

I am still a huge proponent of learning to script so that you can understand how tools work and fine-tune open-source tools for yourself. But when it comes to translating your tooling ideas into code, you could probably let ChatGPT do the heavy lifting. So this time, I tried to replicate the recon tool I wrote in the book with ChatGPT.

Let's see what it comes up with! :)

## v1: basic functionality

I started by describing the simple tool we built in Chapter 5 of Bug Bounty Bootcamp to ChatGPT:

```
Please write a bash script that will:\
--- Look for open ports\
--- Look for subdomains\
--- Look for file paths\
On a target domain, and store the scan results to file.\
The script should be able to scan multiple domains at once,\
and produce human-readable report files.
```

Seems like a fairly simple prompt, here's what ChatGPT came back with:

```bash
#!/bin/bash

# Set the target domains as an array\
domains=("example.com" "test.com")

# Define the output directory\
output_dir="./scan_results"

# Loop through the target domains\
for domain in "${domains[@]}"\
do\
    # Create a directory for the current domain's results\
    mkdir -p "$output_dir/$domain"

    # Run the nmap command to scan for open ports\
    nmap -v -oN "$output_dir/$domain/open_ports.txt" "$domain"

    # Run the sublist3r command to scan for subdomains\
    sublist3r -d "$domain" -o "$output_dir/$domain/subdomains.txt"

    # Use Gobuster to scan for file paths\
    gobuster dir -u "https://$domain" -w /usr/share/wordlists/dirb/common.txt -o "$output_dir/$domain/file_paths.txt" -k\
done
```

This generated script scans the target domains for open ports, subdomains, and file paths. The results for each domain are stored in a separate directory within the `scan_results` folder. Here, ChatGPT assumed that I was using Kali Linux since the `/usr/share/wordlists/dirb/common.txt` is a wordlist included in Kali by default.

ChatGPT recommended using `Nmap`, `Sublist3r`, and `Gobuster` for the scan. It also provided the instructions for using the tool:

```
To use this script, save it to a file (e.g., scan_domains.sh),\
make it executable (chmod +x scan_domains.sh), and run it (./scan_domains.sh).\
Make sure to install the required tools (nmap, sublist3r, and gobuster)\
before running the script.
```

## v2: making it interactive

I wanted the tool to be interactive, so I instructed ChatGPT to change the script:

Please make the tool interactive, and make each scan type optional.

This newly generated script prompts the user for the target domains and scan types. The user can choose which scan types to run by entering 'y' or 'n' for each one.

```bash
#!/bin/bash

# Define the output directory\
output_dir="./scan_results"

# Prompt the user for the target domain(s)\
read -p "Enter the target domain(s) (comma-separated): " domain_list

# Convert the comma-separated list to an array\
IFS=',' read -ra domains <<< "$domain_list"

# Prompt the user for the scan type(s)\
read -p "Do you want to scan for open ports? [y/n]: " scan_ports\
read -p "Do you want to scan for subdomains? [y/n]: " scan_subdomains\
read -p "Do you want to scan for file paths? [y/n]: " scan_paths

# Loop through the target domains\
for domain in "${domains[@]}"\
do\
    # Create a directory for the current domain's results\
    mkdir -p "$output_dir/$domain"

    # Scan for open ports if requested\
    if [[ $scan_ports == "y" ]]\
    then\
        nmap -v -oN "$output_dir/$domain/open_ports.txt" "$domain"\
    fi

    # Scan for subdomains if requested\
    if [[ $scan_subdomains == "y" ]]\
    then\
        sublist3r -d "$domain" -o "$output_dir/$domain/subdomains.txt"\
    fi

    # Scan for file paths if requested\
    if [[ $scan_paths == "y" ]]\
    then\
        gobuster dir -u "https://$domain" -w /usr/share/wordlists/dirb/common.txt -o "$output_dir/$domain/file_paths.txt" -k\
    fi\
done
```

## v3: add notification functionality

Finally, I asked ChatGPT to change the tool so that it would automatically run every night, and notify me of new findings.

```
Schedule the script to run everyday at midnight,\
and notify me if it found anything different from the previous scan.
```

Here's the final version of the tool. The script compares the new scan results with previous ones using Git and sends an email to your email address if there are any differences.

```bash
#!/bin/bash

# Define the output directory\
output_dir="./scan_results"

# Prompt the user for the target domain(s)\
read -p "Enter the target domain(s) (comma-separated): " domain_list

# Convert the comma-separated list to an array\
IFS=',' read -ra domains <<< "$domain_list"

# Loop through the target domains\
for domain in "${domains[@]}"\
do\
    # Create a directory for the current domain's results\
    mkdir -p "$output_dir/$domain"

    # Scan for open ports\
    nmap -v -oN "$output_dir/$domain/open_ports.txt" "$domain"

    # Scan for subdomains\
    sublist3r -d "$domain" -o "$output_dir/$domain/subdomains.txt"

    # Scan for file paths\
    gobuster dir -u "https://$domain" -w /usr/share/wordlists/dirb/common.txt -o "$output_dir/$domain/file_paths.txt" -k\
done


# Compare the new scan results with the previous ones and send an email if there are any differences\
changes=$(git --git-dir="$output_dir/.git" --work-tree="$output_dir" diff HEAD~1 HEAD)

if [[ -n $changes ]]\
then\
    echo "$changes" | mail -s "Scan results for $domain have changed" youremail@example.com\
fi
```

ChatGPT also created a line you can put in your crontab to schedule nightly scans:

```bash
0 0 * * * /path/to/script.sh >> /path/to/logfile.log 2>&1
```

However, this script is not really usable since it's supposed to run without human intervention. Although ChatGPT removed the user prompts to select scan types, it is still prompting the user for target domains. After I pointed that out, ChatGPT fixed the script:

```bash
#!/bin/bash

# Define the target domains\
domains=("example.com" "test.com" "example.org")

# Define the output directory\
output_dir="./scan_results"

# Loop through the target domains\
for domain in "${domains[@]}"\
do\
    # Create a directory for the current domain's results\
    mkdir -p "$output_dir/$domain"

    # Scan for open ports\
    nmap -v -oN "$output_dir/$domain/open_ports.txt" "$domain"

    # Scan for subdomains\
    sublist3r -d "$domain" -o "$output_dir/$domain/subdomains.txt"

    # Scan for file paths\
    gobuster dir -u "https://$domain" -w /usr/share/wordlists/dirb/common.txt -o "$output_dir/$domain/file_paths.txt" -k\
done

# Compare the new scan results with the previous ones and send an email if there are any differences\
changes=$(git --git-dir="$output_dir/.git" --work-tree="$output_dir" diff HEAD~1 HEAD)

if [[ -n $changes ]]\
then\
    echo "$changes" | mail -s "Scan results have changed" youremail@example.com\
fi
```

This is the tool we built in Chapter 5, and it took ChatGPT a total of 3 minutes to replicate the functionality. I imagine it would be easy to build out more complicated tools as well.

It was pretty amazing to see ChatGPT at work. It made some incorrect assumptions and did not build a usable tool right away. But if you understand how to program, this will make it very easy to add new functionality to your tools or get a jump start on anything you are building.

Understanding how to write accurate and efficient ChatGPT prompts is probably the next big security skill.

Want to connect? What other security concepts do you want to learn about? I'd love to know. Feel free to connect on Twitter [@vickieli7](https://twitter.com/vickieli7).