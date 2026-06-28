# 🕵️‍♂️ Proyecto Red Team: Ataque Man-in-the-Middle (MitM) vía ARP/DNS Spoofing y Evasión de HSTS

**Autor:** Córdoba

**Fecha:** 28/06/2026

**Categoría:** Red Team / Network Security / Penetration Testing

**Objetivo:** Desplegar un escenario de interceptación activa Man-in-the-Middle (MitM) en entorno de red local para la evaluación de debilidades en protocolos de broadcast (ARP) y resolución de nombres (DNS), analizando el impacto de las contramedidas criptográficas modernas.

## 1. Reconocimiento Pasivo y Descubrimiento de Objetivos

El proyecto comenzó con la identificación de la topología lógica de la red local para aislar los objetivos principales y evitar ruido innecesario en el segmento de red clase C (192.168.1.0/24).

* **Escaneo de Red:** Se utilizó la herramienta Nmap mediante un Ping Sweep agresivo (-sn) para forzar las peticiones ARP locales y listar las IPs activas sin levantar alertas por escaneo de puertos individuales.

Evidencias:

<img width="737" height="596" alt="nmap" src="https://github.com/user-attachments/assets/ebc60a88-02bc-4b7a-94c0-af7ff4954cf8" />

(Nota de OpSec: Se han filtrado y difuminado del reporte los dispositivos ajenos al laboratorio, manteniendo visibles únicamente los actores principales del experimento).

## 2. Preparación del Sistema y Fontanería de Red (ARP Spoofing)

Una vez localizados los objetivos, procedimos a reconfigurar los parámetros del kernel de Linux para permitir el tránsito de paquetes y levantar el escenario de interceptación dúplex.

* **Modificación del Kernel y Firewall:** Se habilitó el reenvío de paquetes IP (IP Forwarding) a nivel de núcleo para evitar actuar como un agujero negro y se deshabilitó el cortafuegos local (ufw) para garantizar la recepción de tráfico externo.

Evidencias:

<img width="526" height="174" alt="activar fowarding" src="https://github.com/user-attachments/assets/cbe40af1-f4af-4e74-8a39-a00608ffb34a" />

* **Envenenamiento de Tablas ARP:** Se inicializó la herramienta Bettercap apuntando en exclusividad a la dirección IP del dispositivo móvil víctima (192.168.1.151) para suplantar la identidad de la puerta de enlace de forma bidireccional.

Evidencias:

<img width="948" height="460" alt="arrancar bettercap" src="https://github.com/user-attachments/assets/b1de4dd9-cb7f-4e27-ac2d-2f2adbd71a48" />

## 3. Despliegue de la Trampa DNS y Servidor Web Local

Trabajando con el flujo de datos controlado a través de nuestra interfaz, construimos la infraestructura de red necesaria para capturar las peticiones de navegación y servir el código modificado.

* **Configuración del Servidor Web Efímero:** Se diseñó un archivo index.html básico con un vector de inyección JavaScript (alert) y se levantó un servidor web en segundo plano usando el módulo nativo de Python en el puerto 80.

Evidencias:

<img width="1029" height="135" alt="servidor python activo" src="https://github.com/user-attachments/assets/2884d36a-f12d-4cc7-8b5c-7137ba68e283" />

* **Falsificación de Dominios (DNS Spoofing):** Debido a las restricciones de las cachés DNS de los navegadores actuales, se configuró el módulo dns.spoof apuntando al dominio estándar de pruebas de la IANA (example.com), mapeando el comodín comodín hacia nuestra IP atacante (192.168.1.133).

Evidencias:

<img width="951" height="208" alt="configurar suplantacion dns " src="https://github.com/user-attachments/assets/4d6a3558-b06a-4cac-988a-94e70ecd2147" />

<img width="1854" height="503" alt="trampa desplegada" src="https://github.com/user-attachments/assets/6f396e76-5074-46c8-ab7b-2a1e741befa9" />

4. Auditoría de Seguridad y Evasión de Bloqueos (Troubleshooting)

Durante el desarrollo del laboratorio nos enfrentamos a las estrictas capas de optimización y seguridad perimetral de los sistemas móviles modernos, lo que requirió un análisis técnico forense.

* **El Desafío de HTTPS y HSTS:** Al intentar redirigir dominios comerciales masivos (como neverssl.com), el navegador de la víctima forzó la navegación segura a través de listas estáticas pregrabadas en su código (HSTS Preload List) o mediante solicitudes cifradas de DNS sobre HTTPS (DoH). Esto provocaba que el móvil rechazara la conexión mostrando pantallas negras de error en lugar de cargar nuestra alerta.

* **Bastionado del Cliente (Solución Técnica):** Para validar la efectividad de la infraestructura DNS, aislamos las contramedidas del cliente: desactivamos temporalmente el "DNS Seguro" en las conexiones del smartphone, apagamos el uso secundario de datos móviles (tarjeta SIM) y forzamos al navegador a emitir una petición HTTP pura e inequívoca forzando el puerto web tradicional en la URL.

Evidencias:

<img width="1080" height="1447" alt="buscador" src="https://github.com/user-attachments/assets/a8fefa3e-9302-4d9a-a712-37b70e1144bd" />
<img width="1026" height="2048" alt="alert" src="https://github.com/user-attachments/assets/038d70f7-4738-4f7d-89cc-8046662fee8b" />
<img width="1022" height="2048" alt="otra" src="https://github.com/user-attachments/assets/7343b11c-275a-48dd-952d-1d91f5e8426f" />

5. Explotación Exitosa y Monitorización del Tráfico

Con las capas de red depuradas, forzamos el tráfico del cliente hacia nuestro servidor controlado, logrando el impacto visual y el registro síncrono en los logs de auditoría.

Inyección de Código en la Víctima: Al procesar la solicitud DNS manipulada por Bettercap, el navegador del móvil de pruebas se conectó directamente a nuestro Linux, ejecutando instantáneamente el script intrusivo en pantalla.

Evidencias:

<img width="655" height="127" alt="python" src="https://github.com/user-attachments/assets/2bf96176-f981-43e6-b6b0-d1d80d6d9f55" />
<img width="1288" height="251" alt="spoof reedireccion" src="https://github.com/user-attachments/assets/420dfa5e-abde-4f5c-9547-6444aa0dbf41" />

Captura de Evidencias en los Logs: Desde nuestra consola, confirmamos la resolución exitosa del ataque observando cómo el analizador interceptaba el paquete de red y el servidor de Python registraba la petición entrante del terminal móvil con un código de respuesta exitoso (200 OK).

* **Explicación:**

El desglose detallado de los registros capturados permite identificar con precisión la mecánica de la manipulación de red a través de tres patrones de comportamiento técnico:

En primer lugar, se constata el éxito del envenenamiento de tablas a nivel de enlace de datos. El módulo de Bettercap redirige físicamente las tramas de red de la víctima, de modo que cada consulta emitida por el terminal pasa de manera obligatoria por el procesador de nuestra máquina. Al interceptar la consulta DNS broadcast que hace el navegador móvil, el sniffer frena la petición hacia el exterior y responde con un paquete modificado antes de que el router legítimo pueda intervenir, completando el secuestro del canal (hijacking).

En segundo lugar, se detalla la debilidad del protocolo HTTP en texto plano. Debido a que el dominio de pruebas utilizado no implementa firmas digitales criptográficas ni certificados SSL/TLS, la carga útil viaja sin cifrar a través del cable. Esto permite que el servidor web local inyecte etiquetas <script> personalizadas en el cuerpo del HTML de manera transparente, demostrando cómo un atacante podría comprometer sesiones o realizar ataques de ingeniería social (phishing) si el tráfico no está securizado.

Por último, los registros capturan la efectividad de la sandbox del laboratorio para depurar el tráfico. Al finalizar la interacción, el log de Python muestra la desconexión limpia del dispositivo tras haber procesado el recurso. Esto valida el uso de herramientas ligeras integradas en el sistema operativo para simular entornos de explotación web interactiva sin necesidad de desplegar infraestructuras pesadas de servidor, permitiendo estudiar las deficiencias de red en tiempo real.

* **⚠️ Nota de OpSec / Descargo de responsabilidad:** Toda la infraestructura local descrita en este laboratorio ha sido completamente destruida, purgada y desmantelada. El direccionamiento IP asignado dinámicamente y las tablas ARP de los dispositivos físicos han sido restauradas y vaciadas por completo, quedando el entorno local en su estado de producción seguro habitual.

## ✅ Conclusiones

El despliegue de este entorno ofensivo controlado revela lecciones críticas sobre la arquitectura actual de las redes y la seguridad web:

* **Inseguridad Inherente de los Protocolos Base:** Los protocolos ARP y DNS tradicionales carecen de mecanismos natively integrados de autenticación. Cualquier dispositivo en el segmento de red puede emitir respuestas falsas y ser creído por el resto de nodos, lo que resalta la importancia de implementar segmentación de red o aislamiento de clientes inalámbricos en routers comerciales.

* **La Robustez del Ecosistema de Defensa Moderno:** Las protecciones implementadas en los navegadores actuales (HSTS, Certificate Pinning y DNS sobre HTTPS) son extremadamente robustas. Durante el laboratorio se evidenció que un ataque MitM clásico es incapaz de romper la confidencialidad de plataformas corporativas reales, obligando a los analistas de seguridad a pivotar hacia técnicas de clonación de dominios (Phishing/DNS Spoofing total) en lugar de interceptación en caliente.

* **El Valor de la Depuración en Laboratorio:** El éxito del ejercicio radicó en comprender la lógica de los bloqueos (como el forzado automático de la "S" en HTTPS). Resolver estos fallos mediante el uso de puertos explícitos (:80) consolida el aprendizaje en ingeniería de redes, demostrando que analizar el porqué fallan las herramientas ante las defensas actuales es más valioso que la propia automatización del ataque.
