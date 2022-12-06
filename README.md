# eaas-deployment-m 🦀 🧜🏻‍♀️ 🐿️

All rust, all minimized statically linked binaries, three microservices that cover many needs.

This repository contains Kubernetse manifest for morpho-web, merflow, and squirrel-tactix stack.

Template/references for the microservices:

- https://github.com/jpegleg/morpho-web - actix-web front end using rustls and actix middleware
- https://github.com/jpegleg/merflow - redis cache hydration via merflow sidecar
- https://github.com/jpegleg/squirrel-tactix aka sql via json - actix-web backend using Diesel and postgresql

The design is for postgresl and redis storage. The postgresql and redis deployments are not included in this repository.

Morpho is the exposed service, all other services are filtered/hidden away in this design.
Merflow can read from postgres and write to redis, squirrel-tactix can read and write with postgres, and the morpho-web services
can read and write in redis and send API requests to the squirrel-tactix instance.

In this way, the merflow populates redis with postgres contents so that the morpho-web can read from redis for most data reads,
while using async HTTP to perform database writes via squirrel-tactix calls. Morpho-web is the center of activity, while
the other two services deal with different types of persistent or semi-persistent data.

This manifests insecure backends are used, so wireguard is used to encrypt network traffic and NetworkPolicy is used to retrict access.
Any ingress or exposed ports are to match the morpho service. Firewalls can confidently block external access to the other services, as long
as morpho-web can reach them, and they can internally communicate.

This manifest does not install any calico services, but declares felixconfiguration values to enable eBPF, DSR, iptables cleanup, and encrypt pod-to-pod traffic for the cluster with wireguard automatically. 

```
calicoctl apply -f mms-net.yml
```

Validate that the wireguard encryption is working before using real data. And to improve it futher, flying-squirrel-tactix is the https version of squirrel-tactix, which implements rustls for squirrel-tactix. That works here too! Just be sure
to manage trust on the morpho-web side :)

Securing merflow, redis, and postgres require efforts not included in this repository. This manifest is designed to protect them, but only applies automatic VPN for their network traffic, it doesn't inherently solve security issues in those templates. Morpho-web is extremely secure, and holds the front door for us. Merflow, redis, postgres, and squirrel-tactix are less secure because they don't have proper management of authentication or network encryption built in.
