# Configurar Ubiquiti EdgeRouter para Movistar TV

Vamos a suponer que la LAN del HGU es la __192.168.1.0/24__ y que la del ER es __192.168.10.0/24__. Esta configuración debería servir para cualquier router EdgeMAX, concretamente este ejemplo se basa en un __ER-X__ con la WAN en __eth0__ y el resto de puertos configurados como un switch en __switch0__, como se hace en la configuración básica del Wizard.

> HGU = Router de Movistar, ER = EdgeRouter.

## Si pruebas esta configuración y tienes fallos, coméntalos como issues de este repositorio

### Escenario

![escenario](img/mapa.png)

### 1 - Asignar a la interfaz WAN física del ER una IP fija o por DHCP dentro de la red del HGU

![imagen 1-1](img/1-1.png)

### 2 - Enmascarar todo el tráfico hacia la red del HGU con dicha interfaz

![imagen 2-1](img/2-1.png)

### 3 - Añadir un par de rutas al ER para las peticiones IGMP/Multicast

Rutas estáticas de __tipo Gateway__.

![imagen 3-1](img/3-1.png)

No estoy seguro de que sean necesarias todas, pero son las que había en la tabla del router HGU, como se puede ver a continuación:

![imagen 3-2](img/3-2.png)

### 4 - Configurar el Firewall del ER

Me baso en las dos Rulesets definidas por la configuración básica del ER a través del Wizard, WAN_IN y WAN_LOCAL.

![imagen 4-1](img/4-1.png)

#### Reglas de WAN_IN:

Es importante que se __mantengan en el orden__ definido.

![imagen 4-2](img/4-2.png)

![imagen 4-3](img/4-3.png)

![imagen 4-4](img/4-4.png)

#### Reglas de WAN_LOCAL:

__Las mismas__ que las de WAN_IN, pero sospecho que no deberían de hacer falta.

> Necesita mas pruebas.

### 5 - [Habilitar IGMP Proxy en el ER](https://help.ui.com/hc/en-us/articles/204961854-EdgeRouter-IGMP-Proxy)

Nos conectamos a la __terminal__ del ER (por la CLI Web o por SSH).

```
configure
```

```
set protocols igmp-proxy interface eth0 role upstream
set protocols igmp-proxy interface switch0 role downstream
```

```
set protocols igmp-proxy interface eth0 threshold 1
set protocols igmp-proxy interface eth0 alt-subnet 0.0.0.0/0

set protocols igmp-proxy interface switch0 threshold 1
set protocols igmp-proxy interface switch0 alt-subnet 0.0.0.0/0
```

```
commit ; save
exit
```

#### Comandos para verificar que el tráfico multicast está fluyendo:

```
show ip multicast mfc
show ip multicast interfaces
```

### 6 - Configuración estática de red en el aparato de Movistar TV

- Pulsamos el botón superior con forma de persona en el mando de la TV de Movistar repetidas veces durante el arranque, hasta entrar en el __menú de configuración de red__.

![imagen 6-1](img/6-1.png)

- Editamos la configuración en modo estático:
    - Modo de red: __estático__
    - Dirección IP: __192.168.10.40__
    - Máscara de Subred: __255.255.255.0__
    - Pasarela (Gateway): __192.168.10.1__
    - Dirección IP del OPCH: __239.0.2.29:22222__
    - DNS Primario: __172.26.23.3__

### Enlaces en los que me he basado

- https://help.ui.com/hc/en-us/articles/204961854-EdgeRouter-IGMP-Proxy
- https://comunidad.movistar.es/t5/Soporte-Fibra-y-ADSL/Routers-ubiquiti-con-HGU-mitrastar/m-p/3441509#M313299
- https://comunidad.movistar.es/t5/Soporte-Fibra-y-ADSL/configuracion-del-router-ax-11000-asus-por-conexion-pppoe-ONT/m-p/4217145#M427122

## Si pruebas esta configuración y tienes fallos, coméntalos como issues de este repositorio
