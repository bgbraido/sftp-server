# SFTP Server with Docker Compose

A complete Docker Compose setup for running an SFTP (SSH File Transfer Protocol) server.

## Quick Start

### 1. Start the Server

```bash
docker-compose up -d
```

### 2. Connect to the Server

```bash
sftp -P 2222 testuser@localhost
```

When prompted, enter the password: `testpass`

### 3. Upload/Download Files

```bash
# Upload files
put /path/to/local/file

# Download files
get /path/to/remote/file

# List files
ls
```

## Configuration

### Environment Variables

The `.env` file controls the SFTP server configuration. Copy and customize the provided `.env` file:

```bash
# Example .env file content:

# SFTP Port (exposed on host machine)
SFTP_PORT=2222

# SFTP Users - format: username:password:uid[:gid[:dir1[:dir2[:...]]]]
SFTP_USERS=testuser:testpass:1001

# Container settings
CONTAINER_NAME=sftp-server
RESTART_POLICY=unless-stopped
```

### Environment Variables Explained

- **SFTP_PORT**: The port number to expose on your host machine (default: 2222). The container listens on port 22 internally, but you access it via this port.
- **SFTP_USERS**: User credentials (see format below)
- **CONTAINER_NAME**: Docker container name
- **RESTART_POLICY**: Restart behavior (unless-stopped, always, no, on-failure)

### User Format

```
username:password:uid[:gid[:dir1[:dir2[:...]]]]
```

| Field | Required | Example | Description |
|-------|----------|---------|-------------|
| `username` | Yes | `testuser` | Login username |
| `password` | Yes | `testpass` | Login password |
| `uid` | Yes | `1001` | User ID (numeric, must be unique) |
| `gid` | No | `1001` | Group ID (defaults to uid) |
| `dir1, dir2` | No | `upload`, `files` | Directories to create in home |

### Example .env Files

**Single User (Basic)**
```env
SFTP_PORT=2222
SFTP_USERS=testuser:testpass:1001
CONTAINER_NAME=sftp-server
RESTART_POLICY=unless-stopped
```

**Multiple Users**
```env
SFTP_PORT=2222
SFTP_USERS=user1:pass1:1001 user2:pass2:1002 admin:adminpass:1000
CONTAINER_NAME=sftp-server
RESTART_POLICY=unless-stopped
```

**Users with Directories**
```env
SFTP_PORT=2222
SFTP_USERS=john:john123:1001::uploads documents
CONTAINER_NAME=sftp-server
RESTART_POLICY=unless-stopped
```

**Production Setup**
```env
SFTP_PORT=2222
SFTP_USERS=backup:your-strong-password-here:2000 files:another-password:2001
CONTAINER_NAME=sftp-prod
RESTART_POLICY=always
```

## Volume Mounts

The `docker-compose.yml` mounts:

- `./data:/home/testuser/upload` - User data directory
- `./ssh_keys:/etc/ssh/keys` - SSH key storage

Create these directories if they don't exist:

```bash
mkdir -p data ssh_keys
chmod 700 ssh_keys
```

## SSH Key-Based Authentication (Optional)

To use SSH keys instead of passwords:

1. Generate keys:

```bash
ssh-keygen -t rsa -f ssh_keys/id_rsa -N ""
```

2. Update `.env`:

```env
SFTP_USERS=testuser::1001
```

3. Place public keys in `ssh_keys/public_keys/testuser`

## Useful Commands

```bash
# View logs
docker-compose logs -f sftp-server

# Stop the server
docker-compose down

# Rebuild the image
docker-compose build --no-cache

# Access container shell
docker exec -it sftp-server bash

# List active connections
docker exec sftp-server ps aux
```

## Security Recommendations

1. **Change default credentials** in `.env`
2. **Use SSH keys** instead of passwords
3. **Restrict file permissions**: `chmod 700` for ssh_keys
4. **Use strong passwords** if password auth is enabled
5. **Monitor logs** regularly
6. **Use firewall rules** to restrict access

## Troubleshooting

### Connection Refused

```bash
# Check if container is running
docker-compose ps

# Check logs
docker-compose logs sftp-server
```

### Permission Denied

```bash
# Fix directory permissions
chmod 700 ssh_keys
chmod 755 data
```

### Users Not Found

```bash
# Verify user format in .env
# Should be: username:password:uid
```

## Default Credentials

- **Username**: testuser
- **Password**: testpass
- **Port**: 2222 (to local port, internal SSH port is 22)
- **Home**: /home/testuser

## References

- [atmoz/sftp Docker Image](https://github.com/atmoz/sftp)
- [SFTP Protocol](https://tools.ietf.org/html/draft-ietf-secsh-filexfer-extensions)
- [OpenSSH Documentation](https://man.openbsd.org/sshd)
