# Log parsing

## GoAccess

> GoAccess is an open source real-time web log analyzer and interactive viewer that runs in a terminal in *nix systems. It provides fast and valuable HTTP statistics for system administrators that require a visual server report on the fly.

- http://goaccess.io/

**Parse haproxy log files:**

```
goaccess -f haproxy.log --log-format='%^  %^ %^:%^:%^ %^ %^[%^]: %h:%^ [%d:%t.%^] %^ %^ %^/%^/%^/%^/%L %s %b %^ %^ %^ %^/%^/%^/%^/%^ %^/%^ "%r"' --date-format='%d/%b/%Y' --time-format='%H:%M:%S' -q
```
