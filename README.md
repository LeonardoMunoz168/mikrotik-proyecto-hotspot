# 🌐 BarNet Hotspot V2 - Portal Cautivo MikroTik

![RouterOS](https://img.shields.io/badge/RouterOS-7.24.5-blue?logo=mikrotik)
![HTML5](https://img.shields.io/badge/Frontend-HTML5%20%7C%20CSS3-orange)
![Status](https://img.shields.io/badge/Status-Producci%C3%B3n-success)

Sistema de portal cautivo personalizado con temática de Itachi Uchiha, optimizado para la gestión dinámica de cuotas de datos (25 GB / 10 GB) en routers MikroTik hAP ax².

---

## 📑 Tabla de Contenidos
1. [Descripción del Proyecto](#descripción-del-proyecto)
2. [Estructura de Directorios](#estructura-de-directorios)
3. [Requisitos Previos](#requisitos-previos)
4. [Páginas Principales del Frontend](#páginas-principales-del-frontend)
5. [Integración de Variables MikroTik](#integración-de-variables-mikrotik)
6. [Gestión de Caché y Sesiones (mac-cookie)](#gestión-de-caché-y-sesiones)
7. [Flujo con Scripts del Backend](#flujo-con-scripts-del-backend)
8. [Instalación y Despliegue](#instalación-y-despliegue)

---

## 🚀 Descripción del Proyecto
El proyecto `hotspot_BarNet_V2` es un frontend *Mobile-First* diseñado para integrarse con la lógica de Hotspot de RouterOS 7.x. Su principal objetivo es proporcionar una experiencia de usuario fluida, soportando reconexiones invisibles mediante `mac-cookie` mientras muestra el consumo real de datos asignados por scripts en el backend.

---

## 📁 Estructura de Directorios
```text
/hotspot_BarNet_V2
│
├── login.html      # Pantalla principal de autenticación
├── status.html     # Panel de control y estado de la sesión activa
├── logout.html     # Pantalla de confirmación de cierre de sesión
├── alogin.html     # Redirección automática transparente
├── rlogin.html     # Redirección post-login
├── error.html      # Plantilla global de errores (mensajes del router)
│
├── /CSS            # Hojas de estilo personalizadas
│   └── style.css   # Estilos base, colores oscuros y diseño responsivo
│
└── /img            # Recursos gráficos
    ├── bg.jpg      # Fondo principal (Itachi)
    └── logo.png    # Logotipos de BarNet
```

---

## ⚙️ Requisitos Previos
* Router MikroTik compatible con RouterOS v7.x (Probado en hAP ax² - C52iG-5HaxD2HaxD).
* Servicio de Hotspot activo.
* Autorización en el `Walled Garden` para recursos externos (CDN de FontAwesome, Cloudflare).

---

## 🖥️ Páginas Principales del Frontend

### 1. Pantalla de Autenticación (`login.html`)
Integración del formulario estándar de MikroTik (`$(link-login-only)`), ocultando elementos innecesarios y enfocándose en la entrada directa de credenciales. Captura errores de red mediante la variable `$(error)`.

### 2. Panel de Estado (`status.html`)
Muestra al usuario métricas de conexión en tiempo real:
* Tarjeta de usuario (Dirección IP, Subida/Bajada, Tiempo activo).
* **Consumo de Datos:** La fila que muestra el saldo de Gigas se renderiza dinámicamente solo si el usuario tiene un límite asignado en la base de datos de MikroTik.

### 3. Página de Errores (`error.html`)
Captura mensajes de denegación generados por el router y los renderiza en una interfaz acorde al diseño oscuro del portal.

---

## 🔌 Integración de Variables MikroTik
El HTML utiliza el motor de plantillas de RouterOS para inyectar datos en vivo:

| Variable HTML | Función en el Portal |
| :--- | :--- |
| `$(username)` | Muestra el nombre de usuario activo (ej. `admini`). |
| `$(ip)` | Dirección IP asignada por DHCP. |
| `$(bytes-in-nice)` | Consumo actual de subida en formato legible (KiB/MiB). |
| `$(uptime)` | Tiempo de conexión de la sesión actual. |
| `$(remain-bytes-total-nice)` | **Crítico:** Muestra los Gigas restantes. Solo se imprime si existe un límite configurado. |

**Lógica Condicional:**
```html
$(if limit-bytes-total)
<tr>
    <td>Datos Restantes:</td>
    <td>$(remain-bytes-total-nice)</td>
</tr>
$(endif)
```

---

## 🛡️ Gestión de Caché y Sesiones
Para soportar la autenticación automática (`cookie` y `mac-cookie`) sin congelar la información de consumo en los navegadores móviles, se incluyen cabeceras anti-caché estrictas en `<head>`:

```html
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Expires" content="0">
```

---

## 🔗 Flujo con Scripts del Backend
El frontend depende del siguiente flujo automatizado en el evento `On Login` del router:

1. El usuario se conecta (carga `login.html` o vía `mac-cookie`).
2. MikroTik ejecuta el script `On Login`.
3. Si el usuario es nuevo (consumo = 0 y límite = nulo), el script le asigna en DB **26,843,545,600 bytes** (25 GB).
4. El script reinicia la sesión activa (`/ip hotspot active remove`).
5. El cliente se reconecta en milisegundos gracias al `mac-cookie`.
6. `status.html` despliega los datos precisos sin requerir recargas manuales.

---

## 🛠️ Instalación y Despliegue
1. Clona este repositorio o descarga el archivo ZIP.
2. Abre WinBox y ve a la sección **Files**.
3. Arrastra y suelta la carpeta `hotspot_BarNet_V2` en la raíz de los archivos de tu MikroTik.
4. Ve a **IP > Hotspot > Server Profiles**.
5. En la pestaña **General**, cambia el campo `HTML Directory` apuntando a `hotspot_BarNet_V2`.
6. Asegúrate de configurar los permisos en el `Walled Garden` si usas iconos o fuentes externas.