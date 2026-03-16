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
- `-e, --editor CMD` - Editor command (default: nano)
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

## How it works

1. `socat` listens on the configured TCP port and forks a subprocess per incoming connection, re-executing this script with `--handle-connection` so stdin/stdout are wired directly to the socket.
2. On connect, the handler sends a greeting line and then reads the rmate protocol stream. Each `open` command carries headers (token, display-name, selection, data length) followed by the raw file bytes, which are written to a temp file under a per-session directory.
3. The temp file is opened in the configured editor asynchronously. A background subshell runs `inotifywait -e close_write` in a loop on that file; whenever the editor saves it, the updated contents are streamed back to the remote over the same socket (`save` response).
4. When the connection closes (EOF on stdin), an `EXIT` trap kills all watchers, sends `close` notifications for each open file, and removes the session temp directory.

## Remote client

Use [aurora/rmate](https://github.com/aurora/rmate) on the remote machine to connect to this server.

## References

- https://github.com/rafaelmaiolla/remote-vscode
- https://github.com/randy3k/RemoteSubl
