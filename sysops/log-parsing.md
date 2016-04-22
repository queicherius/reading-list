# Log parsing

## GoAccess

> GoAccess is an open source real-time web log analyzer and interactive viewer that runs in a terminal in *nix systems. It provides fast and valuable HTTP statistics for system administrators that require a visual server report on the fly.

- http://goaccess.io/

### Install

```
apt-get install libncursesw5-dev libgeoip-dev make
wget http://tar.goaccess.io/goaccess-0.9.8.tar.gz
tar -xzvf goaccess-0.9.8.tar.gz
cd goaccess-0.9.8/
./configure --enable-geoip --enable-utf8
make
make install
```

### Parse haproxy log files

```
goaccess -f haproxy.log --log-format='%^  %^ %^:%^:%^ %^ %^[%^]: %h:%^ [%d:%t.%^] %^ %^ %^/%^/%^/%^/%L %s %b %^ %^ %^ %^/%^/%^/%^/%^ %^/%^ "%r"' --date-format='%d/%b/%Y' --time-format='%H:%M:%S' -q
```
