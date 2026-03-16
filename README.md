# rmate-server

Editor-agnostic rmate server. Receives files from remote machines via SSH reverse tunnel and opens them in a local editor. Streams saves back.

## SSH setup

```bash
ssh -R 52698:localhost:52698 user@remote-host
# then on remote: rmate file.txt
```

## Remote usage

Set EDITOR variable on the remote to use rmate as the default editor:

```bash
export EDITOR="rmate -w"
# now git commit, less, sudoedit, etc. will open files in your local editor
```

## Commands

- `start` - Start server in background (default)
- `stop` - Stop background server
- `status` - Show server status
- `foreground` - Run in foreground (for debugging)

## Options

- `-H, --host HOST` - Listen address (default: 127.0.0.1)
- `-p, --port PORT` - Listen port (default: 52698)
- `-e, --editor CMD` - Editor command (default: zed)
- `-h, --help` - Show help

## System requirements

- **bash** - POSIX shell
- **socat** - For network socket handling
- **inotify-tools** - For file monitoring (inotifywait)

## Configuration

Configuration can be set via:

1. **CLI arguments** (highest precedence)
2. **Environment variables**: `RMATE_HOST`, `RMATE_PORT`, `RMATE_EDITOR`
3. **Config file**: `~/.config/rmate-server/config` (key=value format: host, port, editor)
4. **Defaults** (lowest precedence)

## Remote client

Use [aurora/rmate](https://github.com/aurora/rmate) on the remote machine to connect to this server.

## References

- https://github.com/rafaelmaiolla/remote-vscode
- https://github.com/randy3k/RemoteSubl
