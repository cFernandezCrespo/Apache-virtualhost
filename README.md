# Apache-virtualhost
Para esta practica a mayores de lo hecho hasta ahora hay que hacer los documentos httpd.conf mime.types y named.conf. de la siguiente forma los creas:
~~~
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > httpd.conf
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/mime.types > mime.types
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > named.conf
~~~
creados, se haran documentos que puedes modificar de la forma que desees en su configuración interna.
a continuación haremos nuestro docker compose:
~~~
# contenedor de apache
services:
  web:
    container_name: docker_web_1
    image: httpd:latest
    ports:
      - "80:80"
    volumes:
      - ./paginas:/usr/local/apache2/htdocs
      - ./conf:/usr/local/apache2/conf
    networks:
      red:
        ipv4_address: 192.168.1.2
  bind9:
        image: ubuntu/bind9
        container_name: asir_apache_dns
        ports:
          - "53:53/tcp"
          - "53:53/udp"
        networks:
          red:
            ipv4_address: 192.168.1.1
        volumes:
          - ./conf:/etc/bind
          - ./zonas:/var/lib/bind

  cliente:
    container_name: cliente_web
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    networks:
      red: 
        ipv4_address: 192.168.1.3
    dns:
      - 192.168.1.1

networks:
  red: 
    ipam:
      config:
        - subnet: 192.168.1.0/24
          gateway: 192.168.1.254
~~~

Aqui ya tendremos para lanzar nuestros 2 clientes y nuestro apache junto con la red, pero nos falta unos cuantos documentos mas, los named.conf.options y named.conf.default-zones seran iguales que en anteriores practicas, pero nuestro named.conf.local sera ligeramente distinto.
~~~

zone "fabulasmaravillosas.int" {
	type master;
	file "/var/lib/bind/db.fabulasmaravillosas.int";
	allow-query {
		any;
		};
	};

zone "fabulasoscuras.int" {
	type master;
	file "/var/lib/bind/db.fabulasoscuras.int";
	allow-query {
		any;
		};
	};
~~~

En este haremos que nuestros contenedores se redirijan a los documentos db.
hablando de los mismos tendremos que configurarlos, para hacer lo que pide la practica tendriamos que tener unos documentos tal que asi:

~~~
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.fabulasoscuras.int. some.email.address. (
				10003   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.fabulasoscuras.int.
ns		IN A		192.168.1.1
www		IN CNAME	ns
~~~

~~~
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.fabulasmaravillosas.int. some.email.address. (
				10003   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.fabulasmaravillosas.int.
ns		IN A		192.168.1.1
www		IN CNAME	ns
~~~

Con todo esto creado solo tendriamos que lanzar el docker compose up y ya estaria listo para hacer los digs.

