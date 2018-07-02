# graylog

## Preconditions

1. This graylog config is designed to use with [docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion).
2. Second network is used to reach the graylog instance from other containers outside of `nginx-network`.
3. Default forwarded TCP/UDP Input ports should be used only by the host itself.
4. Do not forward ports to 0.0.0.0.

### DOCKER SERVICE DISCOVERY WARNING

Overlay attachable networks may have IP resolving bugs since Docker `>17.12.1`.
IP addresses of deleted or previously stopped containers are not properly cleaned from Docker guts causing stale resolve responses.
[Issue](https://github.com/moby/moby/issues/30487)
To check if everything is OK run `nslookup <container_alias>`. It should return only correct addresses actually present in your installation. 

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

        docker network create --attachable --driver overlay --subnet 10.100.100.0/24 graylog-network
9. Adjust `LETSENCRYPT_HOST` && `LETSENCRYPT_EMAIL`.

Nginx cannot start anymore without graylog instance running, because it cannot resolve `graylog.test`. Be careful.

## Configuration

### Host rsyslog configuration

Create a new rsyslog.d config `/etc/rsyslog.d/graylog.conf`:

    *.* @127.0.0.1:514;RSYSLOG_SyslogProtocol23Format

### Container nginx configuration

- patch `log_format` in `nginx.tmpl` according to scheme specified in content pack docs
- comment out `access_log=off` in all virtual host sections

### Geo data

- Download [GeoLite2 Database](https://dev.maxmind.com/geoip/geoip2/geolite2/) and extract it in `./geo` directory
- Update System / Configurations / Geo-Location Processor settings with a new path: `/var/geo/GeoLite2-City.mmdb`
- Put GeoIP resolver last in section `Message Processors Configuration`
- [docs](http://docs.graylog.org/en/2.4/pages/geolocation.html)

### Content packs

- [Generic syslog](https://github.com/jkumar2001/graylog-generic-syslog)
- [Nginx syslog](https://github.com/graylog-labs/graylog-contentpack-nginx)
- [Nginx 1.11.8 JSON syslog](https://github.com/petestorey26/graylog-content-pack-nginx-json)

## TODO

- [~~Make it swarm~~](https://github.com/moby/moby/issues/32299)
