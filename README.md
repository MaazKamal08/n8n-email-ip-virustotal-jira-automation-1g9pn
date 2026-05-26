# n8n-email-ip-virustotal-jira-automation-1g9pn

## Overview

This n8n workflow monitors a specific Gmail inbox for security alerts related to inbound connections. It extracts origin and impacted IP addresses from the email body, then submits these IPs to VirusTotal for analysis. Based on predefined criteria for maliciousness or suspicion, the workflow automatically generates a Jira ticket for the security team, including a detailed table of VirusTotal results, and sends a notification with the Jira link to a Google Chat space.

## Features

- Monitors specific Gmail inbox for security alerts (sender and subject filter).
- Extracts pairs of Origin and Impacted IPv4 addresses from email text.
- De-duplicates extracted IP pairs to avoid redundant scans.
- Scans extracted IP addresses using the VirusTotal API (with rate limiting).
- Parses comprehensive VirusTotal scan results, including maliciousness, reputation, country, and ASN.
- Applies conditional logic to identify critical threats based on multiple VirusTotal metrics (malicious count, suspicious count, harmless count, country).
- Automates the creation of Jira issues for identified malicious/suspicious IPs.
- Generates a formatted Jira markdown table summarizing VirusTotal results for each IP.
- Sends Google Chat notifications with a direct link to the newly created Jira ticket.

## Services Used

- Gmail
- VirusTotal
- Jira Software Cloud
- Google Chat
- n8n (Code, If, Set, Merge, HTTP Request nodes)

## Trigger

The workflow is triggered every minute by the 'Gmail Trigger' node, which checks for new emails from 'titanium-soc@rapidcompute.com' with a subject containing 'threat list: inbound connection detected'.

## Prerequisites

- Active n8n instance.
- Gmail account with read access to the specified security alert inbox.
- VirusTotal API Key (consider premium for higher rate limits).
- Jira Software Cloud instance with a designated project for security tickets.
- Google Chat account and a specific space for notifications.

## Credentials

- Gmail OAuth2 (named 'Gmail account')
- VirusTotal API (named 'VirusTotal account')
- Jira Software Cloud API (named 'Jira SW Cloud account')
- Google Chat OAuth2 API (named 'Chat account')

## Configuration

1. **Gmail Trigger**: Configure with your Gmail OAuth2 credentials. Verify the 'sender' filter (`titanium-soc@rapidcompute.com`) and ensure 'Code in JavaScript3' subject filter (`threat list: inbound connection detected`) matches your alert emails.
2. **Code in JavaScript**: Review the IP extraction regex if your email format differs. The current code assumes IP pairs (Origin, Impacted) are present sequentially.
3. **Code in JavaScript1**: Ensure the output 'ip_address' field correctly maps to the 'Origin' IP for the VirusTotal query.
4. **VirusTotal IP Check2**: Configure with your VirusTotal API key. Adjust 'batchInterval' (currently 15000ms/15s) based on your VirusTotal API rate limits and desired processing speed.
5. **If**: Review and customize the conditions for creating a Jira ticket. Current logic: (Malicious >= 2 OR Harmless >= 10 OR Suspicious >= 5 OR VT_Country != 'PK').
6. **Create an issue**: Configure with your Jira Software Cloud credentials. Update 'Project ID' (currently 10026) and 'Issue Type' (currently 10002 - Task) to match your Jira setup.
7. **Send message and wait for response**: Configure with your Google Chat OAuth2 credentials. Update 'Space ID' (currently `spaces/2QD3BcAAAAE`) to your desired Google Chat space for notifications.

## Usage

1. Activate the n8n workflow.
2. Ensure your configured Gmail account receives security alert emails from the specified sender with the expected subject line.
3. The workflow will automatically trigger every minute, process new emails, scan IPs via VirusTotal, and create Jira tickets for identified threats.
4. Monitor your Jira project for new issues and your Google Chat space for new threat notifications.

## Troubleshooting

- **No emails processed**: Verify Gmail Trigger credentials, sender filter, and the subject line filter in 'Code in JavaScript3'. Check workflow activation status.
- **IPs not extracted correctly**: Examine the 'Code in JavaScript' node; the regex and pairing logic might need adjustment if email format changes.
- **VirusTotal errors/slowdowns**: Check VirusTotal API credentials and monitor your API usage/rate limits. Adjust the 'batchInterval' in 'VirusTotal IP Check2' if hitting limits.
- **No Jira tickets**: Review the 'If' node conditions; they might be too restrictive. Check Jira credentials and ensure 'Project ID'/'Issue Type' are correct and accessible.
- **Incorrect data in Jira/Chat**: Debug 'Parse VirusTotal Results1' for VT data parsing and 'Code in JavaScript2' for Jira table formatting.

## Security Notes

- **API Key Security**: Ensure n8n is hosted securely and all API keys (VirusTotal, Jira, Google Chat) are managed as n8n credentials, not hardcoded.
- **False Positives/Negatives**: Carefully tune the conditions in the 'If' node to balance between blocking legitimate traffic (false positive) and missing actual threats (false negative).
- **VirusTotal Data Disclosure**: IP addresses sent to VirusTotal become publicly visible as part of their dataset. Be aware of any data privacy implications.
- **Rate Limit Management**: VirusTotal API has strict rate limits. The current batching in 'VirusTotal IP Check2' is conservative; adjust as per your specific VirusTotal plan to avoid blacklisting.
