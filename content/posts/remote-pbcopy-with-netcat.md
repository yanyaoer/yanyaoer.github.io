---
title: remote pbcopy with netcat
date: 2015-01-20 11:45:53
---


## Quick start

```bash
while (true); do nc -l 2224 | pbcopy; done
#If your laptop is running linux, replacing pbcopy with xcopy should work:
#while (true); do nc -l 2224 | xcopy; done

echo "This text gets sent to clipboard" | nc localhost 2224

echo "RemoteForward 2224 localhost:2224" >> ~/.ssh/config
ssh remote -t 'cat blablabla | nc -q0 localhost 2224'
```


## Daemonizing pbcopy 

`launchctl load ~/Library/LaunchAgents/local.pbcopy.plist`

```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>localhost.pbcopy</string>
    <key>ProgramArguments</key>
    <array>
       <string>/usr/bin/pbcopy</string>
    </array>
    <key>inetdCompatibility</key>
    <dict>
       <key>Wait</key>
       <false/>
    </dict>
    <key>Sockets</key>
    <dict>
       <key>Listeners</key>
       <dict>
           <key>SockServiceName</key>
           <string>2224</string>
           <key>SockNodeName</key>
           <string>127.0.0.1</string>
       </dict>
    </dict>
</dict>
</plist>
```


## Remote pbcopy

```bash
scp pbcopy remote:/path_to/pbcopy
chmod a+x path_to/pbcopy

#!/bin/bash

[ -n "$SSH_CLIENT" ] && SESSION_TYPE="remote"

if [[ $SESSION_TYPE == "remote" ]]; then
  cat | nc -q0 localhost 2224
else
  cat | pbcopy
fi
```

## links

<http://brettterpstra.com/2014/02/19/remote-pbcopy-on-os-x-systems/>

<http://seancoates.com/blogs/remote-pbcopy>

<http://evolvingweb.ca/blog/exposing-your-clipboard-over-ssh>
