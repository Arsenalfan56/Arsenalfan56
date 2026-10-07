# Diagrama de Arquitectura de Red

```mermaid
graph LR
    %% Definición de Estilos Globales
    classDef sedes fill:#fffde7,stroke:#fbc02d,stroke-width:2px
    classDef red fill:#f9f9f9,stroke:#333,stroke-width:2px
    classDef seguridad fill:#e1f5fe,stroke:#0288d1,stroke-width:2px
    classDef nids fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef datos fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    classDef salida fill:#fafafa,stroke:#616161,stroke-width:2px,stroke-dasharray: 5 5

    %% Elementos Externos y Tráfico
    Internet(("🌐 Internet")):::salida

    %% Subgrafiado de las 3 Conexiones / Zonas Geográficas
    subgraph Zonas ["Zonas de Tráfico Remoto (Sedes)"]
        direction TB
        CBP["🏢 Zona CBP<br/>Tráfico Local Sedes"]:::sedes
        Guaraguao["🏭 Zona Guaraguao<br/>Tráfico Operaciones / OT"]:::sedes
        Colon["🏢 Zona Edificio Colón<br/>Tráfico Administrativo GSI"]:::sedes
    end

    %% Subgrafiado de Infraestructura de Red y Seguridad Perimetral
    subgraph Perímetro ["Infraestructura de Red Core"]
        direction LR
        Router["🎛️ Router Central<br/>Enrutamiento GSI"]:::red
        Firewall["🔥 Firewall Perimetral<br/>Políticas de Seguridad"]:::red
    end

    Tráfico(("🔌 Tráfico de Red Consecutivo<br/>(Consolidated Traffic)")):::red

    %% Subgrafiado de Captura con Herramientas Detalladas
    Sniffer["👁️ Sniffer de Red & Packet Capture<br/>• Wireshark / Tshark<br/>• Tcpdump (CLI)<br/>• Zeek / Bro (Metadata)<br/>• Arkime / Moloch (FPC)"]:::seguridad

    %% Subgrafiado del Ecosistema NIDS (Maltrail Integrado)
    subgraph NIDS ["Sistema NIDS (Maltrail System)"]
        direction TB
        Sensor["📡 Maltrail Sensor<br/>Rust / libpcap<br/>Trail matching & Heuristics"]:::nids
        Server["🖥️ Maltrail Server<br/>Python<br/>Intake, UI & API"]:::nids
        Logs[("📁 Event Logs<br/>LOG_DIR local")]:::nids
    end

    %% Base de Datos Centralizada
    DB[("🗄️ Base de Datos Central<br/>Logs históricos, Alertas<br/>& Listas de Amenazas")]:::datos

    %% Destinos de Salida y Visualización
    Web(("🌍 Sitio Web / Cloud<br/>(Servidor expuesto)")):::salida
    Browser(("💻 Browser<br/>Interfaz de Reportes GSI")):::salida
    SIEM["🛡️ Syslog / SIEM<br/>Formato CEF / JSON"]:::seguridad

    %% --- FLUJOS Y CONEXIONES ---

    CBP -->|Enlace Dedicado / VPN| Router
    Guaraguao -->|Enlace Dedicado / VPN| Router
    Colon -->|Enlace Dedicado / VPN| Router

    Internet <-->|WAN| Router
    Router <-->|LAN| Firewall

    Firewall -->|Espejo de Tráfico (SPAN/TAP)| Tráfico
    Tráfico --> Sniffer

    Sniffer -->|Inyección de Paquetes Raw (libpcap)| Sensor

    Sensor -->|Local Events| Logs
    Sensor -->|Remote Events / LOG_SERVER| Server
    Server -->|Guarda Eventos Remotos| Logs
    Logs -->|Lee Logs Diarios| Server

    Sensor -->|Registra Alertas Tempranas| DB
    Server <-->|Consulta/Actualiza Indicadores| DB

    Sensor -->|Alertas SYSLOG/LOGSTASH| SIEM
    Server <-->|HTTP / SSE| Browser
    Server -->|Publicación / API Sync| Web
```
