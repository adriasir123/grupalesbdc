---
title: "Ejercicio 2: Router-nat (vagrant)"
---

# ENUNCIADO

![](https://fp.josedomingo.org/sri2122/u01/img/router.png)

El escenario a crear con Vagrant debe ser el siguiente:

- **Router**: red pública (conectada a br0) + red privada (veryisolated). IP 10.0.0.1 en red privada.

- **Cliente**: interfaz en *la misma* red privada que router, IP 10.0.0.2.



# REALIZACIÓN

## Configuración de la red pública KVM

Creación del bridge, y enlace de la interfaz física al mismo
```
sudo ip link add br0 type bridge
sudo ip link set enp4s0 master br0
```

Cambios en `/etc/network/interfaces`
```
###
# Changes due to br0 public network config
###

# Specify that the physical interface that should be connected to the bridge
# should be configured manually, to avoid conflicts with NetworkManager

iface enp4s0 inet manual

# The br0 bridge settings
iface br0 inet dhcp
   bridge_ports enp4s0
```

Levantamos el bridge
```
sudo ifup br0
```

Estos son los cambios necesarios en el host.  
En el Vagrantfile añadimos lo siguiente para que "Router" tenga una interfaz conectada a nuestro bridge (creando así la interfaz pública)
```
router.vm.network :public_network,
  :dev => "br0",
  :mode => "bridge",
  :type => "bridge"
```

Después de hacer todo esto, deberíamos tener una interfaz en "Router" con IP de nuestra red real.  
Si no consiguiéramos tener IP de la red real, tendríamos que hacer lo siguiente...

- En vagrant (una de las 2 opciones):
```
vagrant destroy router
vagrant reload router
```
- En nuestro host
```
sudo systemctl restart networking
```



# ENTREGA

## Vagrantfile

```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :router do |router|
    router.vm.box = "debian/bullseye64"
    router.vm.hostname = "router"
    router.vm.network :public_network,
      :dev => "br0",
      :mode => "bridge",
      :type => "bridge"
    router.vm.network :private_network,
      :libvirt__network_name => "interna",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.1",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "interna",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__forward_mode => "veryisolated"
  end

end
```

## Screenshot con ping cliente -> router
![](https://i.postimg.cc/y8kRjqND/Screenshot-from-2021-10-03-13-58-55.png)

Si queremos estar *incluso más seguros* de que el enlace funciona como debería, hacemos lo siguiente
![](https://i.postimg.cc/1z4fPDF4/Screenshot-from-2021-10-03-14-02-39.png)

Así nos aseguramos de que llegamos al destino por donde debería, y no siguiendo un camino alternativo.
