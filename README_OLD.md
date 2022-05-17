# Docker setup: netopeer2

- [Docker setup: netopeer2](#docker-setup-netopeer2)
  - [NETCONF server (in Docker container)](#netconf-server-in-docker-container)
  - [NETCONF client (on host OS)](#netconf-client-on-host-os)
  - [NETCONF client (in Docker container)](#netconf-client-in-docker-container)

## NETCONF server (in Docker container)

`docker system prune` removes all stopped containers, dangling images and unused networks

- **terminal 1**: run NETCONF server in `sysrepo` container
  - `docker pull sysrepo/sysrepo-netopeer2`
  - `docker run -i -t --name sysrepo sysrepo/sysrepo-netopeer2`
    - _For interactive processes, you must use `-i -t` together in order to allocate a tty for the container process_ ([reference](https://docs.docker.com/engine/reference/run/#foreground))

  ![/images/terminal-1](/images/terminal-1.png)

- **terminal 2**: connect to the NETCONF server via SSH
  - `docker inspect sysrepo | grep -w "IPAddress"`
    - `-w`, `--word`: _Select only those lines containing matches that form whole words_ ([reference](https://linuxcommand.org/lc3_man_pages/grep1.html))
  - `docker inspect sysrepo | grep -A1 -w "Ports"`
    - `-A NUM`: _Print NUM lines of trailing context after matching lines_ ([reference](https://linuxcommand.org/lc3_man_pages/grep1.html))
  - `ssh netconf@172.17.0.2 -p 830 -s netconf`
    - password: `netconf`
    - `-s ctl_path`: _Specifies the location of a control socker for connection sharing_ ([reference](https://linux.die.net/man/1/ssh))

  ![/images/terminal-2](/images/terminal-2.png)

- **terminal 3**: access `sysrepoctl` or `sysrepocfg` exec bash in the `sysrepo` container
  - `docker exec -it sysrepo /bin/bash`
  - `sysrepoctl -l`
    - _sysrepoctl is a command-line tool for manipulation of YANG schemes in sysrepo (list currently installed schemas and add, remove or modify them)_ ([reference](https://manpages.debian.org/unstable/sysrepo/sysrepoctl.1.en.html))
  - `sysrepocfg`
    - _sysrepocfg allows to work with configuration in many ways such as importing, exporting, editing and replacing (copying-from a file or datastore) it. It is also possible to send an rpc/action or a notification_ ([reference](https://netopeer.liberouter.org/doc/sysrepo/libyang1/html/sysrepocfg.html))
- **_alternative_ terminal 3**: connect to NETCONF server via [`testconf`](https://hub.docker.com/r/sysrepo/testconf/)
  - `docker run -i -t --link sysrepo --name testconf --rm sysrepo/testconf:latest bash`
    - `--rm`: _By default a container's file system persist even after the container exits. This makes debugging easier (since you can inspect the final state) and you retain all your data by default. But if you are running short-term foreground processes, these container file systems can really pile up. If instead you'd like Docker to automatically clean up the container and remove the file system when the container exits, you can add this flag._ ([reference](https://docs.docker.com/engine/reference/run/#clean-up---rm))
  - `sysrepoctl -l` 
  - `sysrepocfg`

  ![/images/terminal-3](/images/terminal-3.png)

[source](https://hub.docker.com/r/sysrepo/sysrepo-netopeer2)

## NETCONF client (on host OS)

Now we can try connecting a NETCONF client running on our computer to the NETCONF server running inside the Docker container.

Open a new terminal window

`sudo -i`

`netopeer2-cli -v3`

`help`: displays all commands

`connect --host 172.17.0.2 --port 830 --login netconf`

![images/netopeer-client-1.png](/images/netopeer-client-1.png)

`get-config --source running`: get the running netopeer2-server configuration

```xml
DATA
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <keystore xmlns="urn:ietf:params:xml:ns:yang:ietf-keystore">
    <asymmetric-keys>
      <asymmetric-key>
        <name>genkey</name>
        <algorithm>rsa2048</algorithm>
        <public-key>...</private-key>
      </asymmetric-key>
    </asymmetric-keys>
  </keystore>
  <netconf-server xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-server">
    <listen>
      <endpoint>
        <name>default-ssh</name>
        <ssh>
          <tcp-server-parameters>
            <local-address>0.0.0.0</local-address>
            <keepalives>
              <idle-time>1</idle-time>
              <max-probes>10</max-probes>
              <probe-interval>5</probe-interval>
            </keepalives>
          </tcp-server-parameters>
          <ssh-server-parameters>
            <server-identity>
              <host-key>
                <name>default-key</name>
                <public-key>
                  <keystore-reference>genkey</keystore-reference>
                </public-key>
              </host-key>
            </server-identity>
            <client-authentication>
              <supported-authentication-methods>
                <publickey/>
                <passsword/>
                <other>interactive</other>
              </supported-authentication-methods>
            </client-authentication>
          </ssh-server-parameters>
        </ssh>
      </endpoint>
    </listen>
  </netconf-server>
</data>
```

## NETCONF client (in Docker container)

