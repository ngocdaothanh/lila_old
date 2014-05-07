Note about installing on Ubuntu.

## Install Ubuntu server

5GB disk in total is not enough for the default MongoDB in Ubuntu:
Please make at least 3379MB available in /var/lib/mongodb/journal or use —smallfiles

## Install Nginx

See isachy.cz.conf.

See also:
https://github.com/ornicar/lila/tree/ai

## Install MongoDB

```
sudo apt-get install mongodb
```

Confirm MongoDB is running:

```
ps aux | grep mongo
mongo
```

## Install JDK

```
sudo apt-get install openjdk-7-jdk
```

(Oracle JDK is also OK.)

## Install Play

Use Play 2.2.2 (latest version).

```
mkdir /home/jan/opt

cd /home/jan/opt
wget http://downloads.typesafe.com/play/2.2.2/play-2.2.2.zip
sudo apt-get install unzip
unzip play-2.2.2.zip
```

## Install Lila

```
cd /home/jan/opt
sudo apt-get install git
git clone https://github.com/ornicar/lila.git

cd lila
git submodule update --init —recursive
```

Try running Lila (1st run will take several minutes to download dependency .jar files, may need to run several times for all .jar files to be successfully downloaded):

```
cd /home/jan/opt/lila
/home/jan/opt/play-2.2.2/play run
```

Access the server at http://isachy.cz:9000/
1st access will take about 1 minutes for the source files to be compiled.
There will be error.

## Optimize images

Install https://github.com/RazrFalcon/SVGCleaner and run:

```
bin/svg-optimize
```

Or manually add chess piece images:

```
cd /home/jan/opt/lila/public/images
ln -s piece-src piece
ln -s wN-bg.src.svg wN-bg.svg
```

## Optimize JS

### Install Closure

https://code.google.com/p/closure-compiler/

Download compiler-latest.zip and create this script:

```
#!/bin/sh

if [ -h $0 ]
then
  ROOT_DIR="$(cd "$(dirname "$(readlink -n "$0")")" && pwd)"
else
  ROOT_DIR="$(cd "$(dirname $0)" && pwd)"
fi
cd "$ROOT_DIR"

java -jar compiler.jar $@
```

### Run Closure to optimize JS

```
bin/closure
```

A quick dirty way:

```
cd /home/jan/opt/lila/public/vendor/pgn4web
ln -s pgn4web.js pgn4web-compacted.js
```

## Fix domains

```
cp conf/application.conf.main conf/application.conf
```

See the port in application.conf:

```
include "base"

http {
  port = 80
}
```

Modify base.conf:

```
net {
  #domain = "lichess.org"
  domain = "en.isachy.cz"
```

Modify /etc/hosts on your local machine (Yes! On your machine, not the server
isachy.cz, because of the way Lila redirects. 37.59.2.183 is the
IP address of isachy.cz. This problem can be fixed by modifying
the source code of Lila or using the domain that we control.)

```
37.59.2.183 en.isachy.cz
37.59.2.183 socket.en.isachy.cz
37.59.2.183 static.isachy.cz
37.59.2.183 en.static.isachy.cz
```

## Fix domain when develop on local machine

Modify big.js:

```
//SetImagePath('http://' + document.domain.replace(/^\w+/, 'static') + "/assets/images/piece");
SetImagePath("/assets/images/piece");
//var baseUrl = 'http://' + document.domain.replace(/^\w+/, 'static') + '/assets/sound/';
var baseUrl = '/assets/sound/';
```

Modify boardEditor.js:

```
//var pieceTheme = 'http://' + document.domain.replace(/^\w+/, 'static') + '/assets/images/piece/{piece}.svg';
var pieceTheme = '/assets/images/piece/{piece}.svg’;
```

## Geo-IP

https://github.com/snowplow/scala-maxmind-geoip

TODO: Where to put Geo-IP data file?

## Run Lila

Rerun Lila in production mode (need sudo if use port 80):

```
cd /home/jan/opt/lila
sudo /home/jan/opt/play-2.2.2/play “run 80”
```

(See: http://stackoverflow.com/questions/8205067/how-do-i-change-the-default-port-9000-that-play-uses-when-i-execute-the-run)

From browser, go to:
http://en.isachy.cz/

## Create public/trans/*.js

Use the password of the user `cli.username` in conf/base.conf (you may want to
change the username and restart the server):

```
curl -X POST -v -d 'password=<the password above>&command=i18n js dump' http://en.isachy.cz/cli
```
