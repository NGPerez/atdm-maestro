# Suite de Pruebas Maestro - TecnoShopper

Este proyecto contiene una suite de pruebas de Maestro para la aplicación TecnoShopper, configurada con un archivo `config.yaml` centralizado para controlar la ejecución, descubrimiento de pruebas, categorización por tags y salida de evidencias.

## Estructura del Proyecto

```
tecnoshopper-test-e2e/
├── config.yaml                    # Configuración central de la suite
├── android/                       # Directorio de pruebas Android
│   ├── login/                     # Módulo de login  
│   │   ├── test/                  # Tests de login
│   │   ├── task/                  # Tasks reutilizables
│   │   ├── pom/                   # Page Object Model
│   │   └── data/                  # Datos de prueba
│   ├── cart/                      # Módulo de carrito
│   ├── profile/                   # Módulo de perfil
│   └── register/                  # Módulo de registro
├── evidencias/                    # Directorio de salida de evidencias
│   └── maestro/                   # Evidencias generadas por Maestro
└── README.md                      # Este archivo
```

## Configuración (config.yaml)

El archivo `config.yaml` centraliza el control de toda la suite de pruebas:

### Descubrimiento de Pruebas
```yaml
flows:
  - android/login/test/*.test.yaml
  - android/cart/test/*.test.yaml
  - android/profile/test/*.test.yaml
  - android/register/test/*.test.yaml
```

### Orden de Ejecución
Los tests se ejecutan en el siguiente orden lógico:
1. Login exitoso
2. Login fallido
3. Registro exitoso
4. Registro duplicado
5. Comprar producto
6. Cancelar carrito
7. Actualizar perfil
8. Validar campos perfil

### Tags (Etiquetas)
Los tests están categorizados con los siguientes tags:

**Tags de módulo:**
- `login` - Pruebas de autenticación
- `register` - Pruebas de registro
- `carrito` - Pruebas de carrito de compras
- `profile` - Pruebas de perfil de usuario

**Tags de tipo de prueba:**
- `happy_path` - Casos positivos/exitosos
- `regression` - Pruebas de regresión

**Tags de prioridad:**
- `high_priority` - Pruebas críticas
- `medium_priority` - Pruebas de importancia media
- `low_priority` - Pruebas de menor importancia

### Salida de Evidencias
```yaml
testOutputDir: evidencias/maestro
```

Todas las evidencias (screenshots, logs, metadatos) se guardan en el directorio `evidencias/maestro/` con marca de tiempo.

## Formas de Ejecución

### 1. Ejecutar un Test Específico con Config

```bash
.\maestro\maestro\bin\maestro.bat test --config config.yaml android/login/test/login-exitoso.test.yaml
```

### 2. Ejecutar Toda la Suite con Config

```bash
.\maestro\maestro\bin\maestro.bat test --config config.yaml .
```

Esto ejecutará todos los tests en el orden definido en `config.yaml`.

### 3. Ejecutar por Tags de Inclusión

```bash
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags happy_path .
```

**Ejemplos de inclusión:**
```bash
# Solo tests de happy path
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags happy_path .

# Solo tests de alta prioridad
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags high_priority .

# Solo tests de regresión
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags regression .

# Solo tests de login
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags login .

# Múltiples tags (OR logic)
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags happy_path,high_priority .
```

### 4. Ejecutar Excluyendo Tags

```bash
.\maestro\maestro\bin\maestro.bat test --config config.yaml --exclude-tags low_priority .
```

**Ejemplos de exclusión:**
```bash
# Excluir tests de baja prioridad
.\maestro\maestro\bin\maestro.bat test --config config.yaml --exclude-tags low_priority .

# Excluir tests en desarrollo
.\maestro\maestro\bin\maestro.bat test --config config.yaml --exclude-tags wip .

# Excluir tests manuales
.\maestro\maestro\bin\maestro.bat test --config config.yaml --exclude-tags manual .
```

### 5. Ejecutar sin Config (Comportamiento por Defecto)

```bash
.\maestro\maestro\bin\maestro.bat test android/login/test/login-exitoso.test.yaml
```

Sin el `--config`, Maestro usa configuraciones por defecto y no respeta el orden de ejecución ni los tags definidos.

### 6. Ejecutar con Archivo de Config Alternativo

```bash
.\maestro\maestro\bin\maestro.bat test --config smoke-config.yaml .
```

Útil para tener diferentes configuraciones para diferentes escenarios (smoke tests, CI, PR, etc.).

## Estrategias de Ejecución Basadas en Config

### Smoke Tests (Pruebas de Humo)

El archivo `smoke-config.yaml` está configurado para ejecutar solo los tests críticos y happy path:

```yaml
flows:
  - android/login/test/*.test.yaml
  - android/cart/test/*.test.yaml
  - android/profile/test/*.test.yaml
  - android/register/test/*.test.yaml
includeTags:
  - happy_path
  - high_priority
excludeTags:
  - wip
  - manual
  - deprecated
  - low_priority
executionOrder:
  continueOnFailure: false
  flowsOrder:
    - android/login/test/login-exitoso
    - android/register/test/register-exitoso
    - android/cart/test/comprar-producto
    - android/profile/test/actualizar-perfil
testOutputDir: evidencias/smoke
```

Ejecutar:
```bash
.\maestro\maestro\bin\maestro.bat test --config smoke-config.yaml .
```

### Regression Suite
Usa el config.yaml principal:
```bash
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags regression .
```

### Por Módulo
```bash
# Solo login
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags login .

# Solo carrito
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags carrito .
```

### Por Prioridad
```bash
# Alta prioridad
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags high_priority .

# Media y alta prioridad
.\maestro\maestro\bin\maestro.bat test --config config.yaml --include-tags high_priority,medium_priority .
```

## Verificación de Evidencias

Después de ejecutar los tests, las evidencias se encuentran en:
```
evidencias/maestro/
└── [timestamp]/
    ├── screenshots/
    ├── logs/
    └── metadata/
```