# eduroam-freeradius-sp
Docker Container containing FreeRadius for an Eduroam SP-only configuration.

Eudroam is the secure, world-wide roaming access (IE Internet) service developed for the international research and education community. You will need to sign up to host Eduroam. Once you receive access to the Eduroam Administrator console, you should be able to generate a shared secret that you will to federate requests to the federation level Radius servers (FLR).

This uses an Alpine Linux image, installs FreeRadius and a few other tools and then copies configurations to the proper place. I chose to use an Alpine image because the freeradius/freeradius-server Alpine images do not include ARM64 architecture (I wanted to run this on a Raspberry Pi).

The /config folder is mounted read-only within the container. The /config/run.sh is what is called when the container starts. This script will copy the configuration from this folder to the proper place, fix permissions on the logging folder, and then start FreeRadius. A second mount is exposed to the host which contains the log files from FreeRadius, in-order to satisfy logging recommendations.

This was configured following the instructions on GEANT's [wiki](https://wiki.geant.org/display/H2eduroam/eduroam+SP).

## Installation

* git clone the repo
* In the /config folder copy the following files:
    * clients.conf.sample to clients.conf
    * eduroam.sample to eduroam
    * proxy.conf.sample to proxy.conf
* Ensure the /config/run.sh is executable (You may need to run chmod 700 /config/run.sh)
* Edit the aformentioned files replacing any `<<VARIABLES>>` within the files with the correct values
* Edit the docker-compose.yml file. Ensure the binding for the logs is in a location that exists.
* run `docker-compuse up -d` . This should create the Docker image, copy the configuration you provided, and start FreeRadius

## Troubleshooting

* To access a currently running container run: `docker exec -it eudroam-freeradius-sp /bin/bash`
* To start the container without starting FreeRadius: `docker run --rm -it -v /home/user/repos/eduroam-freeradius-sp/config:/config:ro -v /home/freeradius/logs:/var/logs/radius eduroam-freeradius-sp:1 /bin/bash` . If you want to start FreeRadius run /config/run.sh
* To restart the container from the eduroam-freeradius-sp folder run: `docker-compose restart freeradius`
* To test with eapol_test:
    * Access the container with the instructions above
    * Copy the /config/test.conf.template to /root/test.conf
    * Edit the test.conf replacing the username, password, and anonymous identity.
    * Run `/sbin/eapol_test -c /root/test.conf -a 127.0.0.1 -p 1812 -s TestSecret`
* Docker under WSL does source NAT on connections to the container. This changes the IP address that FreeRadius sees connecting. You can confirm this by checking the radius.log file or by running `tcpdump udp port 1812` and then sending another request to the Radius server.

## Uninstallation
From the eduroam-freeradius-sp folder run `docker-compose down`