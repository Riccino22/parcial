# Repaso Parcial — Lo más probable que va

> Documento de estudio enfocado en los temas que (según lo indicado) tienen más chance de aparecer:
> **Firewall (sí o sí)**, **3 ejercicios de máscaras**, **operaciones AND y OR**, **NAT y PAT**, y **ACL** (posible).
> Todo el contenido sale de los documentos del curso. Donde los documentos se contradicen entre sí, lo aclaro explícitamente.

**Prioridad sugerida de estudio:**
1. 🔥 Firewall — *cae seguro*
2. 🧮 Máscaras (3 ejercicios) + AND/OR — *muy probable*
3. 🌐 NAT y PAT — *probable*
4. 🚦 ACL — *capaz algunas preguntas*

---

## 1. 🔥 FIREWALL / CORTAFUEGOS (cae sí o sí)

### ¿Qué es?
Un **Firewall** es un sistema ubicado **entre dos redes** que aplica una **política de seguridad**: protege una **red confiable** de una **red no confiable**.

### Los 2 principios fundamentales ⭐
1. **Todo el tráfico** (entrante y saliente) **debe pasar a través de él**.
2. **Solo pasa el tráfico autorizado** según la política de seguridad.

### Para qué SIRVE y para qué NO
| ✅ SIRVE para | ❌ NO SIRVE para |
|---------------|------------------|
| Defensa **perimetral** de la red | Ataques o errores que vienen del **interior** |
| | Proteger una vez que el intruso **ya lo traspasó** |

### Cómo puede ser
- **Tipo:** Físicos (hardware), Software, Mixtos.
- **Formato:** 1) Personales, 2) De subred, 3) Redes perimetrales.

### Políticas de diseño
- **¿Qué proteger?** Todos los elementos de la red interna (hardware, software, datos).
- **¿De quién?** De accesos no autorizados desde el exterior.
- **Paradigmas de seguridad:**
  - Permitir todo excepto lo prohibido.
  - **Prohibir todo excepto lo permitido** ← **el más usado y seguro** ⭐
- **Estrategias de seguridad** (de + a − segura):
  | Estrategia | Significado |
  |------------|-------------|
  | **Paranoica** | Se controla todo, no se permite nada (la más segura) |
  | **Prudente** | Se controla y se conoce todo |
  | **Permisiva** | Se controla pero se permite demasiado |
  | **Promiscua** | No se controla y se permite todo |

### Limitaciones ⭐ (pregunta clásica)
1. El **hueco que no se tapa** (que un intruso descubre con su ingenio).
2. **No es inteligente:** si un paquete no está definido como amenaza, lo deja pasar (riesgo de **back doors**).
3. **No es contra humanos:** no detecta si alguien roba o difunde contraseñas físicamente.
4. **No protege contra virus** en archivos (hace falta un antivirus).
5. **No protege del interior** de la red.

### Los 7 Tipos de Firewall ⭐
| # | Tipo | Idea clave |
|---|------|-----------|
| 1 | **Filtrado de paquetes** | Router con reglas; filtra por IP, puerto y protocolo. El dispositivo que lo implementa = **Choke**. Económico, rápido, transparente |
| 2 | **Proxy-Gateway de aplicación** | **Bastion Host**: intermediario (proxy); permite flujo de información, NO de paquetes; atiende un solo servidor |
| 3 | **Dual-Homed Host** | Conectado a ambas redes (2 interfaces); **NO deja pasar paquetes IP** (IP-Forwarding desactivado) |
| 4 | **Screened Host** | Router + host bastión; la seguridad principal viene del **filtrado de paquetes** |
| 5 | **Screened Subnet** | Aísla el **Nodo Bastión** (la máquina más atacada) con una **DMZ**, usando 2 routers (exterior + interior) |
| 6 | **Inspección de paquetes (SPI)** | Filtrado **dinámico**; sigue el **estado** de las conexiones; solo pasa lo que coincide con una conexión activa conocida |
| 7 | **Firewalls personales** | Software instalado en el PC del usuario final, protege ese único equipo |

**Detalles que suelen preguntar:**
- **Filtrado de paquetes:** analiza **protocolos, IP origen/destino, puerto TCP-UDP origen/destino y comandos de red**; usa solo las **cabeceras** (no el área de datos).
- **Bastion Host:** *Single-homed* (1 interfaz), *Dual-homed* (≥2 interfaces), *Multihomed* (interno).
- **DMZ (Zona Desmilitarizada):** zona entre router externo e interno que **aísla los servicios públicos** (WWW/FTP/SMTP/POP) de la red interna; no hay conexión directa interna↔externa.
- **SPI / cortafuegos de estado:** guarda en RAM el **estado de la conexión** = IPs + puertos + nº de secuencia.

### iptables / Netfilter ⭐⭐ (muy probable)
**Acciones (targets):**
| Acción | Qué hace |
|--------|----------|
| **ACCEPT** | Acepta el paquete |
| **REJECT** | Rechaza y **envía respuesta** negativa a la fuente |
| **DROP** | Descarta y **NO envía nada** (el atacante no sabe si la máquina existe) |

> 💡 Diferencia clave: **REJECT avisa, DROP calla.**

**Las 3 tablas:**
| Tabla | Función |
|-------|---------|
| **FILTER** | Filtrado de paquetes (implementa el firewall) |
| **NAT** | Masquerading: que otros salgan con nuestra IP |
| **MANGLE** | Alterar el estado de un paquete (casos críticos) |

**Cadenas (chains) de FILTER:** `INPUT` (entrante), `OUTPUT` (saliente), `FORWARD` (enrutar hacia otro equipo).

**Estados de conexión (`ipt_conntrack`):** `NEW`, `ESTABLISHED`, `RELATED` (nuevo pero ligado a una conexión existente, ej. FTP), `INVALID`.

**Órdenes básicas:**
| Comando | Acción |
|---------|--------|
| `iptables -F` | Borrar (flush) todas las reglas |
| `iptables -L` | Listar reglas aplicadas |
| `iptables -A` | Append: añadir regla (al final) |
| `iptables -D` | Borrar una regla |

**Anatomía de una regla:**
```
iptables -A INPUT -i eth0 -s 0.0.0.0/0 -p TCP --dport www -j ACCEPT
```
`-A INPUT` añade a la cadena INPUT · `-i eth0` interfaz de entrada · `-s` origen · `-p TCP` protocolo · `--dport www` puerto destino (80) · `-j ACCEPT` acción.

---

## 2. 🧮 MÁSCARAS — 3 EJERCICIOS (parecidos a los del documento)

### Fórmulas que necesitás ⭐
| Qué calcular | Fórmula |
|--------------|---------|
| Cantidad de subredes | **2ⁿ** (n = bits tomados para subred) |
| Hosts útiles por subred | **2ʰ − 2** (h = bits de host; −2 = red + broadcast) |
| Salto (block size) | **256 − (octeto de la máscara)** |

> ⚠️ **Aviso de los documentos:** para la cantidad de subredes hay **dos criterios** según el material:
> - El doc *“Crear subredes a partir de una máscara”* usa **2ⁿ**.
> - El doc *“Máscaras y subredes”* usa **2ⁿ − 2** (resta la primera y la última subred).
>
> **Confirmá con el profesor cuál criterio usar.** En los ejercicios de abajo indico ambos donde aplica.

---

### 📝 Ejercicio 1 — Dividir `192.168.1.0/24` en 4 subredes
*(ejemplo del documento "Crear subredes a partir de una máscara")*

| Paso | Cálculo | Resultado |
|------|---------|-----------|
| 1. Bits necesarios | `2ⁿ ≥ 4` → `2² = 4` | **2 bits** |
| 2. Nueva máscara | `/24 + 2` | **/26 = 255.255.255.192** |
| 3. Hosts por subred | quedan 6 bits → `2⁶ − 2` | **62 hosts** |
| 4. Salto (block size) | `256 − 192` | **64** |

**5. Subredes resultantes:**
| Subred | Rango de hosts | Broadcast |
|--------|----------------|-----------|
| 192.168.1.0/26 | .1 – .62 | .63 |
| 192.168.1.64/26 | .65 – .126 | .127 |
| 192.168.1.128/26 | .129 – .190 | .191 |
| 192.168.1.192/26 | .193 – .254 | .255 |

---

### 📝 Ejercicio 2 — Red `192.168.1.0` con máscara `255.255.255.224` (/27)
*(ejemplo guía del documento "Máscaras y subredes", que usa el criterio 2ⁿ − 2)*

Tomados **3 bits** para subred, quedan **5 bits** para host.

| Qué calcular | Cálculo | Resultado |
|--------------|---------|-----------|
| Subredes utilizables | `2³ − 2` | **6** *(con criterio 2ⁿ serían 8)* |
| Hosts útiles por subred | `2⁵ − 2` | **30** |
| Salto | `256 − 224` | **32** |

**Direcciones de subred (sumando 32):**
| Subred | Dirección de red |
|--------|------------------|
| subred 0 | 192.168.1.0 |
| subred 1 (1ª útil) | 192.168.1.32 |
| subred 2 | 192.168.1.64 |
| subred 3 | 192.168.1.96 |
| subred 4 | 192.168.1.128 |

**Detalle de una subred /27 (32 direcciones):**
- **.32** = dirección de red
- **.33** = primer host útil  *(red + 1)*
- **.62** = último host útil  *(broadcast − 1)*
- **.63** = broadcast  *(dirección de la subred siguiente − 1)*

---

### 📝 Ejercicio 3 — `10.0.0.0/16` con 500 hosts por subred
*(ejemplo avanzado "por hosts" del documento "Crear subredes a partir de una máscara")*

| Paso | Cálculo | Resultado |
|------|---------|-----------|
| 1. Bits de host necesarios | `2⁸ − 2 = 254` (poco) → `2⁹ − 2 = 510 ≥ 500` ✅ | **9 bits de host** |
| 2. Nueva máscara | `32 − 9` | **/23 = 255.255.254.0** |

> 💡 Truco: cuando piden **por hosts**, calculás `h` y la máscara sale de **32 − h**.

---

### Tabla rápida CIDR (memorizar) ⭐
| CIDR | Máscara | Hosts útiles | IPs totales |
|------|---------|--------------|-------------|
| /24 | 255.255.255.0 | 254 | 256 |
| /25 | 255.255.255.128 | 126 | 128 |
| /26 | 255.255.255.192 | 62 | 64 |
| /27 | 255.255.255.224 | 30 | 32 |
| /28 | 255.255.255.240 | 14 | 16 |
| /29 | 255.255.255.248 | 6 | 8 |
| /30 | 255.255.255.252 | 2 | 4 |

**Máscaras por defecto por clase:** A = 255.0.0.0 (/8) · B = 255.255.0.0 (/16) · C = 255.255.255.0 (/24).

---

## 3. ➕ OPERACIONES AND y OR

### Posición y valor de los bits (de un octeto)
| 2⁷ | 2⁶ | 2⁵ | 2⁴ | 2³ | 2² | 2¹ | 2⁰ |
|----|----|----|----|----|----|----|----|
| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

> `128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255` (todos los bits en 1).

### Tabla de verdad AND
| A | B | A AND B |
|---|---|:-------:|
| 1 | 1 | **1** |
| 0 | 1 | 0 |
| 0 | 0 | 0 |
| 1 | 0 | 0 |

> Regla: el AND da **1 solo si los dos bits son 1**.

### Tabla de verdad OR
| A | B | A OR B |
|---|---|:------:|
| 0 | 0 | **0** |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

> Regla: el OR da **0 solo si los dos bits son 0**.

### Para qué se usan ⭐
| Operación | Sirve para | Cómo |
|-----------|-----------|------|
| **AND** | Obtener la **dirección de RED** | `IP AND máscara` |
| **OR** | Obtener el **BROADCAST** | `IP OR (inverso de la máscara)` |

**Ejemplo AND (dirección de red):**
`192.168.1.100` AND `255.255.255.192` → **192.168.1.64**

**Ejemplo OR (broadcast):**
`192.168.1.100` OR `0.0.0.63` → **192.168.1.127**

> Para obtener la **máscara** a partir de red y broadcast: se comparan bit a bit → los bits que **coinciden** son **1** en la máscara, los que **difieren** son **0**.

---

## 4. 🌐 NAT y PAT

### El problema
IPv4 tiene una **cantidad limitada** de direcciones y, con millones de dispositivos, **comenzaron a agotarse**.

### NAT — Network Address Translation
- **Función:** traducir **IP privadas → IP públicas**.
- **Rangos de direcciones privadas:**
  | Rango |
  |-------|
  | 10.0.0.0/8 |
  | 172.16.0.0 – 172.31.0.0 |
  | 192.168.0.0/16 |
- **Funcionamiento:** la PC interna genera tráfico → el router reemplaza la IP privada por la pública → Internet responde → el router devuelve el tráfico al equipo correcto.
- **Ventajas:** ahorra IP públicas, mejora la organización, **oculta parcialmente la red interna**, facilita la administración.
- **Limitaciones:** **NO reemplaza al firewall**, puede dificultar el diagnóstico y **no evita el malware**.

### PAT — Port Address Translation
- **Diferencia con NAT:** usa **una IP pública compartida** y diferencia las conexiones por **puertos distintos**.
- **Analogía:** una recepción de hotel (IP pública) con habitaciones diferentes (puertos) para cada equipo.

  | Equipo | Puerto |
  |--------|--------|
  | 192.168.0.10 | 3001 |
  | 192.168.0.11 | 3002 |
  | 192.168.0.12 | 3003 |

- **Configuración:**
  ```
  Router(config)# access-list 1 permit 192.168.0.0 0.0.0.255
  Router(config)# ip nat inside source list 1 interface g0/0 overload
  ```
  - `access-list 1` → define la red interna a traducir.
  - **`overload`** → permite **múltiples conexiones** sobre la misma IP pública (esto es lo que define a PAT).

---

## 5. 🚦 ACL (Listas de Control de Acceso de red)

> Estas son las **ACL de red (routers/switches)**, del mismo documento que NAT/PAT — *no* las ACL de Linux (`setfacl`).

### ¿Qué son?
**ACL = Access Control List** (Lista de Control de Acceso): conjuntos de **reglas que permiten o bloquean tráfico** dentro de una red. Funcionan como **filtros** que analizan los paquetes y deciden si **permitirlos, denegarlos, continuar o descartarlos**.

- **Se configuran en:** routers, switches de capa 3, firewalls y equipos de seguridad.
- **Objetivos:** controlar acceso, mejorar seguridad, reducir tráfico innecesario e implementar políticas.

### Funcionamiento (clave para el parcial) ⭐
- El dispositivo revisa las reglas **en orden**; con la **primera coincidencia** ejecuta la acción.
- Si **no hay coincidencia**, el tráfico **se bloquea** (deny implícito).
- **El orden es extremadamente importante:** una regla mal ubicada puede bloquear toda la red.

**Ejemplo correcto** (primero permite lo autorizado, luego niega el resto):
```
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any
```
**Ejemplo incorrecto** (la primera regla niega todo → bloquea la red entera):
```
access-list 10 deny any
access-list 10 permit 192.168.1.0 0.0.0.255
```

### ACL estándar
- Trabajan principalmente con la **IP de origen**.
- **No** analizan puertos, protocolos específicos ni servicios.

### Wildcard mask
- La **máscara wildcard** indica **qué parte de la dirección puede variar**.
- Es el **inverso** de la máscara normal:
  | Máscara normal | Wildcard |
  |----------------|----------|
  | 255.255.255.0 | 0.0.0.255 |

### Inbound vs Outbound
- **Inbound:** filtra el tráfico **al entrar** a la interfaz.
- **Outbound:** filtra el tráfico **al salir** de la interfaz.
- Se aplica a una interfaz, p. ej.: `ip access-group 10 in`.

### Ejemplos típicos
**Permitir solo a una red (Administración) y bloquear el resto:**
```
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any
Router(config)# interface g0/0
Router(config-if)# ip access-group 10 in
```
**Bloquear un equipo específico y permitir el resto:**
```
access-list 20 deny host 192.168.1.50
access-list 20 permit any
```

### Buenas prácticas y errores frecuentes
| ✅ Buenas prácticas | ❌ Errores frecuentes |
|---------------------|----------------------|
| Documentar (propósito, fecha, ubicación, responsable) | Crear la ACL pero **olvidar aplicarla** a una interfaz |
| Mantener la **simplicidad** | Colocar las reglas en **orden incorrecto** |
| Aplicar el **mínimo privilegio** (permitir solo lo necesario) | **No probar** antes de producción |

---

## ✅ Checklist final de repaso
- [ ] Firewall: los 2 principios, qué sí/no protege, estrategias, los 7 tipos, iptables (ACCEPT/REJECT/DROP, 3 tablas).
- [ ] Máscaras: resolver los 3 ejercicios sin mirar; fórmulas `2ⁿ`, `2ʰ−2`, salto `256−octeto`.
- [ ] AND → red · OR (con inverso) → broadcast.
- [ ] NAT (privadas→públicas, rangos privados) vs PAT (`overload`, puertos).
- [ ] ACL: orden de reglas, permit/deny, wildcard, inbound/outbound.

---

*Fuentes: documentos del curso — "Firewall - Cortafuegos", "Resumen Firewall - Tipos", "Máscaras y Subredes", "Crear subredes a partir de una máscara", "Apoyo IP – Máscaras" (tablas de bits, AND/OR y máscaras) y "ACL básicas - Port Security y NAT/PAT". No se agregó información externa a esos documentos.*
