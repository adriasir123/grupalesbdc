# Ejercicio 1: Instalación y configuración DHCP

## ENTREGA

### Parte 1
> Entrega la configuración del servidor DHCP.

### Parte 2
> Muestra la lista de concesiones en el servidor.

### Parte 3
> Muestra la modificación en la configuración que has hecho en el cliente para que tome la configuración de forma automática.

### Parte 4
> Muestra la salida del comando ` ip address` en el cliente.

### Parte 5
> Muestra una comprobación de que el cliente tiene resolución DNS.








## Escenario

- Servidor DHCP
- Cliente

Vagrantfile:
```
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define :servidor do |servidor|
    servidor.vm.box = "debian/bullseye64"
    servidor.vm.hostname = "servidor"
    servidor.vm.network :private_network,
      :libvirt__network_name => "ej1-dhcp",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente do |cliente|
    cliente.vm.box = "debian/bullseye64"
    cliente.vm.hostname = "cliente"
    cliente.vm.network :private_network,
      :libvirt__network_name => "ej1-dhcp",
      :libvirt__dhcp_enabled => false,
      :libvirt__forward_mode => "veryisolated"
  end

end
```


## Realización

### Parte 1
> Configuración de `isc-dhcp-server`

```
sudo apt update
sudo apt install isc-dhcp-server
```

En `/etc/default/isc-dhcp-server`:
- Descomentar la línea: `DHCPDv4_CONF=/etc/dhcp/dhcpd.conf`
- Especificar por qué interfaz escuchará y servirá IPs: `INTERFACESv4="eth1"`  

En `/etc/dhcp/dhcpd.conf`:
```
subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.100 192.168.0.110;
  option subnet-mask 255.255.255.0;
  option routers 192.168.0.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  max-lease-time 3600;
}
```










### Parte 2
> Configura los clientes para obtener direccionamiento dinámico. Comprueba las configuraciones de red que han tomado los clientes.
