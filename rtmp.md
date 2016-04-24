# Setup Nginx-RTMP on Ubuntu 14.04

## Installing Nginx and Nginx-RTMP

```
apt-get install build-essential libpcre3 libpcre3-dev libssl-dev
mkdir ~/working
cd ~/working
```

```
wget http://nginx.org/download/nginx-1.7.5.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
```

## Reference

https://www.vultr.com/docs/setup-nginx-rtmp-on-ubuntu-14-04

https://github.com/arut/nginx-rtmp-module/wiki/Directives
https://github.com/arut/nginx-rtmp-module/wiki/Getting-started-with-nginx-rtmp

## Test

publish
```
ffmpeg -re -f concat -i list.txt -c copy -f flv rtmp://ecs3.shbaiche.com/live/mystream
```

play
```
ffplay rtmp://ecs3.shbaiche.com/live/mystream
```
