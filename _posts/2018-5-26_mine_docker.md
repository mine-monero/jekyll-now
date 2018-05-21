---
layout: post
title: How to mine Monero behind a Tor router with Docker
---


This is a complete tutorial on how to connect a Docker container to a Tor router.

For the example, we will connect a Monero miner, but you can connect really any container you like. In fact, we will also connect a full desktop browser to check that we are really in the Tor network.

Enjoy!

# Step 1: set up Tor

    docker run -d --name tor-router --cap-add NET_ADMIN --dns 127.0.0.1 flungo/tor-router
    
That's it! But you can also inspect the logs of the tor router with `docker logs tor-router`.

The container's now set up to route also the DNS requests through Tor, but you can specify the dns you want.

# Step 2: check that you're really behind Tor

Just to be sure. Run a browser attached to the connection of the container created at the step before:

    docker run -it --rm --net=container:tor-router -ti --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix dmouse/browser
    
I'm not sure it will work also on Windows, because of the `$DISPLAY` variable...you can skip this step, or dive into the container and surf from the command line :)

If you have a browser, check these sites:

1. https://check.torproject.org
2. https://www.dnsleaktest.com
    
# Step 3: launch a miner

First we need a Docker container with a miner. We could build one ourselves, but let's go through the lazy way and use the one from servethehome.com.

In this example I put an example pool and address, choose the one you prefer! (you'll probably want to put your wallet address, not mine)

```bash
#!/bin/bash
#for Debian based
sudo sysctl vm.nr_hugepages=128

wallet=46M2tRgiT5wduamnv9cbpQHXxQcwFWcTtVDPF1AZ4mm7fjB8zfkuCDf7ofR98FAvFEVkgUtgDx56h3SSzceuZkzSJRjcRy2
numthr=8
pool=pool.supportxmr.com
port=5555
pass=x
image=servethehome/universal_cryptonight:latest

docker run --net=container:tor-router -itd -e xmrpool=$pool -e startport=$port -e username=$wallet -e pass=$pass -e numthreads=$numthr --cpuset-cpus="0-7" $image
docker run --net=container:tor-router -itd -e xmrpool=$pool -e startport=$port -e username=$wallet -e pass=$pass -e numthreads=$numthr --cpuset-cpus="8-15" $image

```

As you see, we launch our container connected to the network of the tor container created above. The example above shows how to exploit 16 cores, change as you like.
    
# Donations

If you like this tutorial, you can show your appreciation to the following address:

46M2tRgiT5wduamnv9cbpQHXxQcwFWcTtVDPF1AZ4mm7fjB8zfkuCDf7ofR98FAvFEVkgUtgDx56h3SSzceuZkzSJRjcRy2

BTW, let me know if you're insterested in a similar howto on how to run a docker container behind a VPN, for now I leave it as an exercise to the reader.

Comments [here](https://github.com/mine-monero/mine-monero.github.io/issues).

