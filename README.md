<div align="center">

![WIP](https://img.shields.io/badge/work%20in%20progress-yellow?style=for-the-badge)
![System & Kernel](https://img.shields.io/badge/System-brown?style=for-the-badge)
![Job Control](https://img.shields.io/badge/Job-Control-blue?style=for-the-badge)
![Daemon Proceds](https://img.shields.io/badge/Daemon-Process-green?style=for-the-badge)
![C++ Language](https://img.shields.io/badge/Language-C++-red?style=for-the-badge)

*A job-control daemon with supervisor-like features*

</div>

<div align="center">
  <img src="/W_Taskmaster.png">
</div>

# Taskmaster

[README en Español](README_es.md)

`Taskmaster` is a `42 School` project that implements a complete job-control daemon with supervisor-like functionality. This system allows you to manage background processes, keep them alive, restart them automatically when needed, and provide full control over their lifecycle. The project includes both the main daemon and a control client with an advanced interactive shell.

## ✨ Features

### Main Daemon (taskmasterd)
- `Real Daemon`: Process that runs independently in the background
- `Process Control`: Full supervision of process lifecycles
- `Dynamic Configuration`: Reload configuration without stopping the daemon (SIGHUP)
- `Logging System`: Advanced management with automatic rotation and syslog support
- `Privilege De-escalation`: Ability to run as a specific user
- `Multiple Protocols`: Connection via UNIX sockets and INET sockets
- `Advanced Variables`: Support for variable expansion in configuration
- `Resource Management`: Control of file descriptors and minimum processes

### Control Client (taskmasterctl)
- `Advanced Interactive Shell`: Custom readline with `vi` mode support
- `Full History`: History search and expansion
- `Autocomplete`: Intelligent completion system
- `Attach/Detach`: Connect and disconnect to supervised processes in real time
- `Remote Control`: Manage the daemon from a remote client

### Configuration Parser
- `Supervisor Syntax`: Compatible with the standard supervisor format
- `Custom Parser`: In-house implementation for maximum control
- `Dynamic Variables`: Full support for system and custom variables
- `Variable Modifiers`: Advanced syntax for string and number manipulation

## 🔧 Installation

```bash
git clone https://github.com/Kobayashi82/taskmaster.git
cd taskmaster
make

# Binaries generated in the bin/ folder:
# taskmasterd     - Main daemon
# taskmasterctl   - Interactive control client
```

## 🖥️ Daemon Usage (taskmasterd)

### Available Options

```bash
Usage: taskmasterd: [ OPTION... ]
```

#### Configuration and Execution:
- `-c, --configuration=FILENAME`: Configuration file
- `-n, --nodaemon`: Run in foreground (not as daemon)
- `-s, --silent`: Do not show logs on stdout (only when not running as daemon)
- `-d, --directory=DIRECTORY`: Daemon working directory
- `-k, --nocleanup`: Prevent automatic cleanup on start

#### User and Permissions Control:
- `-u, --user=USER`: Run as a specific user (or numeric UID)
- `-m, --umask=UMASK`: Umask for child processes (default: 022)

#### Logging System:
- `-l, --logfile=FILENAME`: Main log file
- `-y, --logfile_maxbytes=BYTES`: Maximum log file size
- `-z, --logfile_backups=NUM`: Number of log rotation backups
- `-e, --loglevel=LEVEL`: Log level (debug,info,warn,error,critical)
- `-q, --childlogdir=DIRECTORY`: Directory for child process logs
- `-t, --strip_ansi`: Strip ANSI codes from process output

#### Instance Control:
- `-j, --pidfile=FILENAME`: Daemon PID file
- `-i, --identifier=STR`: Unique identifier for this instance

#### System Requirements:
- `-a, --minfds=NUM`: Minimum number of file descriptors to start
- `-p, --minprocs=NUM`: Minimum number of processes available to start

#### Information:
- `-h, --help`: Show full help
- `-v, --version`: Show program version

### Basic Usage

```bash
# Run with basic configuration
sudo ./taskmasterd -c /etc/taskmaster/taskmaster.conf

# Run in foreground (development/debug)
./taskmasterd -c config.conf -n -e debug

# Run as specific user
sudo ./taskmasterd -c config.conf -u user

# Full logging configuration
./taskmasterd -c config.conf -l /var/log/taskmaster/daemon.log -y 10MB -z 5 -e info -q /var/log/taskmaster/children/
```

### Daemon Management

```bash
# Check daemon status
ps aux | grep taskmasterd
cat /var/run/taskmasterd.pid

# Reload configuration (without stopping unchanged processes)
sudo kill -HUP $(cat /var/run/taskmasterd.pid)

# Stop daemon gracefully
sudo kill -TERM $(cat /var/run/taskmasterd.pid)

# View logs in real time
tail -f /var/log/taskmaster/daemon.log
```

## 🖥️ Client Usage (taskmasterctl)

### Interactive Shell Features

- `Custom Readline`: In-house implementation
- `Full History`: Navigation, search, and history expansion
- `Vi Mode`: Line editing with `vi` commands
- `Autocomplete`: Intelligent completion for commands and program names
- `Colored Syntax`: Visual command highlighting

### Available Commands

```bash
# Start the interactive client
./taskmasterctl

# Commands inside the shell:
taskmaster> status                    # Show status of all programs
taskmaster> status nginx              # Show status of a specific program
taskmaster> start nginx               # Start program
taskmaster> stop nginx                # Stop program
taskmaster> restart nginx             # Restart program
taskmaster> reload                    # Reload daemon configuration
taskmaster> shutdown                  # Stop the daemon completely
taskmaster> attach nginx:0            # Attach to a specific process
taskmaster> detach                    # Detach from current process
taskmaster> tail nginx stderr         # View process logs
taskmaster> clear nginx               # Clear process logs
taskmaster> help                      # Show help
taskmaster> quit                      # Exit the client
```

### Advanced Functions

```bash
# Attach to a specific process (interactive mode)
taskmaster> attach webapp:2
# You are now connected directly to the process
# Ctrl+C to detach and return to the taskmaster shell

# View logs in real time
taskmaster> tail -f nginx stdout
taskmaster> tail -100 webapp stderr

# Granular management
taskmaster> start webapp:*            # Start all webapp processes
taskmaster> stop webapp:0 webapp:1    # Stop specific processes
taskmaster> restart all               # Restart all programs
```

## ⚙️ Configuration File

The configuration file uses supervisor-like syntax.

### Configuration Parameters

#### Taskmasterd Configuration

| Parameter                 | Description                    | Values                           | Default                  |
|---------------------------|--------------------------------|----------------------------------|--------------------------|
| `nodaemon`                | Run in foreground              | true/false                       | false                    |
| `silent`                  | Do not show logs on stdout     | true/false                       | false                    |
| `user`                    | User to run daemon             | string or UID                    | (current user)           |
| `umask`                   | Permission umask               | octal                            | 022                      |
| `directory`               | Working directory              | path                             | (current directory)      |
| `logfile`                 | Daemon log file                | path/AUTO/NONE                   | AUTO                     |
| `logfile_maxbytes`        | Max log size                   | bytes (KB/MB)                    | 50MB                     |
| `logfile_backups`         | Number of log backups          | int                              | 10                       |
| `loglevel`                | Logging level                  | debug/info/warn/error/critical   | info                     |
| `pidfile`                 | Daemon PID file                | path/AUTO                        | AUTO                     |
| `identifier`              | Instance identifier            | string                           | taskmaster               |
| `childlogdir`             | Child process logs directory   | path/AUTO                        | AUTO                     |
| `strip_ansi`              | Strip ANSI codes               | true/false                       | false                    |
| `nocleanup`               | Do not clean up on start       | true/false                       | false                    |
| `minfds`                  | Minimum file descriptors       | int                              | 1024                     |
| `minprocs`                | Minimum available processes    | int                              | 200                      |
| `environment`             | Global environment variables   | KEY=Value                        | (empty)                  |

#### Program Configuration

| Parameter                 | Description                    | Values                           | Default                  |
|---------------------------|--------------------------------|----------------------------------|--------------------------|
| `command`                 | Command to execute             | string                           | (required)               |
| `numprocs`                | Number of processes to keep    | int                              | 1                        |
| `process_name`            | Process name pattern           | string                           | $PROGRAM_NAME            |
| `directory`               | Working directory              | path                             | /                        |
| `umask`                   | Permission umask               | octal                            | 022                      |
| `user`                    | User to run                    | string                           | (current user)           |
| `autostart`               | Start automatically            | bool                             | true                     |
| `autorestart`             | Restart policy                 | true/false/unexpected            | unexpected               |
| `exitcodes`               | Expected exit codes            | list                             | 0                        |
| `startretries`            | Start retries                  | int                              | 3                        |
| `starttime`               | Minimum run time               | seconds                          | 1                        |
| `stopsignal`              | Signal to stop gracefully      | TERM/HUP/INT/QUIT/KILL/USR1/USR2 | TERM                     |
| `stoptime`                | Time before SIGKILL            | seconds                          | 10                       |
| `priority`                | Start/stop priority            | int                              | 999                      |
| `stdout_logfile`          | stdout log file                | path/AUTO/NONE                   | AUTO                     |
| `stderr_logfile`          | stderr log file                | path/AUTO/NONE                   | AUTO                     |
| `stdout_logfile_maxbytes` | Max stdout size                | bytes  (KB/MB)                   | 50MB                     |
| `stderr_logfile_maxbytes` | Max stderr size                | bytes  (KB/MB)                   | 50MB                     |
| `stdout_logfile_backups`  | stdout backups                 | int                              | 10                       |
| `stderr_logfile_backups`  | stderr backups                 | int                              | 10                       |
| `environment`             | Global environment variables   | KEY=Value                        | (empty)                  |

#### Group Configuration

| Parameter                 | Description                    | Values                           | Default                  |
|---------------------------|--------------------------------|----------------------------------|--------------------------|
| `programs`                | Group program list             | list                             | (required)               |
| `priority`                | Group start priority           | int                              | 999                      |

#### UNIX Server Configuration

| Parameter                 | Description                    | Values                           | Default                  |
|---------------------------|--------------------------------|----------------------------------|--------------------------|
| `file`                    | UNIX socket path               | path                             | /var/run/taskmaster.sock |
| `chmod`                   | Socket permissions             | octal                            | 0700                     |
| `chown`                   | Socket owner                   | user:group                       | root:root                |

#### INET Server Configuration

| Parameter                 | Description                    | Values                           | Default                  |
|---------------------------|--------------------------------|----------------------------------|--------------------------|
| `port`                    | Host/IP and listening port     | host:port or *:port              | *:9001                   |
| `username`                | Authentication username        | string                           | (no auth)                |
| `password`                | Authentication password        | string                           | (no auth)                |

### Configuration File Example

```yaml
# Global server configuration
[taskmasterd]
nodaemon=false
silent=false
user=1000
umask=077
directory=.
logfile=logfile.log
logfile_maxbytes=50MB
logfile_backups=1000
logfile_syslog=false
loglevel=debug
pidfile=pidfile.pid
identifier=taskmaster
childlogdir=.
strip_ansi=true
nocleanup=false
minfds=512
minprocs=100
environment=VAR1=VALUE1, VAR2="VALUE2 with variable \$USER = $USER"

# Program definitions
[program:date]
command=date
process_name=${PROCESS_NAME}_${PROCESS_NUM:*02d}
numprocs=3
directory=$HERE
tty_mode=false
umask=077
priority=500
autostart=true
autorestart=unexpected
startsecs=5
startretries=5
exitcodes=0
stopsignal=TERM
stopwaitsecs=10
stopasgroup=true
killasgroup=true
user=root
redirect_stderr=false
stdout_logfile=$HOME/${PROCESS_NAME}_${PROCESS_NUM:*02d}_stdout.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_logfile_syslog=true
stderr_logfile=~/${PROCESS_NAME}_${PROCESS_NUM:*02d}_stderr.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_logfile_syslog=false
environment=VAR1='VALUE1 with spaces', VAR2=VALUE2\ with \ spaces
serverurl=unix://~/taskmaster.sock

[program:nginx]
command=/usr/local/bin/nginx -c /etc/nginx/test.conf
numprocs=1
directory=/tmp
umask=022
autostart=true
autorestart=unexpected
exitcodes=0,2
startretries=3
starttime=5
stopsignal=TERM
stoptime=10
stdout_logfile=/var/log/nginx/stdout.log
stderr_logfile=/var/log/nginx/stderr.log
environment=STARTED_BY="taskmaster",ANSWER="42",PATH="/usr/bin:${PATH}"
user=nginx
priority=999

# Group definitions
[group:mygroup]
programs=date, nginx
priority=999

# Connectivity configuration
[unix_http_server]
file=/var/run/taskmaster.sock
chmod=0700
chown=taskmaster:taskmaster

[inet_http_server]
port=127.0.0.1:9001
username=admin
password=secret
```

## 🔤 Variables

Taskmaster's variable system enables dynamic and flexible configuration through advanced variable expansion syntax. It supports default values, string manipulation, numeric formatting, and configuration variables.

#### Default Values

| Syntax                    | Description                                                       | Example             |
|---------------------------|-------------------------------------------------------------------|---------------------|
| `${VAR:-default}`         | If `VAR` is unset or empty, returns `default`                     | ${PORT:-8080}       |
| `${VAR:+default}`         | If `VAR` is set and not empty, returns `default`                  | ${DEBUG:+--verbose} |

#### String Manipulation

| Syntax                    | Description                                                       | Example             |
|---------------------------|-------------------------------------------------------------------|---------------------|
| `${VAR:offset}`           | From position `offset` to the end                                 | ${HOST:2}           |
| `${VAR:offset:len}`       | From position `offset`, take `len` characters                     | ${HOST:0:3}         |
| `${VAR: -offset}`         | From `offset` characters from the end to the end                  | ${HOST: -2}         |
| `${VAR: -offset:len}`     | From `offset` characters from the end, take `len` characters      | ${HOST: -2:3}       |
| `${VAR:^}`                | Uppercase first letter                                            | ${USER:^}           |
| `${VAR:^^}`               | Uppercase all                                                     | ${USER:^^}          |
| `${VAR:,}`                | Lowercase first letter                                            | ${USER:,}           |
| `${VAR:,,}`               | Lowercase all                                                     | ${USER:,,}          |
| `${VAR:~}`                | Remove from start to last / (file name only)                      | ${USER:~~}          |

#### Numeric Formatting

| Syntax                    | Description                                                       | Example             |
|---------------------------|-------------------------------------------------------------------|---------------------|
| `${#VAR}`                 | Variable length                                                   | ${#PROCESS_NUM}     |
| `${VAR:d}`                | Signed decimal integer                                            | ${PROCESS_NUM:*d}   |
| `${VAR:02d}`              | Integer with leading zeros                                        | ${PROCESS_NUM:*02d} |
| `${VAR:x}`                | Lowercase hexadecimal                                             | ${PORT:*x}          |
| `${VAR:X}`                | Uppercase hexadecimal                                             | ${PORT:*X}          |
| `${VAR:#x}`               | Lowercase hexadecimal (prefixed with 0x)                          | ${PORT:*#x}         |
| `${VAR:#X}`               | Uppercase hexadecimal (prefixed with 0X)                          | ${PORT:*#X}         |
| `${VAR:o}`                | Octal                                                             | ${UMASK:*o}         |    

#### Taskmaster Variables

| Syntax                    | Description                                                                             |
|---------------------------|-----------------------------------------------------------------------------------------|
| `TASKMASTER_ENABLED`      | Flag indicating the process is under Taskmaster control                                 |
| `TASKMASTER_PROCESS_NAME` | Process name specified in the configuration file                                        |
| `TASKMASTER_GROUP_NAME`   | Group name the process belongs to                                                       |
| `TASKMASTER_SERVER_URL`   | Internal server URL                                                                     |

#### Configuration Variables

| Variable                 | Description                                                                              |
|--------------------------|------------------------------------------------------------------------------------------|
| `HERE`                   | Configuration file directory                                                             |
| `HOST_NAME`              | System host name                                                                         |
| `GROUP_NAME`             | Current group name                                                                       |
| `PROGRAM_NAME`           | Program name                                                                             |
| `NUMPROCS`               | Total number of processes                                                                |
| `PROCESS_NUM`            | Process number (0, 1, 2...)                                                              |

## 🧪 Testing

### Basic Daemon Tests

```bash
# Basic startup test
./taskmasterd -c test.conf -n -e debug

# Invalid configuration test
./taskmasterd -c invalid.conf -n

# Privilege de-escalation test
sudo ./taskmasterd -c test.conf -u nobody -n

# Lock file test
./taskmasterd -c test.conf
./taskmasterd -c test.conf  # Should fail
```

### Process Control Tests

```bash
# Create test config with a simple program
cat > test.conf << EOF
[program:test]
command=/bin/sleep 30
numprocs=2
autostart=true
autorestart=true
EOF

# Run daemon and client
./taskmasterd -c test.conf -n
./taskmasterctl

# In the client:
taskmaster> status
taskmaster> stop test:0
taskmaster> start test:0
taskmaster> restart test
```

### Variable Tests

```bash
# Configuration with complex variables
cat > vars.conf << EOF
[program:webapp]
command=/usr/bin/webapp --port=800${PROCESS_NUM:*02d} --name=${PROGRAM_NAME}_${PROCESS_NUM}
environment=LOG_LEVEL="${LOG_LEVEL:-info}", WORKER_NAME="${USER:*upper}_${PROCESS_NUM:*02d}", CONFIG_PATH="${HERE}/config"
numprocs=3
EOF

# Variable expansion test
LOG_LEVEL=debug ./taskmasterd -c vars.conf -n
```

### Logging and Rotation Tests

```bash
# Log rotation test
./taskmasterd -c test.conf -l /tmp/daemon.log -y 1KB -z 3 -n

# Generate logs until rotation
./taskmasterctl << EOF
status
tail test:0 stdout
tail test:1 stderr
EOF
```

### Attach/Detach Tests

```bash
# Configuration with an interactive program
cat > interactive.conf << EOF
[program:shell]
command=/bin/bash
numprocs=1
autostart=true
stdout_logfile=NONE
stderr_logfile=NONE
EOF

# Attach test
./taskmasterd -c interactive.conf -n
./taskmasterctl

# In the client:
taskmaster> attach shell:0
# You are now in an interactive bash
# Ctrl+C to detach
```

### Remote Connection Tests

```bash
# Configuration with inet server
cat > inet.conf << EOF
[inet_http_server]
port=127.0.0.1:9001

[program:test]
command=/bin/echo "Hello World"
EOF

# Connect from remote client
./taskmasterctl -s http://127.0.0.1:9001
```

## 📝 Log Examples

```bash
2024-08-28 10:30:15,123 INFO     taskmasterd started with pid 12345
2024-08-28 10:30:15,124 INFO     spawned: 'nginx' with pid 12346
2024-08-28 10:30:15,125 INFO     spawned: 'webapp_00' with pid 12347
2024-08-28 10:30:15,126 INFO     spawned: 'webapp_01' with pid 12348
2024-08-28 10:30:20,130 INFO     success: nginx entered RUNNING state, process has stayed up for > than 5 seconds (startsecs)
2024-08-28 10:30:20,131 INFO     success: webapp_00 entered RUNNING state, process has stayed up for > than 3 seconds (startsecs)
2024-08-28 10:30:25,135 WARN     webapp_01 process terminated unexpectedly (exit code 1)
2024-08-28 10:30:25,136 INFO     spawned: 'webapp_01' with pid 12350
2024-08-28 10:30:28,140 INFO     success: webapp_01 entered RUNNING state, process has stayed up for > than 3 seconds (startsecs)
2024-08-28 10:31:15,200 INFO     received SIGHUP indicating configuration reload
2024-08-28 10:31:15,201 INFO     configuration reloaded successfully
2024-08-28 10:31:15,202 INFO     stopped: nginx (exit status 0)
2024-08-28 10:31:15,203 INFO     spawned: 'nginx' with pid 12355
2024-08-28 10:31:20,210 INFO     success: nginx entered RUNNING state
```

## 🏗️ Technical Architecture

### Daemon Structure
-`Daemonizatio`: Double fork for full terminal independence
-`Instance Contro`: PID and lock files to prevent multiple instances
-`Signal Managemen`: Full handling of SIGHUP, SIGTERM, SIGINT, SIGCHLD
-`Privilege De-Escalatio`: Safe user switch after startup

### Process System
-`Supervisio`: Continuous monitoring of child process state
-`Smart Restar`: Configurable automatic restart policies
-`Timeout Contro`: Start and stop timing management
-`Resource Contro`: Verification of available descriptors and processes

### Communication System
-`UNIX Socket`: High-speed local communication
-`INET Socket`: Remote access with authentication
-`Custom Protoco`: Structured messages for client-daemon communication
-`Attach/Detac`: I/O multiplexing for direct process access

### Configuration Parser
-`Supervisor Synta`: Compatibility with existing configurations
-`Dynamic Variable`: Advanced expansion with modifiers
-`Validatio`: Full syntax and semantic validation
-`Hot Reloa`: Update without interrupting stable services

### Logging System
-`Automatic Rotatio`: Based on size and number of files
-`Multiple Level`: DEBUG, INFO, WARN, ERROR, CRITICAL
-`Syslog Integratio`: Optional output to `syslog`
-`Per-Process Log`: Separate `stdout`/`stderr` per process

## 📄 License

This project is licensed under the WTFPL – [Do What the Fuck You Want to Public License](http://www.wtfpl.net/about/).

---

<div align="center">

**📋 Developed as part of the 42 School curriculum 📋**

"*Because processes need a reliable babysitter"*

</div>
