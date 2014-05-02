# A Tilestream Server

Some notes and configuration for running a simple Tilestream server on EC2.  We occasionally have need to run a Tilestream server, but the goal is not to be a permanent solution as Mapbox is overall cheaper and more robust.

Using: Ubuntu Server 14.04 LTS (PV) - ami-018c9568 (64-bit)

## Install

These are just more specific instructions that the [ones that come with Tilestream](https://github.com/mapbox/tilestream/tree/master#installation-ubuntu-1004).

1. `sudo apt-get install curl build-essential libssl-dev libsqlite3-0 libsqlite3-dev git-core nodejs npm nginx;`
1. Node installs as `nodejs` so create a link: `sudo ln -s /usr/bin/nodejs /usr/bin/node;`
1. Make place for Tilestream: `mkdir -p ~/applications && cd ~/applications;`
1. Get Tilestream: `git clone https://github.com/mapbox/tilestream.git && cd tilestream && npm install;`
1. Had to install sqlite separately: `npm install sqlite3`

### Configure Tilestream

1. Get this repository: `cd ~/applications/ && git clone https://github.com/MinnPost/tilestream-server.git;`
1. Copy config file: `cp tilestream-server/tilestream.json ./;`
    * Update hosts as needed.
1. Test with (changing host appropriately): `./tilestream/index.js start --config=tilestream.json
    * Go to in browser: http://ec2-54-82-59-19.compute-1.amazonaws.com:9000/

## Uploading tiles

Put files here as Tilestream will make this directory when you run as this user: `/home/ubuntu/Documents/MapBox/tiles`

Here's an example using a named host in your `.ssh/config` file:

    scp -F ~/.ssh/config ~/Downloads/hennepin-parcels_6b3a38.mbtiles minnpost-tilestream:/home/ubuntu/Documents/MapBox/tiles/hennepin-parcels.mbtiles

## Shore up the edges

We want to make sure that Tilestream start automatically.

1. Copy the upstart script: `sudo cp tilestream-server/tilestream.conf /etc/init/tilestream.conf`
1. Start server: `sudo start tilestream`

We use `nginx` for some simple caching

1. Make directory for cache: `sudo mkdir -p /data/nginx/cache`
1. Copy conf to nginx: `sudo cp tilestream-server/tilestream.nginx.conf /etc/nginx/sites-available/tilestream.nginx.conf`
1. Enable it: `sudo ln -s /etc/nginx/sites-available/tilestream.nginx.conf /etc/nginx/sites-enabled/tilestream.nginx.conf`
1. Remove default config: `sudo rm /etc/nginx/sites-enabled/default`
1. Restart nginx: `sudo service nginx restart`

Finally

1. For the UI, go here: http://ec2-54-82-59-19.compute-1.amazonaws.com:9000/
2. For use in maps, use things like:
    * http://ec2-54-82-59-19.compute-1.amazonaws.com:9003/v2/hennepin-parcels/10/245/368.png
    * http://ec2-54-82-59-19.compute-1.amazonaws.com:9003/v2/hennepin-parcels/10/245/368.grid.json
    * http://ec2-54-82-59-19.compute-1.amazonaws.com:9003/v2/hennepin-parcels/{z}/{x}/{y}.png
