# Parcial 3 Sistemas Distribuidos
### Autor: Nicolás Machado Sáenz
### Tema: Descubrimiento de servicios con Consul-Template

Esta actividad tiene como objetivo implementar el Descubrimiento de Servicios sonre una infraestructura
automatizada de contenedores. Esta herramienta permite llevar un registro continuo de cada aplicación
que resida en un contenedor, así como los servicios nuevos y descontinuados de la infraestructura. Para
el ejercicio, el descubridor residirá en otro contenedor y utilizará la aplicación Consul.

Para este ejercicio, se utilizaron los siguientes recursos:
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
