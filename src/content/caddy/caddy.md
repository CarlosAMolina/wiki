# Caddy

## Caddyfile

An example given by Caddy can be checked at /etc/caddy/Caddyfile.

Example with a log file:

```
:8080 {
    respond "Hello, world!"
    log {
        output file /tmp/access.log {
            roll_size 10MB # Create new file when size exceeds 10MB
            roll_keep 5 # Keep at most 5 rolled files
        }
    }
}
```

## References

- [Caddy documentation](https://caddyserver.com/docs/)
