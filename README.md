# eaas-deployment-m 🦀 🧜🏻‍♀️ 🐿️

All rust, all minimized statically linked binaries, three microservices that cover many needs.

This repository contains Kubernetse manifest for morpho-web, merflow, and squirrel-tactix stack.

Template/references for the microservices:

- https://github.com/jpegleg/morpho-web - actix-web front end using rustls and actix middleware
- https://github.com/jpegleg/merflow - redis cache hydration via merflow sidecar
- https://github.com/jpegleg/squirrel-tactix - actix-web backend using Diesel and postgresql

The design is for postgresl and redis storage. The postgresql and redis deployments are not included in this repository.

Morpho is the exposed service, all other services are filtered/hidden away in this design.
Merflow can read from postgres and write to redis, squirrel-tactix can read and write with postgres, and the morpho-web services
can read and write in redis and send API requests to the squirrel-tactix instance.

In this way, the merflow populates redis with postgres contents so that the morpho-web can read from redis for most data reads,
while using async HTTP to perform database writes via squirrel-tactix calls. Morpho-web is the center of activity, while
the other two services deal with different types of persistent or semi-persistent data.
