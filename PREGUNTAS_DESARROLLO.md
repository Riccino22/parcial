# 20 Preguntas de Desarrollo — Repaso Parcial

> Preguntas para responder **desarrollando** (no son de opción múltiple).
> Temas: Firewall, Máscaras/Subredes, AND/OR, NAT/PAT y ACL.
> Intentá responderlas vos primero; al final está el **solucionario desarrollado** para autocorregirte.
> Todo el contenido sale de los documentos del curso.

---

## Parte A — Firewall

1. ¿Qué es un firewall y cuáles son sus **dos principios/objetivos fundamentales** de diseño?

2. Explicá **para qué sirve** y **para qué NO sirve** un firewall.

3. Describí las **cuatro estrategias de seguridad** (paranoica, prudente, permisiva, promiscua) e indicá cuál es el **paradigma de seguridad más usado**.

4. Explicá las principales **limitaciones** de un firewall (al menos cuatro).

5. Nombrá y describí brevemente los **7 tipos de firewall**.

6. ¿Qué es una **DMZ (Zona Desmilitarizada)**, en qué tipo de firewall se usa y qué logra?

7. En **iptables**, explicá la diferencia entre `ACCEPT`, `REJECT` y `DROP`, y describí las **3 tablas** (FILTER, NAT, MANGLE).

---

## Parte B — Máscaras, Subredes y operaciones AND/OR

8. Explicá qué es una **máscara de subred** y cómo se interpreta la **notación CIDR** (`/n`).

9. **Desarrollá el ejercicio:** dividir `192.168.1.0/24` en **4 subredes**. Indicá la nueva máscara, los hosts por subred, el salto y la tabla de subredes con su rango y broadcast.

10. **Desarrollá el ejercicio:** red `192.168.1.0` con máscara `255.255.255.224` (`/27`). Calculá subredes, hosts por subred, salto, y para las primeras subredes su dirección de red, primer host, último host y broadcast.

11. Explicá cómo se obtiene la **dirección de red** (con AND) y el **broadcast** (con OR), dando un ejemplo con `192.168.1.100` y máscara `/26`.

12. Explicá **por qué se resta 2** en la fórmula de hosts y cómo se calcula el **salto (block size)**.

13. Completá y explicá la **tabla CIDR de `/24` a `/30`** (máscara decimal, hosts útiles e IPs totales) y la relación entre cada fila.

---

## Parte C — NAT y PAT

14. ¿Qué **problema** resuelve NAT y cuál es su **función principal**? Nombrá los **rangos de direcciones privadas**.

15. Indicá las **ventajas** y las **limitaciones** de NAT.

16. Explicá la **diferencia entre NAT y PAT** y qué hace la palabra clave **`overload`**.

---

## Parte D — ACL (de red)

17. ¿Qué son las **ACL** y por qué es **crítico el orden** de las reglas? Dá un ejemplo **correcto** y uno **incorrecto**.

18. ¿Qué es una **ACL estándar** y qué es la **wildcard mask**? Explicá la diferencia entre **inbound** y **outbound**.

---

## Parte E — Integración

19. Explicá **cómo distingue las conexiones** un firewall (políticas por IP, filtrado de contenido, antimalware y DPI).

20. Escribí la configuración de una **ACL** que permita el acceso solo a la red `192.168.1.0/24` y bloquee el resto, **aplicada** a una interfaz; explicá cada línea.

---
---

# 🔑 Solucionario desarrollado

> Para autocorregir. Las respuestas siguen los documentos del curso.

### 1. Qué es y dos principios
Un **firewall** es un sistema ubicado **entre dos redes** que aplica una **política de seguridad**, protegiendo una **red confiable** de una **no confiable**. Sus **dos principios** son: **(1)** todo el tráfico (entrante y saliente) **debe pasar a través de él**; **(2)** **solo pasa el tráfico autorizado** según la política.

### 2. Para qué sirve / no sirve
- **Sí sirve:** defensa **perimetral** de la red.
- **No sirve:** contra ataques o errores que vienen del **interior** de la red, ni para proteger una vez que el intruso **ya lo traspasó**.

### 3. Estrategias y paradigma
- **Paranoica:** se controla todo, no se permite nada (la más segura).
- **Prudente:** se controla y se conoce todo.
- **Permisiva:** se controla pero se permite demasiado.
- **Promiscua:** no se controla y se permite todo (la menos segura).
- **Paradigma más usado y seguro:** *"prohibir todo excepto lo expresamente permitido"*.

### 4. Limitaciones
1. El **hueco que no se tapa** y que un intruso descubre. 2. **No es inteligente:** si un paquete no está definido como amenaza, lo deja pasar (back doors). 3. **No es contra humanos:** no detecta si alguien roba/difunde contraseñas. 4. **No protege contra virus** (hace falta antivirus). 5. **No protege del interior** de la red.

### 5. Los 7 tipos
1. **Filtrado de paquetes** (router con reglas por IP/puerto/protocolo; lo implementa el *Choke*). 2. **Proxy-Gateway de aplicación** (*Bastion Host*: proxy intermediario; deja pasar información, no paquetes). 3. **Dual-Homed Host** (2 interfaces; no enruta paquetes IP). 4. **Screened Host** (router + bastión; seguridad por filtrado). 5. **Screened Subnet** (aísla el bastión con una **DMZ** y 2 routers). 6. **Inspección de paquetes (SPI)** (filtrado dinámico, sigue el estado de las conexiones). 7. **Firewalls personales** (software en el PC del usuario).

### 6. DMZ
La **DMZ (Zona Desmilitarizada)** es una zona ubicada **entre el router exterior y el interior** que se usa en el **Screened Subnet**. **Aísla los servicios públicos** (WWW, FTP, SMTP, POP) de la red interna, de modo que **no exista conexión directa** entre la red interna y la externa, protegiendo así el nodo bastión (la máquina más expuesta).

### 7. iptables — acciones y tablas
- **ACCEPT:** acepta el paquete. **REJECT:** lo rechaza y **envía una respuesta** negativa. **DROP:** lo descarta y **no envía nada** (el atacante no sabe si la máquina existe). → *REJECT avisa, DROP calla.*
- **Tablas:** **FILTER** (filtrado de paquetes, implementa el firewall), **NAT** (masquerading: que otros salgan con nuestra IP), **MANGLE** (alterar el estado de un paquete, casos críticos).

### 8. Máscara y CIDR
La **máscara de subred** es un patrón de **32 bits** que distingue **qué bits de la IP son de red** y cuáles de **host**. En **CIDR**, `/n` indica la **cantidad de bits en 1** (de red) que tiene la máscara; ej. `255.255.255.0` = 24 unos = `/24`.

### 9. Ejercicio `192.168.1.0/24` en 4 subredes
- Bits necesarios: `2ⁿ ≥ 4` → `2² = 4` → **2 bits**.
- Nueva máscara: `/24 + 2` = **/26 = 255.255.255.192**.
- Hosts por subred: `2⁶ − 2` = **62**.
- Salto: `256 − 192` = **64**.

| Subred | Rango de hosts | Broadcast |
|--------|----------------|-----------|
| 192.168.1.0/26 | .1 – .62 | .63 |
| 192.168.1.64/26 | .65 – .126 | .127 |
| 192.168.1.128/26 | .129 – .190 | .191 |
| 192.168.1.192/26 | .193 – .254 | .255 |

### 10. Ejercicio `192.168.1.0` con `/27`
3 bits para subred, 5 bits para host.
- Subredes utilizables: `2³ − 2` = **6** *(con el criterio 2ⁿ serían 8 — confirmar con el profesor)*.
- Hosts por subred: `2⁵ − 2` = **30**.
- Salto: `256 − 224` = **32**.

| Subred | Red | Primer host | Último host | Broadcast |
|--------|-----|-------------|-------------|-----------|
| subred 1 | 192.168.1.32 | .33 | .62 | .63 |
| subred 2 | 192.168.1.64 | .65 | .94 | .95 |
| subred 3 | 192.168.1.96 | .97 | .126 | .127 |

### 11. AND → red, OR → broadcast
- **Dirección de red:** `IP AND máscara`. Ej.: `192.168.1.100` AND `255.255.255.192` = **192.168.1.64**.
- **Broadcast:** `IP OR (inverso de la máscara)`. Ej.: `192.168.1.100` OR `0.0.0.63` = **192.168.1.127**.

### 12. Por qué −2 y cómo es el salto
Se resta 2 porque en cada subred **una dirección es la de red** (la primera) y **otra es la de broadcast** (la última), y ninguna se asigna a un host. El **salto (block size)** se calcula como **`256 − (octeto de la máscara)`**; las subredes van avanzando de ese valor en ese valor.

### 13. Tabla CIDR /24 a /30
| CIDR | Máscara | Hosts útiles | IPs totales |
|------|---------|--------------|-------------|
| /24 | 255.255.255.0 | 254 | 256 |
| /25 | 255.255.255.128 | 126 | 128 |
| /26 | 255.255.255.192 | 62 | 64 |
| /27 | 255.255.255.224 | 30 | 32 |
| /28 | 255.255.255.240 | 14 | 16 |
| /29 | 255.255.255.248 | 6 | 8 |
| /30 | 255.255.255.252 | 2 | 4 |

Relación: por cada bit que se "apaga" a la derecha (al bajar de /30 a /24), las **IPs totales se duplican**; los hosts útiles son siempre **IPs totales − 2**.

### 14. NAT — problema, función y rangos privados
NAT resuelve el **agotamiento de direcciones IPv4**. Su **función** es traducir **IP privadas → IP públicas**. **Rangos privados:** `10.0.0.0/8`, `172.16.0.0 – 172.31.0.0` y `192.168.0.0/16`.

### 15. NAT — ventajas y limitaciones
- **Ventajas:** ahorra IP públicas, mejora la organización, **oculta parcialmente la red interna** y facilita la administración.
- **Limitaciones:** **no reemplaza al firewall**, puede **dificultar el diagnóstico** y **no evita el malware**.

### 16. NAT vs PAT y `overload`
**NAT** traduce direcciones privadas a públicas (típicamente 1 a 1). **PAT** usa **una sola IP pública compartida** y diferencia las conexiones por **puertos distintos** (analogía: hotel con habitaciones). La palabra clave **`overload`** es la que **permite múltiples conexiones sobre la misma IP pública**, es decir, lo que convierte la traducción en PAT.

### 17. ACL y orden de reglas
Las **ACL (Access Control List)** son conjuntos de **reglas que permiten o bloquean tráfico**; el dispositivo las revisa **en orden** y aplica la **primera coincidencia**. El **orden es crítico** porque una regla mal ubicada bloquea la red.
- **Correcto:** `permit 192.168.1.0 0.0.0.255` y luego `deny any`.
- **Incorrecto:** `deny any` primero → bloquea todo antes de llegar al `permit`.

### 18. ACL estándar, wildcard, inbound/outbound
- **ACL estándar:** filtra principalmente por la **IP de origen** (no mira puertos ni protocolos).
- **Wildcard mask:** indica **qué parte de la dirección puede variar**; es el **inverso** de la máscara (255.255.255.0 → 0.0.0.255).
- **Inbound:** filtra el tráfico **al entrar** a la interfaz; **Outbound:** **al salir**.

### 19. Cómo distingue conexiones un firewall
Mediante: **(a)** **políticas por IP** (suspende peticiones externas y oculta los recursos internos); **(b)** **filtrado de contenido** (distingue lo problemático y puede bloquear webs/servidores); **(c)** **servicios de antimalware** (definiciones de virus); **(d)** **DPI (Deep Packet Inspection)**, que revisa el contenido profundo del paquete como una **segunda capa de seguridad**.

### 20. ACL que permite solo `192.168.1.0/24`
```
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any
Router(config)# interface g0/0
Router(config-if)# ip access-group 10 in
```
- **Línea 1:** crea la ACL 10 y **permite** la red 192.168.1.0 (la wildcard `0.0.0.255` deja variar el último octeto).
- **Línea 2:** **niega** todo el resto del tráfico.
- **Línea 3:** entra a configurar la interfaz `g0/0`.
- **Línea 4:** **aplica** la ACL al tráfico **entrante** (`in`) de esa interfaz. *(Crear la ACL no alcanza: hay que aplicarla a una interfaz.)*

---

*Fuentes: documentos del curso (Firewall - Cortafuegos; Resumen Firewall - Tipos; Máscaras y Subredes; Crear subredes a partir de una máscara; Apoyo IP – Máscaras; ACL básicas - Port Security y NAT/PAT). Sin información externa.*
