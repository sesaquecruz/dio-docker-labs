SHELL = /bin/bash

cluster-up:
	vagrant up
	
cluster-down:
	vagrant destroy -f

swarm-up:
	docker swarm init | grep "docker swarm join --token" > token
	cut -c 117-129 token > ip
	vagrant ssh worker-1 -- -t "`cat token`; exit"
	vagrant ssh worker-2 -- -t "`cat token`; exit"
	vagrant ssh worker-3 -- -t "`cat token`; exit"

swarm-down:
	vagrant ssh worker-1 -- -t "docker swarm leave; exit"
	vagrant ssh worker-2 -- -t "docker swarm leave; exit"
	vagrant ssh worker-3 -- -t "docker swarm leave; exit"
	docker swarm leave --force
	rm token

web-server-up:
	docker node update --availability pause `hostname`
	docker volume create app
	docker service create --name web-server --replicas 3 -dt -p 80:80 --mount type=volume,src=app,dst=/var/www/html php:apache

web-server-down:
	docker service rm web-server
	docker volume rm app

nfs-server-up:
	echo "`pwd`/app *(rw,sync,subtree_check)" > /etc/exports
	exportfs -ar

nfs-server-down:
	echo "" > /etc/exports
	exportfs -ar

nfs-clients-up:
	vagrant ssh worker-1 -- -t "sudo mount `cat ip`:`pwd`/app /var/lib/docker/volumes/app/_data; exit"
	vagrant ssh worker-2 -- -t "sudo mount `cat ip`:`pwd`/app /var/lib/docker/volumes/app/_data; exit"
	vagrant ssh worker-3 -- -t "sudo mount `cat ip`:`pwd`/app /var/lib/docker/volumes/app/_data; exit"

nfs-clients-down:
	vagrant ssh worker-1 -- -t "sudo umount -f -l /var/lib/docker/volumes/app/_data; exit"
	vagrant ssh worker-2 -- -t "sudo umount -f -l /var/lib/docker/volumes/app/_data; exit"
	vagrant ssh worker-3 -- -t "sudo umount -f -l /var/lib/docker/volumes/app/_data; exit"
	rm ip