TESTING.md# TESTING n8n 2.0 para automatizaciones de bufetes de abogados

Este documento describe las pruebas recomendadas para los flujos de n8n 2.0 en este repositorio, as como las adaptaciones necesarias por paquete para que las automatizaciones funcionen correctamente en distintos entornos.

## 1. Alcance de las pruebas

- Validar que todos los workflows importan sin errores en n8n 2.x.
- Comprobar que las credenciales requeridas estn definidas y documentadas.
- Verificar que los nodos crticos (HTTP Request, Function, Code, Webhook, Execute Workflow) funcionan segn lo esperado.
- Asegurar que los flujos son idempotentes (se pueden re-ejecutar sin efectos secundarios indeseados).
- Revisar la compatibilidad con ejecuci3n manual y programada (cron/trigger).

## 2. Requisitos previos de entorno

Antes de ejecutar las pruebas:

- n8n >= 2.0.0 (recomendado: 32ltima versi3n estable LTS).
- Base de datos configurada (PostgreSQL o SQLite, segn tu despliegue).
- Variables de entorno definidas (ver `.env.example` si existe o secci3n de adaptaciones por paquetes).
- Credenciales para:
  - Correo (SMTP/IMAP) si hay flujos de notificaci3n.
  - CRM/ERP jur3dico si el flujo se integra con software de despacho.
  - Almacenamiento de archivos (Google Drive, OneDrive, S3, etc.).

## 3. Plan de pruebas gen3rico

Para cada workflow:

1. **Importaci3n**: importar el JSON del flujo en n8n 2.0 y comprobar que no aparecen errores de nodos deprecados.
2. **Configuraci3n**: rellenar credenciales y variables requeridas (por ejemplo, IDs de carpetas, URLs base, tokens de API) segn la secci3n de "Adaptaciones por paquetes".
3. **Datos de prueba**:
   - Crear un expediente de prueba ficticio.
   - Usar un cliente de prueba (no real) para datos personales.
4. **Ejecuci3n manual**: ejecutar el flujo paso a paso desde n8n y revisar:
   - Que las entradas y salidas de cada nodo son coherentes.
   - Que los documentos generados/almacenados se guardan donde corresponde.
   - Que los eventos de calendario, tareas o comunicaciones se crean una sola vez.
5. **Ejecuci3n automatizada**: activar los triggers (cron/webhook/app) y comprobar:
   - Frecuencia correcta.
   - Manejo de errores (retry, fallbacks, notificaciones).
6. **Limpieza**: borrar o marcar los datos de prueba para no mezclarlos con datos reales.

## 4. Casos de prueba recomendados

### 4.1 Alta de nuevo expediente

- Crear un expediente de prueba desde el origen previsto (formulario, CRM, email, etc.).
- Verificar que:
  - Se crea la carpeta/documentaci3n base.
  - Se registran tareas iniciales para el abogado responsable.
  - Se env3a correo de bienvenida o confirmaci3n al cliente (si aplica).

### 4.2 Control de plazos

- Simular resoluciones o hitos procesales.
- Comprobar que se crean eventos en calendario con recordatorios (p.ej. -7 d3as, -1 d3a).
- Asegurar que los cambios de fecha se sincronizan correctamente si el hito se modifica.

### 4.3 Generaci3n de documentos

- Lanzar el flujo que genera minutas, escritos o contratos.
- Confirmar que los datos de cliente/expediente aparecen correctos.
- Validar que la nomenclatura de archivos y ubicaci3n de almacenamiento es consistente.

### 4.4 Facturaci3n y tiempos

- Crear registros de tiempo/hitos de facturaci3n de prueba.
- Ejecutar el flujo de generaci3n de facturas y comprobar totales, impuestos y estados.

### 4.5 Notificaciones y seguimiento

- Verificar env3o de notificaciones internas (Teams/Slack/Email) para eventos clave.
- Probar reintentos ante errores temporales (timeout HTTP, rate limit, etc.).

## 5. Adaptaciones necesarias por paquetes (dependencias)

Esta secci3n recoge las adaptaciones m3s habituales necesarias por paquetes/dependencias al ejecutar los flujos en, por ejemplo, Docker, n8n Cloud o despliegues on-premise.

### 5.1 Paquetes de nodos personalizados

Si utilizas nodos personalizados:

- Aseg3rate de que el paquete est compilado para n8n 2.x.
- Revisa el `package.json` para comprobar compatibilidad con la versi3n de n8n instalada.
- Documenta en el README c3mo instalarlo (`N8N_CUSTOM_EXTENSIONS`, `npm install`, etc.).

### 5.2 Paquetes usados en nodos Code/Function

Para c3digo que depende de paquetes externos:

- Listar los paquetes necesarios (p. ej. `luxon`, `lodash`, `ajv`, etc.).
- Indicar c3mo se cargan en el entorno de n8n (preinstalados en la imagen Docker, `npm install` en volumen, etc.).
- Especificar versiones m3nimas probadas.

Ejemplo de tabla de adaptaciones:

| 0rea / Flujo                       | Paquete          | Versi3n m3nima | Notas de adaptaci3n |
|--------------------------------------|------------------|------------------|----------------------|
| Fechas de plazos procesales          | `luxon`          | ^3.x             | Configurar zona horaria por defecto |
| Validaci3n de datos de cliente     | `ajv`            | ^8.x             | Esquemas JSON ubicados en `/schemas` |
| Normalizaci3n de nombres de archivos | `slugify`        | ^1.x             | Evitar caracteres especiales en nombres |

### 5.3 Integraciones con APIs externas

- Documentar requisitos de cada API (autenticaci3n, scopes, IPs permitidas).
- Indicar variables de entorno necesarias (`API_BASE_URL`, `API_KEY`, `TENANT_ID`, etc.).
- Especificar l3mites de rate limit y c3mo se manejan (retries, backoff).

### 5.4 Entornos (dev / pre / prod)

- Definir c3mo se parametriza cada entorno mediante:
  - Variables de entorno.
  - Credenciales separadas.
  - Etiquetas o naming de workflows.
- Incluir recomendaciones para pruebas seguras con datos anonimizados.

## 6. Checklist r1pida antes de subir cambios

Antes de subir cambios a los workflows o adaptaciones de paquetes:

- [ ] Todos los flujos se han probado en n8n 2.x.
- [ ] No hay nodos deprecados sin alternativa.
- [ ] Las credenciales necesarias estn documentadas.
- [ ] Las dependencias externas estn listadas y probadas.
- [ ] Los datos de prueba se han limpiado o anonimizado.
- [ ] El README est alineado con este documento de pruebas.

---

Si necesitas ampliar este documento, puedes a1adir secciones espec3ficas por flujo (p.ej. `TESTING-facturacion.md`, `TESTING-plazos.md`) o incluir ejemplos de respuestas de nodos para facilitar la depuraci3n.
