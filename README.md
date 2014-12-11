spec-dockerbox
==============

spec-dockerbox is a small prototype to ensure that docker images and containers on a host adhere to given conditions, i.e.
  * "all my images should be 64 bit"
  * "no container should run as user root"
  * "containers matching /db/ should define Environment entry xyz"
  * "no container should have added capabilities"
  * "containers with names matching /nginx/ should only expose ports 80 and 443"
  * "all images should define an explicit entrypoint"
  * "allowed run users for containers are www-data,tomcat,xyz"
  * and so on.

Main purpose is to define architecture and security specifications for a container-based environment.

It works by examining `docker inspect` with filter options matching the above criteria.  

License 
-------

The MIT License (MIT) Copyright (c) 2014 Andreas Schmidt