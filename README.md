# TCP-UDP-Flood

`flood.py` is a small Python 3 command-line script that generates repeated TCP or UDP traffic toward a specified host and port. It is intended only for controlled, authorized network testing in an isolated lab or against infrastructure you own and are permitted to test.

Do not run this tool against public systems, third-party networks, production services, or any target where you do not have explicit written permission. Unapproved traffic flooding can disrupt services and may be illegal.

## Project Contents

```text
.
├── flood.py    # Python traffic generation script
└── README.md   # Project documentation
```

## Requirements

- Python 3
- Network access to the test host
- Permission to send test traffic to the selected host and port

The script uses only Python standard library modules:

- `argparse`
- `random`
- `socket`
- `threading`

No third-party packages are required.

## Basic Usage

Run the script with a target IP address and port:

```bash
python3 flood.py --ip <authorized-test-host> --port <port>
```

By default, the script uses UDP mode, sends `50000` packets per loop per thread, and starts `5` worker threads.

## Command-Line Options

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `-i`, `--ip` | Yes | None | Target host IP address for an authorized test system. |
| `-p`, `--port` | Yes | None | Target port number. |
| `-c`, `--choice` | No | `y` | Selects protocol mode. `y` runs UDP mode; any other value runs TCP mode. |
| `-t`, `--times` | No | `50000` | Number of send attempts per loop for each worker thread. |
| `-th`, `--threads` | No | `5` | Number of worker threads to start. |

## Examples

UDP mode against an authorized lab host:

```bash
python3 flood.py --ip 192.0.2.10 --port 9999 --choice y
```

TCP mode against an authorized lab host:

```bash
python3 flood.py --ip 192.0.2.10 --port 8080 --choice n
```

Lower-volume local test configuration:

```bash
python3 flood.py --ip 127.0.0.1 --port 9000 --choice y --times 100 --threads 1
```

## How It Works

The script parses command-line arguments, prints a short banner, then starts the requested number of worker threads.

In UDP mode, each worker thread:

1. Creates a UDP socket.
2. Builds a 1024-byte random payload.
3. Sends that payload repeatedly to the configured host and port.
4. Prints a status message after each send loop.

In TCP mode, each worker thread:

1. Creates a TCP socket.
2. Connects to the configured host and port.
3. Builds a 16-byte random payload.
4. Sends that payload repeatedly over the connection.
5. Prints a status message after each send loop.

Both modes run continuously until the process is stopped.

## Stopping the Script

Use `Ctrl+C` in the terminal where the script is running.

If the process does not stop cleanly, find and terminate it from another shell:

```bash
ps aux | grep flood.py
kill <pid>
```

## Safety Guidance

- Use this only in a private lab or another explicitly authorized environment.
- Start with low values for `--times` and `--threads`.
- Monitor CPU, memory, network usage, and service logs during tests.
- Avoid running this from shared networks, corporate networks, public Wi-Fi, or cloud environments unless you have approval.
- Coordinate tests with anyone responsible for the target system or surrounding network.
- Keep written authorization and a defined test window for any non-local test.

## Troubleshooting

### The script shows Python usage text

The required `--ip` and `--port` arguments were probably omitted. Provide both values:

```bash
python3 flood.py --ip <authorized-test-host> --port <port>
```

### TCP mode prints errors

This usually means the target port is closed, the host is unreachable, a firewall is blocking the connection, or the service is refusing new connections.

### UDP mode appears to run but the target shows no traffic

UDP does not require a connection, so the sender may continue printing status messages even if the destination is not listening. Check local routing, host firewalls, and packet captures on the authorized test system.

### Python reports a syntax error

Check that the import statements at the top of `flood.py` are valid Python statements and are separated correctly.

## Limitations

- There is no built-in rate limiter.
- There is no built-in duration limit.
- The script does not validate whether the target is authorized.
- The TCP function opens sockets inside a continuous loop and has limited cleanup behavior.
- The broad exception handling hides specific error causes.
- Output is minimal and does not include packet counts, throughput, latency, or timestamps.

## Responsible Use

This repository is for educational and authorized testing scenarios only. If your goal is production-grade load testing or resilience validation, use a purpose-built load testing tool with authentication, rate controls, reporting, and safety mechanisms.
