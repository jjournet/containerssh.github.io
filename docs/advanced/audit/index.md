<h1>Audit logging {{ upcoming("0.4.0") }}</h1>

!!! warning
    This is a feature still in development and is only available when [building from source](https://github.com/containerssh/containerssh/).

ContainerSSH contains an audit logging facility that can log every interaction happening over SSH. This functionality is disabled by default as it has serious security and privacy implications, as well as severe resource requirements.

Audit logging can be enabled in the configuration using the following structure:

```yaml
audit:
  format: none|audit|asciinema # Which format to log in. Defaults to none.
  storage: none|s3|file        # Where to write audit log. Defaults to none.
  intercept:
    stdin: true|false          # Intercept keystrokes from user
    stdout: true|false         # Intercept standard output
    stderr: true|false         # Intercept standard error
    passwords: true|false      # Intercept passwords during authentication
```

Audit logging is a powerful tool. It can capture the following events.

- Connections
- Authentication attempts, optionally with credentials
- Global and channel-specific SSH requests
- Programs launched from SSH
- Input from the user (optional)
- Output and errors to the user (optional)

The events recorded depend on the chosen format. With the `audit` format all information is recorded with nanosecond timing, so events can be accurately reconstructed after the fact.

## About interceptions

The `intercept` options give you a wide range of options when it comes to detailed logging of actions by users. You may want to, for example, enable `stdout` logging while keeping `stdin` disabled to avoid accidentally capturing passwords typed into the console.

However, this approach may fail if SFTP is enabled as you will fail to capture binaries uploaded to the server. Audit logging should therefore be enjoyed with great care and the logs should always be stored on an encrypted storage device.

## Log formats

### The `audit` format (recommended)

The audit format is intended for an accurate reconstruction of everything happening during an SSH session. It allows for accurate reconstruction of what happened during the session.

Audit logs are stored in a [compressed binary format](format.md) and can be decoded to a series of JSON messages using the `containerssh-auditlog-decoder` supplied as part of the ContainerSSH release. Alternatively, you can [implement your own decoder](format.md).

### The `asciinema` format

The [asciinema format](https://github.com/asciinema/asciinema/blob/develop/doc/asciicast-v2.md) stores logs in a format suitable for replay in the [Asciinema player](https://asciinema.org/).

!!! note
    Make sure you enable the `stdout` and `stderr` interceptions, otherwise the `asciinema encoder won't capture anything 

!!! warning
    Asciinema is intended for entertainment purposes only and doesn't store all relevant information required for an accurate audit log.

## Storage backends

### The `s3` storage (recommended)

The S3 storage sends the logs to an S3-compatible object storage for long term storage. This is the recommended way of storing audit logs because it is a server-independent storage device that supports permissions.


The S3 storage can be configured as follows:

```yaml
audit:
  storage: s3
  s3:
    local: /local/storage/directory
    accessKey: "your-access-key-here"
    secretKey: "your-secret-key-here"
    bucket: "your-existing-bucket-name-here"
    region: "your-region-name-here"
    endpoint: "https://your-custom-s3-url" # Optional
    cacert: | # Optional
      Your trusted CA certificate in PEM format here for your S3 server.
```

!!! warning
    Since the S3 upload can be slow, the S3 storage requires a local directory. This directory should be stored on a persistent storage and must not be shared between multiple ContainerSSH instances.

!!! tip
    You may also want to investigate if your S3 provider supports WORM / object locking, object lifecycles, or server side encryption for compliance.

### The `file` storage

The file storage writes audit logs to files on the disk. The storage location can be configured using the following option:

```yaml
audit:
  type: file
  file:
    directory: /var/log/audit
```