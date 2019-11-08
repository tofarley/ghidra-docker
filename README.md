# ghidra-docker

This is a dockerized version of Ghidra, meant as a server for multi-user
projects, and for headless analysis.  By default, it stays within whatever
memory limits are set set in Docker or Kubernetes on the container (via the
[container
awareness](https://blog.docker.com/2018/04/improved-docker-container-integration-with-java-10/)
in Java 10+). Instead of running Ghidra's Linux service, this container runs
the Ghidra server directly, and is configured to log to stdout.

## Docker

If you want to have a user created on first start with the default password of
'changeme', set `GHIDRA_DEFAULT_USERS` to the comma-separated usernames.

For example, to run Ghidra server in a container with a memory limit of 1GB and
create users named `esfried` and `ghidra`, use:

```bash
docker run -it --rm -m 1G --env HOST_IP=$(curl -s http://whatismyip.akamai.com/) --env GHIDRA_DEFAULT_USERS=tofarley,wnshobe -p 13100-13102:13100-13102 <image>
```

To connect locally:
```bash
docker run -it --rm -m 1G --env HOST_IP=127.0.0.1 --env GHIDRA_DEFAULT_USERS=tofarley -p 13100-13102:13100-13102 <container>
```

We pass the environment variable `$HOST_IP` to the container to allow us to bind
in spite of ghidra's reverse DNS requirements (See Documentation).

If you would like to pass any additional flags to the Ghidra server, set
`GHIDRA_FLAGS` to specify the flags and values. 

For example, to run the Ghidra server with anonymous access enabled and the
password reset window set to 3 days instead of 1, use:

```bash
docker run -it --rm --env "GHIDRA_FLAGS=-anonymous -e3" bskaggs/ghidra
```

## Helm

There is also a Helm chart for Kubernetes in the [charts/ghidra-server charts
directory](/charts/ghidra-server) that will create a one-pod StatefulSet with a
persistent volume for storing the repository information.

## Headless Analysis

You can use Ghidra for headless analysis; be sure to read
`support/analyzeHeadlessREADME.html` in the Ghidra distribution to find out
more.

User names are by default based on the OS user name, so it's easiest to make one
for the user running the GUI, and one for `ghidra` for headless analysis in a docker
container.  However, if you'd like, you can change your user name when launching
Ghidra (either with the GUI, or headless in docker) by setting the following
environment variable:

```bash
VMARGS=-Duser.name=esfried
```

To create the initial repository on the server, you must currently connect once
via the GUI (instructions will change once source code is released).  Create a
shared project (`foo` in our example) with your user as the admin, and make sure
to let the `ghidra` user we will use for the headless analysis have Read/Write
access.  

If, for example, the Ghidra server container you created is running with the IP
address on the docker bridge of 172.17.0.2, you can then launch an analyzer
container.  In this case, we will mount in `/usr/bin` on the host as `/data`,
and then analyze every binary in the directory:

```bash
echo changeme | docker run -i --rm -m 4G -v /usr/bin:/data:ro bskaggs/ghidra \
     support/analyzeHeadless ghidra://172.17.0.2/foo -p -import /data
```

If you have your GUI also logged in to the server, you will see the programs
being added as they are analyzed
