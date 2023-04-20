# Learning eBPF with Docker

A Docker environment for experimenting with eBPF.

This image includes the [Learning eBPF](https://github.com/lizrice/learning-ebpf) repository by Liz Rice and all the tools that you need to get started with eBPF.

## Get a copy of this repo

```sh
git clone https://github.com/cthain/learning-ebpf-docker.git
cd learning-ebpf-docker/
```

## Build the Docker image locally

This will create a BPF-enabled Ubuntu image with:
- libbpf
- bpftool
- learning-ebpf installed in `/src/learning-ebpf`

It may take a couple minutes to build.

```sh
docker build -t learning-ebpf:latest .
```

## Run the Docker container

**Note**: You **must** run the container using `--privileged` flag to enable support for BPF.

```sh
docker run --privileged -it --rm --name learning-ebpf learning-ebpf
```

Now that you're inside the container (as `root`!!) you can play around with the chapter exercises from Learning eBPF.

**Note**: If you want to run python BCC examples you must mount the `debugfs`. It is not mounted by default:

```sh
mount -t debugfs none /sys/kernel/debug
```

### Chapter 2 (python/BCC):

```sh
cd /src/learning-ebpf/chapter2
./hello.py
```

In a new terminal window, `exec` into the container to trigger some syscall events that will be logged in the first terminal window:

```sh
docker exec -it learning-ebpf /bin/bash
```

### Chapter 3 (C):

```sh
cd /src/learning-ebpf/chapter3
make hello
bpftool prog load hello.bpf.o /sys/fs/bpf/hello
bpftool prog list | grep 'name hello' -A3
export PROG_ID=$(bpftool prog list | grep 'name hello' | cut -d':' -f 1) && echo $PROG_ID
bpftool prog show id $PROG_ID --pretty | jq .
bpftool net attach xdp id $PROG_ID dev eth0
ip link
bpftool prog tracelog
^C
```

Cleanup:

```sh
bpftool net detach xdp dev eth0
rm /sys/fs/bpf/hello
```
