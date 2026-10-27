

# Lab 04: SAML 2.0 SSO con Microsoft Entra ID — Operación Amador Rivas

## 1. Descripción del proyecto
En **Mirador de Montepinar**, la seguridad de las identidades empieza a ponerse seria. Después de trabajar la identidad híbrida, la autenticación multifactor (MFA) y la gestión del ciclo de vida de los usuarios (JML), toca resolver otro problema crítico: que los empleados y vecinos puedan acceder a las aplicaciones corporativas mediante un sistema de inicio de sesión único, sin tener que gestionar ni introducir credenciales independientes en cada servicio.

En este laboratorio, **Amador Rivas (`arivas`)** es la identidad corporativa utilizada para configurar y validar la federación mediante **Single Sign-On (SSO)** utilizando el estándar **SAML 2.0**.

La práctica utiliza **Microsoft Entra ID** como proveedor de identidad (*Identity Provider, IdP*) y **Microsoft Entra SAML Toolkit** como aplicación externa (*Service Provider, SP*). El objetivo es establecer la relación de confianza criptográfica entre ambas partes, definir las notificaciones (*claims*) e identificadores intercambiados y auditar el flujo de autenticación real inspeccionando el tráfico con **SAML-tracer**.

### Objetivos técnicos
* Configurar e integrar una aplicación empresarial (*Enterprise Application*) en Microsoft Entra ID.
* Establecer la relación de confianza SAML 2.0 entre el IdP y el SP mediante intercambio de metadatos y certificados X.509.
* Configurar los identificadores críticos de federación: *Entity ID*, *ACS URL* y *Sign-on URL*.
* Definir el identificador único de sujeto (`NameID`) y el mapeo de atributos (*Claims*) de Amador Rivas.
* Ejecutar un flujo real de autenticación **SP-Initiated SSO**.
* Interceptar y analizar la aserción XML SAML mediante SAML-tracer para verificar el acceso autenticado.
* Documentar las evidencias técnicas y los controles de seguridad asociados al protocolo.

> **Escenario corporativo:** *Mirador de Montepinar S.L.*, donde Amador Rivas necesita demostrar que puede entrar a los sistemas corporativos de forma centralizada sin tener que dar explicaciones ni recordar contraseñas distintas cada vez que cambia de puerta.

---

## 2. Arquitectura del laboratorio

| Componente | Función en la Arquitectura |
| :--- | :--- |
| **Microsoft Entra ID** | Proveedor de identidad (*Identity Provider - IdP*) |
| **Microsoft Entra SAML Toolkit** | Aplicación SaaS de destino / Proveedor de servicios (*Service Provider - SP*) |
| **Amador Rivas (`arivas`)** | Identidad corporativa utilizada para validar la federación |
| **SAML 2.0** | Estándar abierto de federación e intercambio de aserciones XML |
| **SAML-tracer** | Herramienta de auditoría para inspección de peticiones HTTP/SAML |

### Flujo de autenticación (SP-Initiated SSO)
1. **Inicio en el SP:** Amador accede directamente a la URL de inicio de sesión del Toolkit.
2. **Solicitud SAML (`AuthnRequest`):** La aplicación detecta que no hay sesión activa y redirige el navegador de Amador hacia Microsoft Entra ID.
3. **Autenticación en el IdP:** Microsoft Entra ID valida las credenciales de Amador, comprueba la asignación de acceso a la aplicación y construye una respuesta SAML firmada digitalmente.
4. **Entrega de la aserción:** El navegador envía mediante una petición `POST` la respuesta SAML (`SAMLResponse`) al endpoint **ACS (Assertion Consumer Service)** del Toolkit.
5. **Validación y sesión:** El SP verifica la firma criptográfica del certificado, valida las condiciones del token y establece la sesión autenticada.
6. **Acceso transparente:** Amador navega por la aplicación bajo su identidad federada sin gestionar credenciales locales en el destino.

---

## 3. Configuración e implementación

### Fase 1. Preparación de la aplicación empresarial y control de acceso
Se desplegó la instancia **Microsoft Entra SAML Toolkit** dentro del catálogo de *Enterprise Applications* en Microsoft Entra ID y se asignó explícitamente la cuenta de Amador Rivas (`arivas`) para autorizar la emisión de tokens desde el IdP.

![SAML](01_SAML.png)

### Fase 2. Configuración básica de SAML
Se definieron los identificadores de entidad y los endpoints de comunicación requeridos por el proveedor de servicios:

* **Identifier (Entity ID):** `https://samltoolkit.azurewebsites.net`
* **Reply URL (Assertion Consumer Service - ACS):** `https://samltoolkit.azurewebsites.net/SAML/Consume/22786`
* **Sign-on URL:** `https://samltoolkit.azurewebsites.net/SAML/Login/22786`

Estos parámetros garantizan que Entra ID emita aserciones dirigidas exclusivamente a este servicio (*Audience Restriction*) y entregue el token en el receptor exacto de la instancia configurada.

![SAML_Conf](02_SAMLConfi.png)

### Fase 3. Configuración de NameID y mapeo de atributos (Claims)
Se estableció el identificador principal (`NameID`) en formato de dirección de correo electrónico mapeado al atributo `user.userprincipalname`. Adicionalmente, se configuró el envío de los siguientes atributos en la aserción:

| Atributo de salida (Claim) | Atributo de origen en Entra ID |
| :--- | :--- |
| `emailaddress` | `user.mail` |
| `givenname` | `user.givenname` |
| `name` | `user.userprincipalname` |
| `surname` | `user.surname` |

Como parte de la vinculación de identidad (*Account Matching*) con el usuario registrado en el SP, se normalizó el atributo `mail` de Amador en Entra ID para garantizar la correspondencia exacta durante la validación del token.

### Fase 4. Establecimiento de confianza en el Proveedor de Servicios (SP)
En la consola de configuración del SAML Toolkit se importaron los parámetros criptográficos y de enrutamiento de Microsoft Entra ID:
* **Login URL** y **Logout URL** del tenant.
* **Microsoft Entra Identifier** (Emisor / `Issuer` de la aserción).
* **Certificado de firma X.509 (Base64)** generado por Entra ID para que el SP pueda verificar la integridad y autenticidad de las respuestas SAML entrantes.

### Fase 5. Ejecución y auditoría de tráfico con SAML-tracer
Se ejecutó el flujo *SP-Initiated* con el usuario `arivas`. Tras superar el desafío de autenticación en Entra ID, se capturó el tráfico de retorno mediante **SAML-tracer**, identificando la entrega de la aserción al endpoint ACS:

* **Petición HTTP:** `POST https://samltoolkit.azurewebsites.net/SAML/Consume/22786` *(etiquetada como tráfico SAML)*.
* **Inspección del XML:** Se verificó en el bloque de respuesta el destino configurado (`Destination`) y la confirmación de éxito en el protocolo:
  `<samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>`

  ![SAML](07_SAMLTracer.png)

### Fase 6. Validación de la sesión autenticada
Una vez consumida la aserción por el endpoint ACS, la aplicación estableció la sesión mostrando en pantalla **`Hello arivas`**, confirmando la extracción exitosa del *Claim* de identidad y permitiendo la navegación autenticada sin intervención de contraseñas locales.

![SAML](06_ToolkitArivas.png)

---

## 4. Resultados obtenidos
El laboratorio validó de extremo a extremo la arquitectura de federación SAML 2.0 con Microsoft Entra ID:
* Integración de endpoints SAML (*Entity ID*, *ACS URL*, *Sign-on URL*).
* Establecimiento de confianza bilateral mediante certificados X.509.
* Mapeo y transformación de atributos de directorio hacia notificaciones SAML (*Claims*).
* Ejecución satisfactoria del flujo **SP-Initiated SSO** con la identidad `arivas`.
* Auditoría técnica de la aserción XML y validación del estado `Success` mediante SAML-tracer.

---

## 5. Consideraciones de seguridad (IAM & GRC)
* **Prevención de manipulación de tokens (*Token Tampering*):** El proveedor de servicios debe validar obligatoriamente el bloque `<ds:Signature>` contra el certificado público del IdP antes de confiar en los atributos recibidos.
* **Restricción de audiencia y destino:** La validación estricta de `Issuer`, `Audience` (*Entity ID*) y `Recipient` (*ACS URL*) impide que una aserción interceptada pueda ser redirigida hacia otro servicio distinto.
* **Protección contra ataques de repetición (*Replay Attacks*):** En entornos de producción es crítico auditar las ventanas temporales del bloque `<saml:Conditions>` (`NotBefore` y `NotOnOrAfter`) junto con el identificador único de mensaje (`ID`).
* **Principio de minimización de datos:** Los *Claims* emitidos en la aserción deben limitarse estrictamente a los atributos que la aplicación de destino necesita para operar.
* **Higiene documental (OPSEC):** Las trazas SAML reales contienen identificadores de directorio, firmas y estructuras internas; cualquier evidencia publicada en repositorios públicos debe ser previamente anonimizada o generada en entornos aislados de laboratorio.

---

## 6. Herramientas y tecnologías
* **Microsoft Entra ID** (Enterprise Applications & Single Sign-On)
* **Microsoft Entra SAML Toolkit**
* **Estándar SAML 2.0** (OASIS)
* **SAML-tracer** (Análisis de tráfico de federación)
* **Federación de Identidades y Gestión de Claims**

---

## 7. Conclusión
Amador Rivas ha conseguido autenticarse en la aplicación corporativa delegando la seguridad en Microsoft Entra ID mediante SAML 2.0, eliminando la proliferación de contraseñas locales. 

Esta práctica consolida la comprensión arquitectónica entre proveedores de identidad (IdP) y proveedores de servicios (SP), así como la capacidad de auditar flujos de autenticación a nivel de protocolo, completando junto a los laboratorios de ciclo JML, identidad híbrida y MFA un bloque sólido de competencias prácticas en **Identity and Access Management (IAM)**.
