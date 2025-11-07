# gns3-docker
GNS3 Web-ui for Network Trailning
# GNS3 Docker Prototype — Quick Start


Prereqs: Docker Engine & Docker Compose (v2+).


1. Copy `.env.example` to `.env` and edit `BASE_DOMAIN` and cert/email fields.


2. Build and start services:


```bash
docker compose up -d --build
