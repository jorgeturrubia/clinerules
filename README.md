# ClinerRules Bank - SportPlannerV2

## Estructura del Banco de Reglas

Este directorio contiene las reglas modulares para Cline, organizadas por categorías y numeradas para fácil activación/desactivación.

### Categorías de Reglas

#### 01-05: Reglas Globales
- `01-clean-code.md` - Principios de código limpio y SOLID
- `02-naming.md` - Convenciones de nomenclatura
- `03-adr.md` - Architecture Decision Records
- `04-testing.md` - Estándares de testing y cobertura
- `05-security.md` - Seguridad y mejores prácticas

#### 10-14: Reglas por Tecnología
- `10-angular-structure.md` - Estructura Angular 20 con standalone components
- `11-tailwind.md` - Tailwind CSS v4 configuration y utilities
- `12-material-animations.md` - Material Design y animaciones
- `13-supabase-jwt.md` - Autenticación JWT con Supabase
- `14-dotnet-backend.md` - .NET 8 Clean Architecture

#### 20+: Reglas por Feature
- `20-features-planifications.md` - Feature específica de planificaciones

## Cómo Usar Este Banco

### Activación por Sprint/Feature
1. **Selección Granular**: Activa solo los .md relevantes para tu Sprint actual
2. **Popover de Cline**: Usa el panel para activar/desactivar reglas específicas
3. **Combinaciones**: Puedes combinar reglas globales + tecnología + feature específica

### Ejemplos de Combinación

#### Sprint de Frontend Angular
```
✓ 01-clean-code.md
✓ 02-naming.md
✓ 04-testing.md
✓ 10-angular-structure.md
✓ 11-tailwind.md
✓ 12-material-animations.md
```

#### Sprint de Backend API
```
✓ 01-clean-code.md
✓ 02-naming.md
✓ 03-adr.md
✓ 04-testing.md
✓ 05-security.md
✓ 14-dotnet-backend.md
```

#### Sprint de Feature Completa
```
✓ 01-clean-code.md
✓ 02-naming.md
✓ 10-angular-structure.md
✓ 13-supabase-jwt.md
✓ 14-dotnet-backend.md
✓ 20-features-planifications.md
```

## Mantenimiento

### Versionado
- Cada regla se versiona en el repositorio
- Los cambios se revisan en cada PR
- Historial de cambios rastreado por Git

### Auditoría
- Revisar reglas activas al inicio de cada Sprint
- Documentar decisiones de activación/desactivación
- Mantener coherencia entre miembros del equipo

### Escalabilidad
- Agregar nuevas reglas numeradas secuencialmente
- Crear subcategorías si una tecnología requiere múltiples archivos
- Mantener archivos por debajo de 200 líneas para claridad

## Banco de Rules Externo

### Estructura Recomendada
```
clinerules-bank/
├── clients/
│   ├── cliente-a/
│   └── cliente-b/
├── technologies/
│   ├── react/
│   ├── vue/
│   ├── python/
│   └── java/
├── domains/
│   ├── ecommerce/
│   ├── fintech/
│   └── healthcare/
└── templates/
    ├── basic-frontend/
    ├── basic-backend/
    └── fullstack/
```

### Workflow de Reutilización
1. **Copia Selectiva**: Copia solo las reglas necesarias del banco al proyecto
2. **Adaptación**: Modifica las reglas copiadas según el contexto específico
3. **Contribución**: Actualiza el banco con mejoras generalizables

## Mejores Prácticas

### Nomenclatura
- Prefijos numéricos para orden y agrupación
- Nombres descriptivos y concisos
- Consistencia en la estructura de archivos

### Contenido
- Una responsabilidad por archivo
- Ejemplos concretos y aplicables
- Referencias a documentación oficial cuando sea relevante

### Activación
- Comenzar con reglas globales (01-05)
- Agregar tecnologías específicas según stack
- Incluir features solo cuando se esté trabajando en ellas

### Colaboración
- Sincronizar reglas activas en el equipo
- Documentar cambios significativos
- Revisar y actualizar reglas periódicamente

## Comandos Útiles

### Git Hooks (Opcional)
```bash
# Pre-commit hook para validar reglas activas
#!/bin/bash
echo "Validating active Cline rules..."
# Verificar que las reglas activas son coherentes con el tipo de cambios
```

### Scripts de Gestión
```bash
# Listar reglas activas
ls -la .clinerules/*.md

# Activar reglas para frontend
cp clinerules-bank/frontend/*.md .clinerules/

# Activar reglas para backend
cp clinerules-bank/backend/*.md .clinerules/
```

## Troubleshooting

### Conflictos entre Reglas
- Priorizar reglas más específicas sobre generales
- Documentar excepciones en el archivo de regla correspondiente
- Resolver conflicts en reuniones de equipo

### Performance de Cline
- No activar más de 10-12 reglas simultáneamente
- Desactivar reglas irrelevantes para el Sprint actual
- Monitorear tiempo de respuesta del agente

### Mantenimiento
- Revisar reglas obsoletas mensualmente
- Actualizar ejemplos con nuevas versiones de tecnologías
- Archivar reglas deprecated en lugar de eliminarlas

---

**Fecha de creación**: 2024-01-XX  
**Última actualización**: 2024-01-XX  
**Versión**: 1.0.0
