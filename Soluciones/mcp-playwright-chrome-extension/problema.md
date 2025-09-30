# Problema con MCP Playwright y Extensión de Chrome

**Descripción:** Chrome borra ocasionalmente su carpeta de extensiones (`/home/c2mismo/.config/google-chrome/Default/Extensions/`), lo que causa la pérdida de la extensión `playwright-mcp-extension-0.0.39` necesaria para el funcionamiento del MCP playwright. La extensión funciona correctamente cuando está instalada, pero su eliminación automática por Chrome interrumpe el servicio.

## Configuración Actual del MCP Playwright
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--extension",
        "--executable-path=/usr/bin/google-chrome-stable"
      ],
      "disabled": false,
      "alwaysAllow": []
    }
  }
}
```

## Plan de Solución

### Análisis del Problema
- **Causa raíz:** Chrome realiza limpiezas automáticas que eliminan la carpeta de extensiones
- **Impacto:** El MCP playwright deja de funcionar cuando falta la extensión
- **Recursos disponibles:** La extensión está guardada en `/home/c2mismo/Templates/extensionChromeMCPplaywright/playwright-mcp-extension-0.0.39`
- **Estado actual:** La extensión existe actualmente en el directorio de Chrome

### Estrategias Propuestas por el Usuario (Actualizadas)
1. **Solución directa en Chrome:** Investigar configuración de Chrome para prevenir eliminación de extensiones
2. **Recuperación reactiva con Kilo Code:** Detectar error al acceder al MCP y copiar automáticamente la carpeta de extensión desde templates
3. **Verificación proactiva con Kilo Code:** Antes de ejecutar el MCP playwright, verificar que existe la carpeta `playwright-mcp-extension*` y copiar desde templates si no existe

### Plan de Implementación Recomendado

**Opción 3 (Recomendada): Verificación proactiva con manejo de versiones y detección automática**

#### Fase 1: Creación del Script de Verificación
- **Script:** `/home/c2mismo/.kilocode/scripts/verify-chrome-extension.sh`
- **Funcionalidades:**
  - Verificar existencia de cualquier carpeta `playwright-mcp-extension*` en `/home/c2mismo/.config/google-chrome/Default/Extensions/`
  - Detectar automáticamente la versión disponible en `/home/c2mismo/Templates/extensionChromeMCPplaywright/` (ej: `playwright-mcp-extension-0.0.39`)
  - Copiar extensión desde templates si no existe en Chrome
  - Manejar múltiples versiones y actualizaciones automáticas
  - Soporte para cambios de versión futuros (patrón `playwright-mcp-extension-*`)

#### Fase 2: Integración con Kilo Code
- **Modificación de configuración MCP:** Actualizar `.kilocode/mcp.json` para incluir pre-hook
- **Estrategia de ejecución:** Ejecutar script de verificación antes de iniciar el servidor MCP playwright
- **Manejo de errores:** Capturar errores y proporcionar feedback al usuario
- **Alternativa:** Implementar wrapper script que detecte errores del MCP y realice restauración reactiva

#### Fase 3: Implementación de Logging
- **Registro de eventos:** Log de restauraciones de extensión
- **Notificaciones:** Alertas cuando se realice una restauración
- **Monitoreo:** Seguimiento del estado de la extensión

### Pasos Específicos para el Modo OS Testing

1. **Crear directorio de scripts:**
   ```bash
   mkdir -p /home/c2mismo/.kilocode/scripts/
   ```

2. **Implementar script de verificación:**
   - Crear script bash que:
     - Liste directorios de extensiones en Chrome
     - Busque patrones `playwright-mcp-extension*`
     - Copie desde template si no existe
     - Maneje detección automática de versión (soporte para `playwright-mcp-extension-*`)
     - Soporte para cambios de versión futuros

3. **Modificar configuración MCP:**
   - Actualizar `.kilocode/mcp.json` para ejecutar script previamente
   - Implementar wrapper script o modificar command actual

4. **Probar funcionalidad:**
   - Simular eliminación de extensión
   - Verificar restauración automática
   - Validar funcionamiento del MCP

### Archivos y Recursos Necesarios
- **Script de verificación:** `/home/c2mismo/.kilocode/scripts/verify-chrome-extension.sh`
- **Script de recuperación reactiva (alternativa):** `/home/c2mismo/.kilocode/scripts/restore-chrome-extension.sh`
- **Configuración MCP actualizada:** `.kilocode/mcp.json`
- **Template de extensión:** `/home/c2mismo/Templates/extensionChromeMCPplaywright/`
- **Documentación:** Este archivo de seguimiento

### Consideraciones Importantes
- **Manejo de versiones dinámicas:** El script debe detectar automáticamente la versión disponible en templates (ej: `playwright-mcp-extension-0.0.39`)
- **Patrón de búsqueda:** Usar `playwright-mcp-extension*` para soportar futuras versiones
- **Seguridad:** Verificar integridad de la extensión antes de copiar
- **Rendimiento:** El script debe ser rápido y no afectar el inicio del MCP

## Solución

**Estado:** RESUELTO (Manual)

### Proceso de Implementación Automatizada (Histórico)
1. **Creación del directorio de scripts:** Se ejecutó `mkdir -p .kilocode/scripts` con éxito (exit code 0).

2. **Implementación del script de verificación:** Se creó `.kilocode/scripts/verify-chrome-extension.sh` con logging, detección dinámica y copia condicional. Se hizo executable con `chmod +x` (exit code 0).

3. **Modificación de la configuración MCP:** Se actualizó `.kilocode/mcp.json` para integrar el script como pre-ejecución (exit code 0).

4. **Pruebas Exhaustivas:** Se verificó restauración desde ausencia, simulación de eliminación, corrección de bug (uso de `ls -d`), y validación del MCP (exit code 0 en todas las ejecuciones).

### Logs de Pruebas Ejemplares (Históricos)
- **Restauración desde ausencia:**
  ```
  [2025-09-25 18:35:36] Iniciando verificación de extensión Playwright MCP
  [2025-09-25 18:35:36] Versión de extensión encontrada en templates: playwright-mcp-extension-0.0.39
  [2025-09-25 18:35:36] Extensión no encontrada en Chrome. Copiando desde templates.
  [2025-09-25 18:35:36] Extensión copiada exitosamente: playwright-mcp-extension-0.0.39
  [2025-09-25 18:35:36] Verificación completada.
  ```
- **Verificación con coincidencia (post-corrección):**
  ```
  [2025-09-25 18:37:59] Iniciando verificación de extensión Playwright MCP
  [2025-09-25 18:37:59] Versión de extensión encontrada en templates: playwright-mcp-extension-0.0.39
  [2025-09-25 18:37:59] Extensión ya existe en Chrome: playwright-mcp-extension-0.0.39
  [2025-09-25 18:37:59] Versión coincide, no se necesita copia.
  ```

### Reversión de la Solución Automatizada
Dado que la extensión se configuró manualmente y ahora funciona sin necesidad de automatización, se revirtió la implementación:
1. **Restauración de mcp.json:** Se reescribió `.kilocode/mcp.json` al estado original, removiendo la integración del script (command: "npx", args originales sin bash -c). Exit code 0.
2. **Eliminación del script:** Se removió `.kilocode/scripts/verify-chrome-extension.sh` con `rm` (exit code 0).
3. **Eliminación del directorio de scripts:** Si está vacío, se puede remover opcionalmente con `rmdir .kilocode/scripts` si no se usa para otros fines.

### Consideraciones Finales
- **Compatibilidad:** La configuración manual resuelve el problema inmediato. Monitorear eliminaciones futuras de Chrome y considerar re-implementar automatización si persiste.
- **Seguridad y Rendimiento:** La solución manual evita overhead de scripts, pero requiere intervención manual en caso de pérdida.
- **Mantenimiento:** Mantener el template en `/home/c2mismo/Templates/extensionChromeMCPplaywright/` para futuras configuraciones manuales. Si se actualiza la versión, copiar manualmente.
- **Archivos de prueba:** No hay archivos adicionales; la limpieza está completa.

*El problema se resolvió configurando la extensión manualmente en Chrome. La implementación automatizada se documenta para referencia futura si es necesaria.*