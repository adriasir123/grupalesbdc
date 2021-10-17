# EJ3: Configuración del escenario router-nat

## ENTREGA

### Parte 1
> Desde cliente hacer traceroute a nombre de una web

```
vagrant@cliente:~$ traceroute www.example.org
traceroute to www.example.org (93.184.216.34), 30 hops max, 60 byte packets
 1  10.0.0.1 (10.0.0.1)  0.542 ms  0.459 ms  0.499 ms
 2  Livebox (192.168.1.1)  14.445 ms  14.415 ms  16.238 ms
 3  90.163.182.30 (90.163.182.30)  22.066 ms  22.038 ms  24.943 ms
 4  10.255.120.1 (10.255.120.1)  33.108 ms  33.069 ms  29.791 ms
 5  10.34.142.58 (10.34.142.58)  32.956 ms 10.34.142.62 (10.34.142.62)  40.640 ms  40.567 ms
 6  10.34.149.130 (10.34.149.130)  45.527 ms  17.344 ms 10.34.149.134 (10.34.149.134)  19.203 ms
 7  10.34.206.61 (10.34.206.61)  17.103 ms  25.263 ms  25.178 ms
 8  bundle-ether101-14.madtr6.madrid.opentransit.net (193.251.249.1)  32.250 ms bundle-ether104-14.madtr5.madrid.opentransit.net (193.251.247.13)  37.302 ms  32.108 ms
 9  telia-8.gw.opentransit.net (193.251.150.76)  32.050 ms
 193.251.129.16 (193.251.129.16)  39.332 ms telia-8.gw.opentransit.net (193.251.150.76)  37.063 ms
10  prs-bb2-link.ip.twelve99.net (62.115.123.220)  131.479 ms  135.389 ms  137.604 ms
11  prs-bb2-link.ip.twelve99.net (62.115.123.220)  139.376 ms  143.863 ms ash-bb2-link.ip.twelve99.net (62.115.112.242)  120.468 ms
12  rest-bb1-link.ip.twelve99.net (62.115.122.159)  112.756 ms ash-b2-link.ip.twelve99.net (62.115.123.125)  135.059 ms ash-b2-link.ip.twelve99.net (62.115.123.123)  114.004 ms
13  ash-b2-link.ip.twelve99.net (62.115.123.125)  125.977 ms ash-b2-link.ip.twelve99.net (62.115.123.123)  118.836 ms verizon-ic342246-ash-b2.ip.twelve99-cust.net (62.115.175.71)  125.925 ms
14  verizon-ic-315152-ash-b1.ip.twelve99-cust.net (213.248.83.119)  128.754 ms  128.715 ms ae-66.core1.dcb.edgecastcdn.net (152.195.65.129)  128.692 ms
15  ae-65.core1.dcb.edgecastcdn.net (152.195.64.129)  135.092 ms ae-66.core1.dcb.edgecastcdn.net (152.195.65.129)  134.537 ms  128.926 ms
16  www.example.org (93.184.216.34)  134.311 ms  146.229 ms  139.666 ms
```

### Parte 2
> Screenshot accediendo por ssh a las dos máquinas con las condiciones:  
- Nunca por eth0
- Configurar las conexiones directas en ~/.ssh/config

Elijo la manera de acceso añadiendo mi clave ssh pública personal a ambas máquinas.

He escrito lo siguiente en `~/.ssh/config`
```
# Conexiones ej3 config escenario router nat
Host router
    HostName 192.168.1.132
    User vagrant

Host cliente
  HostName 10.0.0.2
  User vagrant
  ProxyJump router
```

Accedo a router:  
![](https://i.imgur.com/roybChA.png)

Accedo a cliente:  
![](https://i.imgur.com/VbWlczl.png)

### Parte 3
> Integrar el playbook de ansible en vagrant. Enseñar funcionamiento a profesor.

En el Vagrantfile, he añadido lo siguiente:
```
config.vm.provision "ansible" do |ansible|
  ansible.playbook = "/home/atlas/Desktop/servicios/ud1-vagrant-ansible/ej3-config-escenario-router-nat/site.yaml"
end
```



## REALIZACIÓN
![](https://fp.josedomingo.org/sri2122/u01/img/router.png)

Crear playbook en ansible con los roles...

### Common
> Actualizar paquetes en ambos nodos

```
- name: Actualizando paquetes...
  become: yes
  apt: update_cache=yes upgrade=yes
```

### Router
> Convertir en router-nat y cambiar salida por eth1.  
Ambas configuraciones deben ser persistentes.

```
###
# Parte router-nat
###

# Módulo iptable_nat

- name: Cargando el módulo iptable_nat...
  modprobe:
    name: iptable_nat
    state: present

- name: Haciendo iptable_nat persistente...
  lineinfile:
    path: /etc/modules
    line: iptable_nat


# Forwarding
#
# Necesario instalar plugin:
# ansible-galaxy collection install ansible.posix

- name: Habilitando ipv4 forwarding (persistente)...
  ansible.posix.sysctl:
    name: net.ipv4.ip_forward
    value: '1'
    sysctl_set: yes

# Esta acción añade un 1 al fichero
# /proc/sys/net/ipv4/ip_forward
#
# También modifica el fichero
# /etc/sysctl.conf
#
# y así los cambios son persistentes


# Iptables

- name: Añadiendo regla iptable SNAT...
  command: iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth1 -j MASQUERADE

- name: Instalando iptables-persistent...
  apt:
    name: iptables-persistent

# Las reglas que hubiera, se guardan en /etc/iptables/rules.v4
# Este fichero se carga automáticamente en boot



###
# Parte rutas
###

- name: Borrando ruta por defecto...
  command: ip route del default

- name: Añadiendo nueva ruta por defecto...
  command: ip route add default via 192.168.1.1

- name: Haciendo la ruta por defecto persistente...
  copy:
    src: interfaces-router
    dest: /etc/network/interfaces
    owner: root
    group: root
    mode: u=rw,g=r,o=r
```

### Cliente
> Cambiar salida por eth1 y que sea persistente

```
- name: Borrando ruta por defecto...
  command: ip route del default

- name: Añadiendo nueva ruta por defecto...
  command: ip route add default via 10.0.0.1

- name: Haciendo la ruta por defecto persistente...
  copy:
    src: interfaces-cliente
    dest: /etc/network/interfaces
    owner: root
    group: root
    mode: u=rw,g=r,o=r
```
