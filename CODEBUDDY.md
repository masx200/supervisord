# CODEBUDDY.md This file provides guidance to CodeBuddy Code when working with code in this repository.

## Build and Development Commands

### Building the Project
For Linux:
```bash
go generate
GOOS=linux go build -tags release -a -ldflags "-linkmode external -extldflags -static" -o supervisord
```

For Windows:
```bash
go mod tidy
go build -tags release -o supervisord.exe
```

### Running Tests
Run tests with:
```bash
go test ./...
```

Run tests for specific packages:
```bash
go test ./process
go test ./config
go test ./logger
```

### Running the Application
```bash
# Run with config file
supervisord -c supervisor.conf

# Run as daemon with web UI
supervisord -c supervisor.conf -d

# Control commands (requires HTTP server enabled)
supervisord ctl status
supervisord ctl start program_name
supervisord ctl stop program_name
supervisord ctl shutdown
supervisord ctl reload
```

### Development Mode
For development with hot reload of web assets:
```bash
go build -tags dev
```

## Architecture Overview

This is a Go implementation of supervisord, a process control system. The architecture consists of several key components:

### Core Components

1. **Main Entry Point** (`main.go`)
   - Handles command line parsing and initialization
   - Sets up logging configuration
   - Manages signal handling

2. **Supervisor** (`supervisor.go`)
   - Central orchestration component
   - Manages the process manager, XMLRPC interface, and logger
   - Handles configuration loading and process lifecycle

3. **Process Manager** (`process/process_manager.go`)
   - Manages all processes defined in configuration
   - Handles starting, stopping, and monitoring processes
   - Supports both programs and event listeners

4. **Configuration** (`config/config.go`)
   - Parses and validates supervisord configuration files
   - Supports program definitions, groups, and various settings
   - Handles environment variable expansion

5. **XML-RPC Interface** (`xmlrpc.go`, `xmlrpcclient/`)
   - Provides API for process control
   - Enables web GUI and external tool integration

### Module Structure

The project is organized into Go modules with local replacements:
- `config/`: Configuration parsing and management
- `process/`: Process lifecycle management
- `logger/`: Logging system with multiple backends
- `events/`: Event system for process state changes
- `signals/`: Cross-platform signal handling
- `types/`: Shared type definitions
- `util/`: Common utilities
- `xmlrpcclient/`: XML-RPC client implementation
- `faults/`: Error handling
- `webgui/`: Web-based management interface
- `pidproxy/`: Process ID proxy functionality

### Platform-Specific Code

The codebase includes platform-specific implementations for:
- Windows: `*_windows.go` files
- Unix/Linux: `*_unix.go` files
- FreeBSD: `*_freebsd.go` files
- Darwin: `*_darwin.go` files

### Key Features

- Process management with auto-restart capabilities
- Web GUI for process monitoring and control
- XML-RPC API for external integration
- Prometheus metrics endpoint
- Event listener system
- Log rotation and management
- Group-based process organization
- Dependency management between processes

### Configuration File Locations

The system auto-detects configuration files in this order:
1. `$CWD/supervisord.conf`
2. `$CWD/etc/supervisord.conf`
3. `/etc/supervisord.conf`
4. `/etc/supervisor/supervisord.conf`
5. `../etc/supervisord.conf` (relative to executable)
6. `../supervisord.conf` (relative to executable)