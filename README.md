# Parcial 3 Sistemas Distribuidos
### Autor: Nicolás Machado Sáenz
### Tema: Descubrimiento de servicios con Consul-Template

Esta actividad tiene como objetivo implementar el Descubrimiento de Servicios sobre una infraestructura
automatizada de contenedores. Esta herramienta permite llevar un registro continuo de cada aplicación
que resida en un contenedor, así como los servicios nuevos y descontinuados de la infraestructura. Para
el ejercicio, el descubridor residirá en otro contenedor y utilizará la aplicación Consul.

Se utilizaron los siguientes recursos:
  * Equipo anfitrión Ubuntu 16.04
  * Haproxy Load Balancer
  * Discovery Service Server (en un container)
  * Registrator Server
  * Tools: Docker, Docker Compose, Consul Template.
  * Portainer para el monitoreo de containers via Web.
  
Primero debemos asegurarnos de contar con todas las imágenes prerrequisitos para el deployment, como el 
load balancer Haproxy. Luego, dentro de la estructura de directorios, debemos ubicarnos en la carpeta con
el Docker-Compose y construir los containers. Si las imágenes no existen, se realiza un ```docker pull``` de
manera automática.

```bash
cd /home/distribuidos/Documents/talleres/sd-exam3/
docker-compose --build -d
```

El archivo yml del Docker Compose se muestra como sigue.

```yml
version: "2"

services:
  consul:
    container_name: consul
    image: consul:latest
    ports:
      - "8300:8300"
      - "8400:8400"
      - "8500:8500"
    restart: always
  registrator:
    container_name: registrator
    image: gliderlabs/registrator:latest
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock"
    command: consul://consul:8500
    restart: always
    depends_on:
      - consul
  web:
    image: deis/mock-http-server
    ports:
      - 8080
    environment:
      SERVICE_8080_NAME: "web"
      SERVICE_8080_TAGS: "web"
    restart: always
    depends_on:
      - registrator
  haproxy:
    build: ./haproxy
    container_name: haproxy
    image: haproxy:latest
    ports:
      - 80
    depends_on:
      - web

networks:
  default:
```

El Docker Compose invoca el siguiente archivo Dockerfile con la imagen Haproxy, ...

```Dockerfile
FROM haproxy:alpine

ENV CONSUL_TEMPLATE_VERSION=0.16.0

# Update wget to get support for SSL
RUN apk --update add haproxy wget

# Download consul-template
RUN ( wget --no-check-certificate https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip -O /tmp/consul_template.zip && unzip /tmp/consul_template.zip && mv consul-template /usr/bin && rm -rf /tmp/* )

COPY files/haproxy.json /tmp/haproxy.json
COPY files/haproxy.ctmpl /tmp/haproxy.ctmpl

ENTRYPOINT ["consul-template","-config=/tmp/haproxy.json"]
CMD ["-consul=172.26.0.2:8500"]
```

... que utiliza, entre otros scripts, la siguiente plantilla para aprovisionar el balanceador.

```json
template {
  source = "/tmp/haproxy.ctmpl"
  destination = "/etc/haproxy/haproxy.cfg"
  command = "haproxy -f /etc/haproxy/haproxy.cfg -sf $(pidof haproxy) &"
}
```

En Portainer, podemos evidenciar que efectivamente los contenedores están corriendo. Se ubican en una red
NAT provista por un adaptador virtual de Docker, con segmento de red 172.26.0.0/16, mientras que Portainer
cuenta con una dirección IP 172.17.0.2 en otro adaptador virtual. En ese sentido, si se quiere ver la app
del container web, debe ingresarse la IP del host anfitrión (192.168.130.130) seguida de un puerto mapeado.
Desde un equipo 192.168.130.190, esto se ve en el monitoreo de los servicios.

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-01.png)

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-02.png)

Y usando el puerto de cualquier contenedor web ("192.168.130.130:container_port"), este es el resultado:

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-03.png)

En Consul, también es posible escalar un servicio deseado a N contenedores. A pesar que este escalamiento es
manual, el balanceador se actualiza automáticamente, gracias al consul-template que realiza un reload cuando
se "levanta" o se "tumba" un servicio. Este consul-template es actualizado con la plantilla JSON.

Para este caso, vamos a "levantar" 75 contenedores web, con el siguiente comando.

```bash
docker-compose scale web=75
```

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-04.png)

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-05.png)

![Evidencia](https://github.com/MrNickOS/sd-exam3/blob/A00052208/sd-exam3-06.png)

Si queremos "tumbar" N contenedores de los existentes, basta especificar un número menor al ingresado en
el comando previo, pues Docker Compose y Consul Template se encargan de detener los contenedores.

```bash
docker-compose scale web=15
```

URL: https://github.com/MrNickOS/sd-exam3/tree/A00052208
