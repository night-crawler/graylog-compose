# graylog

## Preconditions

1. This graylog config is designed to use with [docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion) in __multi-network__ mode.
2. Second network is used to reach the graylog instance from nginx container.
3. Default forwarded TCP/UDP Input ports should be used only by the host itself.
4. Do not forward ports to 0.0.0.0.

## Deployment

1. `apt install pwgen`.
2. Generate two secrets:
    - `pwgen 64 1` - secret
    - `pwgen 16 1` - password
3. Generate SHA2 password hash: `echo -n yourpassword | shasum -a 256` and save it to `./graylog/config/graylog.conf` `root_password_sha2`.
4. Save your secret in `password_secret` variable.
5. Set public URI `http_external_uri` to actual hostname, i.e. `http://graylog.test/`. Slash is important.
6. Add lines in your host's `/etc/sysctl.conf`:

        fs.inotify.max_user_watches=524288
        vm.max_map_count=262144
7. Run `sysctl -p`.
8. Create a new graylog network:

        docker network create --attachable --driver overlay --subnet 10.100.100.1/24 graylog-network
9. Edit docker-compose-letsencrypt-nginx-proxy-companion `.env` and set `SERVICE_NETWORK=graylog-network`.
10. Set correct `LETSENCRYPT_HOST` && `LETSENCRYPT_EMAIL`.

Nginx cannot start anymore without graylog instance running, because it cannot resolve `graylog.local`. Be careful.

## Configuration

### Host rsyslog configuration

Create a new rsyslog.d config `/etc/rsyslog.d/graylog.conf`:

    *.* @127.0.0.1:514;RSYSLOG_SyslogProtocol23Format

### Container nginx configuration

- patch `log_format` in `nginx.tmpl` according to scheme specified in content pack docs
- comment out `access_log=off` in all virtual host sections

### Content packs

- [Generic syslog](https://github.com/jkumar2001/graylog-generic-syslog)
- [Nginx syslog](https://github.com/graylog-labs/graylog-contentpack-nginx)
- [Nginx 1.11.8 JSON syslog](https://github.com/petestorey26/graylog-content-pack-nginx-json)

## TODO

- Make it swarm
