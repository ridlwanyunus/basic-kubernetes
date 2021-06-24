# Problems and solving

## Tidak dapat login ke docker.io pada ubuntu
Pesan error yang muncul
```bash
$ docker login -u xxxxxxxx
Password:
Error saving credentials: error storing credentials - err: exit status 1, out: `Error spawning command line “dbus-launch --autolaunch=ec2a0ab6dc831dcxxxxxxxxxxxxxxxxxx --binary-syntax --close-stderr”: Child process exited with code 1`
```
Install `gnupg2` dan `pass`
```bash
$ sudo apt install gnupg2 pass
```