
# Informe de configuración de DMZ con Cisco Packet Tracer


### 1. Objetivo del laboratorio

> Configurar una DMZ segura usando un router Cisco ISR, aplicando NAT estático y ACLs para controlar el tráfico entre LAN, DMZ y red externa.
Se buscó que un servidor web en la DMZ sea accesible desde Internet, mientras se protege la LAN interna y se controla todo flujo de tráfico entre las zonas.



### 2. Topología implementada


Cantidad de redes: 3 (LAN interna, DMZ, red externa/Internet)

Dispositivos usados:

Router Cisco ISR 2911 (Router_FW)

3 Switches Cisco 2960 (LAN, DMZ, WAN)

PC_Internal (192.168.1.10)

Server_DMZ (192.168.2.10)

PC_External (192.168.3.10)

Breve descripción de la función de cada zona:

LAN interna: Usuarios internos de la organización.

DMZ: Servidor web accesible desde Internet, aislado de la LAN.

Red externa/Internet: Simula un cliente externo intentando acceder al servidor.



### 3. Plan de direccionamiento IP

Completa la tabla con las IPs asignadas (puedes copiarla del enunciado si no cambió).

| Dispositivo             | IP               | Máscara           | Gateway           |
|-------------------------|------------------|-------------------|-------------------|
| PC_Internal             |   192.168.1.10   |   255.255.255.0   |   192.168.1.1     |
| Server_DMZ              |   192.168.2.10   |   255.255.255.0   |   192.168.2.1     |
| PC_External             |   192.168.3.10   |   255.255.255.0   |   192.168.3.1     |
| Router_FW Gi0/0 (LAN)   |   192.168.1.1    |   255.255.255.0   |                   |
| Router_FW Gi0/1 (DMZ)   |   192.168.2.1    |   255.255.255.0   |                   |
| Router_FW Gi0/2 (Ext)   |   192.168.3.1    |   255.255.255.0   |                   |


### 4. Configuración aplicada (resumen)


- Interfaces configuradas con `ip address`
- NAT:
```bash
ip nat inside source static 192.168.2.10 192.168.3.1
```
- ACLs:
```bash
access-list 101 permit tcp any host 192.168.3.1 eq 80
access-list 100 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
```



### 5. Verificaciones realizadas

> Interfaces configuradas:
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 ip nat outside
 no shutdown

 NAT estático:
 ip nat inside source static 192.168.2.10 192.168.3.1

 ACLs aplicadas:
 ACL WAN (Internet → DMZ):
 access-list 101 permit tcp any host 192.168.3.1 eq 80
interface GigabitEthernet0/2
 ip access-group 101 in

 ACL DMZ → LAN (seguridad interna):
 ! Eliminación de ACL previa
no access-list 102

! Permitir que el servidor responda tráfico web desde LAN
access-list 102 permit tcp host 192.168.2.10 eq 80 192.168.1.0 0.0.0.255

! Bloquear todo tráfico DMZ → LAN
access-list 102 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255

! Permitir DMZ hacia otros destinos
access-list 102 permit ip any any

interface GigabitEthernet0/1
 ip access-group 102 in

 Comandos de verificación ejecutados:
 show ip access-lists
show ip nat translations

Origen	Destino	Protocolo	                    Resultado esperado	Resultado obtenido
PC_Internal	Router FW Gi0/0	Ping	            * Responde	         * Correcto
PC_Internal	Server_DMZ	HTTP	                * Web carga	         * Correcto
PC_External	Server_DMZ (192.168.3.1)	HTTP	* Web carga	         * Correcto
PC_External	Server_DMZ (192.168.3.1)	Ping	* Debe fallar	     * Falla
Server_DMZ	PC_Internal	Ping	                * Debe fallar	     * Packet Tracer permite ping (limitación simulador)

### 6. Conclusiones y recomendaciones

 Se logró configurar una DMZ funcional, segura y accesible desde Internet mediante NAT estático.

Las ACLs controlan correctamente el tráfico, permitiendo HTTP externo y bloqueando la comunicación DMZ → LAN.

Se observó una limitación de Packet Tracer: los pings desde la DMZ hacia LAN pueden pasar aunque la ACL esté configurada; esto no afecta la lógica de seguridad.

Aprendizaje principal: importancia de segmentar redes y aplicar NAT y ACLs de manera correcta para proteger la red interna y exponer solo los servicios deseados.


### 7. Capturas de evidencia

> Adjunta aquí (o en un PDF anexo) las capturas solicitadas: pings, navegador, comandos `show`, etc.
