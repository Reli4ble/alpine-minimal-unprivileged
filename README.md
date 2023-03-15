# NGINX unprivileged image based on alpine-minirootfs

This repo contains the dockerfiles to build NGINX from the Stable-Version on the latest alpine-minirootfs.

All Dockerfiles uses only Multistage-Builds with Scratch-Images and including only that is needed to run NGINX.

NGINX runs here really unprivileged that means master and worker-processes runs under unprivileged user-accounts.
