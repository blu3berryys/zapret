## tpws

`tpws` is a static binary that allows you to run in **SOCKS mode** using **WSL** (Windows Subsystem for Linux) on newer builds of Windows 10 and Windows Server. It's not necessary to install a Linux distribution as suggested in most articles. You only need to install `WSL`.

### Installation Steps

1. **Install WSL**:  
   Run the following command in **Administrator** mode to enable WSL:
   ```bash
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all
   ```

2. **Copy Binary**:  
   Copy the `binaries/x86_64/tpws_wsl.tgz` file to the target system.

3. **Import tpws into WSL**:  
   Use the following command to import `tpws`:
   ```bash
   wsl --import tpws "%USERPROFILE%\tpws" tpws_wsl.tgz
   ```

4. **Run tpws**:  
   Execute `tpws` in WSL with the following command:
   ```bash
   wsl -d tpws --exec /tpws --uid=1 --no-resolve --socks --bind-addr=127.0.0.1 --port=1080 <fooling_options>
   ```

5. **Configure SOCKS Proxy**:  
   Set your SOCKS proxy in your browser or any other program to `127.0.0.1:1080`.

6. **Cleanup**:  
   After you're done, clean up by unregistering `tpws`:
   ```bash
   wsl --unregister tpws
   ```

**Tested on**: Windows 10 Build 19041 (20.04)

### Known Issues

- `--oob`, `--mss`, and `--disorder` do not work.
- RST detection in the autohostlist scheme may not work.
- WSL may glitch with `splice`. You may need to use `--nosplice`.

---

## winws

`winws` is the **Windows version** of `nfqws`, based on **windivert**. Most features are fully functional, but **large IP filters (ipsets)** and **connection sharing** are not supported. **Administrator rights** are required.

### Overview of Packet Filter Operation

Windows lacks a built-in packet selection tool like Linux's `iptables`, `nftables`, `pf`, or `ipfw`. Instead, it uses the **Windivert** library, a packet redirector that works starting from Windows 7.

`winws` automates filter construction using simple filters based on **IP version** and **port**. Raw filters are also supported.

### Common Parameters for `winws`

- `--wf-iface=<int>[:<int>]`: Network interface and subinterface indexes (numeric).
- `--wf-l3=ipv4|ipv6`: Layer 3 protocol filter. Multiple comma-separated values allowed.
- `--wf-tcp=[~]port1[-port2]`: TCP port filter. Use `~` for negation.
- `--wf-udp=[~]port1[-port2]`: UDP port filter. Use `~` for negation.
- `--wf-raw=<filter>|@<filename>`: Raw Windivert filter string or filename.
- `--wf-save=<filename>`: Save Windivert filter string to a file and exit.
- `--ssid-filter=ssid1[,ssid2,ssid3,...]`: Enable `winws` if any of the specified WiFi SSIDs are connected.
- `--nlm-filter=net1[,net2,net3,...]`: Enable `winws` if any of the specified NLM networks are connected.
- `--nlm-list[=all]`: List NLM networks (either all or connected).

### Discovering Interface Indexes

Use this command to find the interface index:
```bash
netsh int ip show int
```
Alternatively, run `winws --debug` to find the interface index there. The subinterface index is typically `0`.

### Multiple Instances

Multiple `winws` instances can run, but **filter overlap is discouraged**. You can enable `winws` only when specific networks are connected by using the `--ssid-filter` or `--nlm-filter` options.

### Troubleshooting

- **Cygwin Issues**: `Cygwin` shell will not run binaries if its directory contains `cygwin1.dll`. If you encounter this, delete or rename `cygwin1.dll` in the `cygwin` directory.
- **Windows 7 Support**: `winws` is compatible with Windows 7 if `windivert` is properly signed. Follow the instructions below for `Windows 7 Windivert Signing`.

---

## Windows 7 Windivert Signing

To use `windivert` on **Windows 7**, the driver must be signed. Since official updates stopped in 2020, you need to use one of the following methods:

1. **Using `windivert64.sys` and `windivert.dll` version `2.2.0-C` or `2.2.0-D`**:
   - Download from [here](https://reqrypt.org/download).
   - Replace these files in the appropriate directories (e.g., `zapret-win-bundle/zapret-winws`).
   - **Note**: This requires a 10+ year old patch for SHA256 signatures.

2. **Hack ESU**:  
   Instructions can be found [here](https://hackandpwn.com/windows-7-esu-patching).

3. **Update Pack for Windows 7**:  
   Use `UpdatePack7R2` from [Simplix](https://blog.simplix.info).

---

## blockcheck

`blockcheck.sh` is written in POSIX shell and requires certain utilities that are not available in Windows by default. To run it, you must use **Cygwin**.

### How to Run `blockcheck.sh`:

1. **Install Cygwin**:
   - Download the Cygwin installer from [here](https://www.cygwin.com/setup-x86_64.exe).
   - Install **curl**, `gcc-core`, `make`, and `zlib-devel` for compiling from source.

2. **Run Scripts**:
   - First, run `install_bin.sh`, then run `blockcheck.sh`.
   - Use Cygwin paths for Windows directories (e.g., `C:/Users/vasya`).

3. **Run as Administrator**:
   - `Cygwin` needs to be run as administrator to execute `blockcheck.sh`.

**Note**: If you're using `cygwin1.dll` in the Cygwin directory, it might prevent `winws` from running. Rename or delete the conflicting `cygwin1.dll`.

---

## Auto Start

To automatically start `winws` with Windows, you can use the **Windows Task Scheduler**. The following batch files in `binaries/win64/zapret-winws` help you manage scheduled tasks:

- `task_create.cmd`: Creates the scheduled task `winws1`.
- `task_remove.cmd`: Removes the scheduled task `winws1`.
- `task_start.cmd`: Starts the scheduled task `winws1`.
- `task_stop.cmd`: Stops the scheduled task `winws1`.

These files must be run with administrator privileges. You can also use **Windows Services** with the `service_*.cmd` files.

---

## zapret-win-bundle

The **zapret-win-bundle** is a pre-configured bundle that includes `cygwin`, `blockcheck`, and `winws`. This makes setup and usage easier.

### Bundle Contents:

- `/zapret-winws`: Standalone version of `winws` for everyday use.
- `/zapret-winws/_CMD_ADMIN.cmd`: Opens a command prompt as administrator.
- `/blockcheck/blockcheck.cmd`: Runs `blockcheck` with logging.
- `/cygwin/cygwin.cmd`: Runs Cygwin shell as the current user.
- `/cygwin/cygwin-admin.cmd`: Runs Cygwin shell as administrator.

The bundle also includes aliases for commonly used commands like `winws`, `blockcheck`, `ip2net`, and `mdig`.

### Debugging `winws`

To enable `winws` debug logging, use:
```bash
winws --debug --wf-tcp=80,443 | tee winws.log
unix2dos winws.log
```
This will create a log file `winws.log` in your Cygwin home directory. Use `unix2dos` to convert it for Windows 7 compatibility (not needed on Windows 10+).

---

## Conclusion

This bundle and the `winws` tool can greatly enhance your experience when using packet filtering on Windows, especially if you require advanced bypass capabilities. For a smoother experience, always refer to the provided configuration scripts and utilities.
