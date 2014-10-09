---
layout: post
title: "a test post"
date: 2014-10-09 13:09:19 +0300
comments: true
categories: [test,new,experiment] 
---
``` bash postgresql setup
sudo pacman -S postgresql
sudo -i -u postgres
initdb --locale en_US.UTF-8 -E UTF8 -D '/var/lib/postgres/data'
exit
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -i -u postgres
createuser ck -P -S -R -D
createdb -O ck msf
```
{% blockquote ck %}
I shall be a denizen of the undeground or a successfull man of the world.
There shall be no compromise!
{% endblockquote %}
