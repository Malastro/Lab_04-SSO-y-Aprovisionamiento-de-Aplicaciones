
# Lab 04: SAML SSO con Microsoft Entra ID — Operación Amador Rivas

## 1. Descripción del proyecto

En Mirador de Montepinar, la seguridad de las identidades empieza a ponerse seria. Después de trabajar la identidad híbrida, la autenticación multifactor y la gestión del ciclo de vida de los usuarios, toca resolver otro problema: que los vecinos puedan acceder a las aplicaciones corporativas mediante un sistema de inicio de sesión único, sin tener que introducir sus credenciales en cada servicio.

En este laboratorio, **Amador Rivas (`arivas`)** será la identidad de prueba utilizada para configurar y validar el inicio de sesión único (*Single Sign-On*, SSO) mediante SAML 2.0.

La práctica utiliza Microsoft Entra ID como proveedor de identidad (*Identity Provider*, IdP) y Microsoft Entra SAML Toolkit como aplicación de prueba (*Service Provider*, SP).

El objetivo es configurar la confianza entre ambas partes, definir los atributos de identidad intercambiados y comprobar el flujo de autenticación mediante una prueba real y el análisis de las peticiones con SAML-tracer.

### Objetivos técnicos

- Configurar una aplicación empresarial en Microsoft Entra ID.
- Establecer los parámetros SAML entre el proveedor de identidad y el proveedor de servicios.
- Configurar Entity ID, ACS URL y Sign-on URL.
- Definir NameID y los atributos de usuario de Amador Rivas.
- Ejecutar el flujo SSO con Microsoft Entra ID.
- Analizar la respuesta SAML y verificar el acceso a una sesión autenticada.
- Documentar las evidencias y las consideraciones de seguridad.

**Escenario ficticio:** Mirador de Montepinar SL, donde Amador Rivas necesita demostrar que puede entrar al sistema sin tener que dar explicaciones cada vez que cambia de puerta.

## 2. Arquitectura del laboratorio

| Componente | Función |
|---|---|
| Microsoft Entra ID | Proveedor de identidad (IdP) |
| Microsoft Entra SAML Toolkit | Aplicación de prueba y proveedor de servicios (SP) |
| Amador Rivas (`arivas`) | Identidad utilizada para validar el acceso |
| SAML 2.0 | Protocolo de federación y autenticación |
| SAML-tracer | Inspección de las peticiones y respuestas del flujo |

### Flujo de autenticación

1. Amador inicia el proceso de acceso desde el Toolkit (SP-Initiated SSO)
2. La aplicación redirige el navegador a Microsoft Entra ID.
3. Microsoft autentica a Amador y genera una respuesta SAML.
4. El navegador envía la respuesta al endpoint ACS del Toolkit.
5. La aplicación procesa la respuesta y establece una sesión autenticada.
6. Amador puede navegar por la aplicación sin volver a introducir las credenciales propias del Toolkit.

## 3. Configuración e implementación

### Fase 1. Preparación de la aplicación empresarial

Se utilizó la aplicación empresarial **Microsoft Entra SAML Toolkit** en Microsoft Entra ID y se asignó la identidad de prueba de Amador Rivas (`arivas`).


![SAML](01_SAML.png)

### Fase 2. Configuración básica de SAML

Se configuraron los endpoints y el identificador de la aplicación según los valores proporcionados por el Toolkit.

- **Identifier (Entity ID):** `https://samltoolkit.azurewebsites.net`
- **Reply URL (ACS):** `https://samltoolkit.azurewebsites.net/SAML/Consume/22786`
- **Sign-on URL:** `https://samltoolkit.azurewebsites.net/SAML/Login/22786`

Estos parámetros permiten identificar al proveedor de servicios, establecer el destino de la respuesta SAML y definir el punto de inicio del proceso de autenticación.

![SAML_Conf](02_SAMLConf.png)

### Fase 3. Configuración de NameID y atributos

Se configuró el NameID con formato de dirección de correo electrónico y se utilizó `user.userprincipalname` como atributo de origen.

También se definieron las siguientes asignaciones:

| Atributo de salida | Atributo de origen |
|---|---|
| `emailaddress` | `user.mail` |
| `givenname` | `user.givenname` |
| `name` | `user.userprincipalname` |
| `surname` | `user.surname` |

Como parte de la preparación de la identidad de prueba, se completó el atributo `mail` de Amador para que coincidiera con la dirección utilizada en el Toolkit.

### Fase 4. Configuración del proveedor de identidad

Se configuraron en el Toolkit los datos de Microsoft Entra ID necesarios para establecer la confianza con el proveedor de identidad:

- URL de inicio de sesión de Entra ID.
- Identificador de Microsoft Entra ID.
- URL de cierre de sesión.
- Certificado de firma descargado desde Entra ID.

Los valores se guardaron en la configuración de la conexión utilizada para el laboratorio.


### Fase 5. Prueba de autenticación SSO

Se inició una prueba real con la identidad de Amador Rivas. Microsoft Entra ID solicitó la autenticación y, tras completarla, el navegador regresó al Toolkit.

Mediante SAML-tracer se identificó el envío de la respuesta al endpoint ACS:

`POST https://samltoolkit.azurewebsites.net/SAML/Consume/22786` (línea de color azul)

La respuesta SAML inspeccionada incluía el destino ACS configurado y la siguiente línea de código: `<samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>`

![SAML](07_SAMLTracer.png)

### Fase 6. Validación de la sesión autenticada

Tras completar el inicio de sesión con Microsoft, el Toolkit mostró `Hello arivas` y permitió navegar por sus páginas sin volver a introducir las credenciales propias de la aplicación.

Esta comprobación, junto con la traza del envío al ACS, aporta evidencia funcional del flujo SSO configurado.

![SAML](06_ToolkitArivas.png)

## 4. Resultados

El laboratorio permitió configurar y probar un escenario de SSO federado con SAML 2.0 utilizando Microsoft Entra ID.

Se verificaron los siguientes resultados:

- Configuración de los endpoints SAML de la aplicación.
- Configuración del proveedor de identidad y del certificado de firma.
- Definición de NameID y atributos de usuario.
- Ejecución del flujo de autenticación con la identidad de Amador Rivas.
- Observación de la respuesta SAML enviada al endpoint ACS.
- Acceso a una sesión autenticada en el Toolkit.

La prueba funcional fue satisfactoria. No se realizó una auditoría independiente de todos los controles internos de validación de la aserción ni una comprobación exhaustiva de cada atributo recibido por la aplicación.

## 5. Consideraciones de seguridad

- El proveedor de servicios debe validar correctamente la firma y los elementos relevantes de la aserción SAML.
- Entity ID, ACS, emisor y destinatario deben coincidir con los valores esperados por la configuración.
- NameID y los atributos deben limitarse a la información necesaria para la aplicación.
- En producción deben revisarse los tiempos de validez y las medidas de protección frente a la reutilización de aserciones.
- Las respuestas SAML pueden contener datos de identidad y material sensible; no deben publicarse completas en repositorios públicos.
- Las capturas deben redactarse para ocultar identificadores innecesarios, datos personales y valores de autenticación.

## 6. Herramientas y tecnologías

- Microsoft Entra ID
- Microsoft Entra SAML Toolkit
- SAML 2.0
- SAML-tracer
- Enterprise Applications
- SSO y federación de identidades

## 7. Conclusión

Amador Rivas ha conseguido acceder al Toolkit mediante el flujo de autenticación con Microsoft Entra ID, sin tener que introducir de nuevo las credenciales propias de la aplicación.

El laboratorio permitió practicar la configuración de SSO con SAML 2.0, comprender la interacción entre un proveedor de identidad y un proveedor de servicios y analizar la respuesta enviada al endpoint ACS.

Esta práctica amplía los laboratorios de Mirador de Montepinar dedicados a identidad híbrida, MFA y gestión del ciclo de vida, reforzando las competencias fundamentales para un perfil inicial de Identity and Access Management (IAM).



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

![SAML_Conf](02_SAMLConf.png)

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
