
# 🎓 Campus Network Infrastructure - JNA University

Este repositorio contiene la arquitectura de red y configuraciones de nivel de ingeniería para un campus universitario a gran escala. El proyecto implementa redundancia de Capa 2, enrutamiento dinámico de Capa 3 y un filtrado granular de seguridad mediante ACLs.

## 🚀 Características Técnicas

* **Segmentación Avanzada:** 9 VLANs para aislar departamentos críticos (Finanzas, IT, Servidores).
* **Alta Disponibilidad:** Configuración de EtherChannel (LACP) para redundancia de enlaces.
* **Enrutamiento Dinámico:** Implementación de **EIGRP AS 100** para una convergencia rápida.
* **Seguridad Perimetral e Interna:** Listas de Control de Acceso (ACL) extendidas aplicadas en las interfaces SVI.
* **Servicios Centralizados:** DHCP Relay (`ip helper-address`) y sincronización horaria vía **NTP**.

---

## 🗺️ Topología de Red

La red sigue un modelo jerárquico de distribución:

1. **Core/Distribution:** 3 Switches Multicapa (DSW1, DSW2, DSW3).
2. **Access Layer:** 5 Switches (SW1 - SW5) conectados vía Trunks y Port-Channels.
3. **Edge:** 2 Routers (Rt1, Rt2) para salida a servicios externos y WAN.

---

## 🛠️ Detalle de Implementación

### Esquema de Direccionamiento (VLAN Matrix)

| VLAN | Departamento | Subred | Gateway |
| --- | --- | --- | --- |
| 10 | Admission | 172.16.1.0/24 | 172.16.1.1 |
| 20 | IT | 172.16.2.0/24 | 172.16.2.1 |
| 50 | Finance | 172.16.5.0/24 | 172.16.5.1 |
| 100 | Servers | 172.16.100.0/24 | 172.16.100.1 |

### Lógica de Seguridad (Access Control Lists)

Se aplicó una política de **mínimo privilegio**. Ejemplo de la lógica en la VLAN de Admisiones:

* **Permitido:** DNS, HTTP/S hacia Internet y servidores internos, comunicación con IT y Finanzas.
* **Denegado:** Acceso a la red de Mantenimiento y protocolos no autorizados.

```bash
# Ejemplo de implementación de seguridad en DSW1
ip access-list extended admission-acl
 permit udp any eq bootpc host 255.255.255.255 eq bootps
 permit tcp 172.16.1.0 0.0.0.255 172.16.100.0 0.0.0.255 eq www
 permit ip 172.16.1.0 0.0.0.255 172.16.2.0 0.0.0.255

```

### Servicios de Red (NTP & DHCP)

* **NTP:** Se configuró el servidor `172.16.100.10` como fuente de tiempo autoritativa para asegurar que los logs de todos los dispositivos estén sincronizados.
* **DHCP Relay:** Los Switches de Distribución actúan como agentes relay para que los clientes en VLANs remotas obtengan direccionamiento del servidor central.

---

## 🔐 Gestión de Acceso

El acceso administrativo está protegido mediante **Autenticación Jerárquica**:

* **Nivel de Usuario (Privilege 1):** Acceso inicial limitado para monitoreo.
* **Nivel Privilegiado (Privilege 15/Enable):** Protegido por una segunda clave robusta para cambios de configuración global.

---

## 📂 Estructura del Repositorio

* `/configs`: Archivos `.txt` con las configuraciones completas de Routers y Switches.
* `/diagrams`: Diagrama de topología en formato imagen o PDF.
* `/scripts`: Scripts básicos de automatización (si aplica).

---

## 🛠️  Conmutación de Capa 2 (Switching)

* **VLAN Segmentation:** División de dominios de difusión mediante 9 VLANs para aislar departamentos (Admisiones, IT, Finanzas, HR, etc.).
* **802.1Q Trunking:** Configuración de enlaces troncales para el transporte eficiente de etiquetas de VLAN entre switches.
* **EtherChannel (LACP - 802.3ad):** Agregación de enlaces lógicos (Port-Channels) para maximizar el ancho de banda y proporcionar redundancia crítica ante fallas de enlaces físicos.
* **Rapid Spanning Tree (PVST+):** Optimización de la topología lógica para evitar bucles de red en enlaces redundantes.

### Enrutamiento de Capa 3 (Routing)

* **Inter-VLAN Routing:** Implementación de Interfaces Virtuales de Switch (SVI) en la capa de distribución como Gateways predeterminados.
* **EIGRP AS 100:** Protocolo de enrutamiento dinámico configurado con máscaras de wildcard y `no auto-summary` para una convergencia ultra rápida y soporte de VLSM.
* **Punto a Punto (/30):** Direccionamiento optimizado en enlaces de infraestructura para conservar el espacio de direcciones IP.

### Servicios de Red y Seguridad (IP Services & Security)

* **Extended ACLs:** Listas de control de acceso granulares aplicadas en la capa de distribución para filtrar tráfico basado en protocolos (TCP/UDP) y puertos específicos (HTTP, HTTPS, DNS, DHCP).
* **DHCP Relay Agent:** Uso de `ip helper-address` para centralizar la asignación de IPs desde un servidor dedicado en la VLAN 100.
* **NTP Synchronization:** Sincronización horaria en todos los nodos de la red para la coherencia en los logs de auditoría.
* **Hierarchical Access:** Seguridad de acceso administrativo mediante niveles de privilegio (User vs. Enable) y base de datos de usuarios locales.

---

## 🔒 Lógica de Seguridad (ACL)

Se implementó un modelo de **Zero Trust Parcial**. A continuación, un ejemplo de la configuración técnica en el switch central (DSW1):

```bash
! Permisos para la VLAN de Admisiones
ip access-list extended admission-acl
 permit udp any eq bootpc host 255.255.255.255 eq bootps   ! DHCP
 permit udp any 172.16.100.0 0.0.0.255 eq domain          ! DNS
 permit tcp 172.16.1.0 0.0.0.255 any eq 443               ! HTTPS a Internet
 permit ip 172.16.1.0 0.0.0.255 172.16.2.0 0.0.0.255      ! Comunicación con IT
 deny ip 172.16.1.0 0.0.0.255 172.16.8.0 0.0.0.255        ! Deny Mantenimiento

```

---

## 🧪 Verificación y Troubleshooting

Para validar la correcta operación, se utilizaron los siguientes comandos de diagnóstico:

* `show ip eigrp neighbors`: Verificar adyacencias de enrutamiento.
* `show etherchannel summary`: Validar el estado de los enlaces agregados.
* `show ip access-lists`: Monitorear el conteo de paquetes permitidos/denegados.
* `show ntp status`: Confirmar sincronización con el servidor central.

---
