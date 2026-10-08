# C# Port Scanner

A small, beginner-friendly TCP port scanner written in C# (.NET 8).
It tries to open a TCP connection to each port. If the connection succeeds before the timeout, the port is **open**.

> ⚠️ **Only scan computers you own or have permission to test.**
> Scanning other people's systems without permission can be illegal.

## Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download) (or newer)

Check it is installed:

```bash
dotnet --version
```

## Create and run

```bash
dotnet new console -n CSharpPortScanner
cd CSharpPortScanner
# replace Program.cs with the one from this repository
dotnet build
dotnet run -- 127.0.0.1 1-1024
```

Arguments:

| Argument | Example | Meaning |
|---|---|---|
| host | `127.0.0.1`, `localhost` | Computer to scan |
| port range | `1-1024`, `80` | First and last port (1–65535) |

If you run `dotnet run` with no arguments, the program asks for the target and the range.
Press **Enter** to use the defaults (`127.0.0.1` and `1-1024`).

## Example output

Real output from a test machine running web servers on ports 80 and 8080:

```text
C# Port Scanner
Target: localhost (127.0.0.1)
Port range: 1-10000

Scanning...

[OPEN] 80
[OPEN] 2024
[OPEN] 2025
[OPEN] 8080

Scan complete.
Open ports: 4 (80, 2024, 2025, 8080)
Time: 0.8 s
```

Your results will be different. They depend on what is running on your computer.

## Want something to find?

Start a small test web server in another terminal, then scan it:

```bash
python3 -m http.server 8080      # or: python -m http.server 8080 on Windows
dotnet run -- 127.0.0.1 8000-8100
```

## How it works

1. **Read input** – host and port range (from arguments or by asking).
2. **Find the IP address** – `Dns.GetHostAddressesAsync` turns a name like `localhost` into an IP.
3. **Try each port** – `TcpClient.ConnectAsync` with a 500 ms timeout (`CancellationTokenSource`).
   - Connected → **open**
   - Refused (`SocketException`) or no answer in time → not open
4. **Run in parallel** – up to 100 ports at the same time (`SemaphoreSlim`), so the scan is fast but polite.
5. **Report** – prints each open port, then a summary.

Settings at the top of `Program.cs`:

```csharp
const int TimeoutMs = 500;     // how long we wait for each port
const int MaxParallel = 100;   // how many ports we check at the same time
```

## Errors it handles

```text
Invalid port range: "0-70000". Use a range like 1-1024 (ports 1 to 65535).
Could not find host: "no-such-host.invalid".
```

## What this project is not

This is a learning tool. It only checks whether a TCP port accepts a connection.
It has no stealth scanning, no service exploitation, and no password guessing.
