# Guía Completa: Git Flow, Conventional Commits y Automatización con GitHub

Esta guía te proporcionará una estrategia completa para implementar buenas prácticas de gestión de código con Git, enfocándose en Git Flow, convenciones de nombrado, y automatización con GitHub.

## Índice
1. [Git Flow: Estructura de Ramas](#1-git-flow-estructura-de-ramas)
2. [Conventional Commits](#2-conventional-commits)
3. [Integración de Git Flow y Conventional Commits](#3-integración-de-git-flow-y-conventional-commits)
4. [Automatización con GitHub Actions](#4-automatización-con-github-actions)
5. [Etiquetas y Versionado Semántico](#5-etiquetas-y-versionado-semántico)
6. [Herramientas de Apoyo](#6-herramientas-de-apoyo)
7. [Plantillas y Documentación](#7-plantillas-y-documentación)

## 1. Git Flow: Estructura de Ramas

Git Flow es un modelo de ramificación que define una estructura estricta para la gestión de ramas en un proyecto.

### Ramas Principales
- **`main`/`master`**: Código en producción, estable.
- **`develop`**: Rama de desarrollo principal, contiene cambios aprobados para la próxima versión.

### Ramas de Soporte
- **`feature/nombre-funcionalidad`**: Para desarrollar nuevas funcionalidades.
- **`hotfix/nombre-corrección`**: Para corregir errores críticos en producción.
- **`release/x.y.z`**: Para preparar una nueva versión para producción.
- **`bugfix/nombre-error`**: Para corregir errores no críticos encontrados en desarrollo.
- **`docs/nombre-documentación`**: Para cambios exclusivamente en documentación.

### Flujo Básico de Trabajo

1. **Desarrollo de funcionalidades**:
   ```bash
   # Crear una rama de funcionalidad desde develop
   git checkout develop
   git checkout -b feature/nueva-funcionalidad
   
   # Después de completar la funcionalidad
   git checkout develop
   git merge --no-ff feature/nueva-funcionalidad
   git push origin develop
   git branch -d feature/nueva-funcionalidad
   ```

2. **Preparación de versiones**:
   ```bash
   # Crear rama de versión desde develop
   git checkout develop
   git checkout -b release/1.0.0
   
   # Ajustes finales, correcciones menores, cambios de versión
   # ...
   
   # Finalizar la versión
   git checkout main
   git merge --no-ff release/1.0.0
   git tag -a v1.0.0 -m "Versión 1.0.0"
   
   git checkout develop
   git merge --no-ff release/1.0.0
   
   git branch -d release/1.0.0
   ```

3. **Correcciones urgentes**:
   ```bash
   # Crear rama de hotfix desde main/master
   git checkout main
   git checkout -b hotfix/error-crítico
   
   # Aplicar la corrección
   # ...
   
   # Finalizar el hotfix
   git checkout main
   git merge --no-ff hotfix/error-crítico
   git tag -a v1.0.1 -m "Versión 1.0.1"
   
   git checkout develop
   git merge --no-ff hotfix/error-crítico
   
   git branch -d hotfix/error-crítico
   ```

## 2. Conventional Commits

Los Conventional Commits proporcionan un formato estandarizado para los mensajes de commit, facilitando la generación automática de changelogs y el control de versiones semántico.

### Estructura Básica
```
<tipo>[ámbito opcional]: <descripción>

[cuerpo opcional]

[nota de pie opcional]
```

### Tipos Principales
- **`feat:`** - Nueva funcionalidad
- **`fix:`** - Corrección de errores
- **`docs:`** - Cambios en documentación
- **`style:`** - Cambios que no afectan el código (formato, espacios, etc.)
- **`refactor:`** - Refactorización de código sin cambios funcionales
- **`perf:`** - Mejoras de rendimiento
- **`test:`** - Adición o corrección de pruebas
- **`build:`** - Cambios en el sistema de construcción
- **`ci:`** - Cambios en la configuración de integración continua
- **`chore:`** - Tareas rutinarias sin cambios en código productivo

### Ejemplos
```
feat(auth): implementar autenticación con Google

fix(api): corregir error 500 al enviar formulario vacío

docs(readme): actualizar instrucciones de instalación

refactor(usuarios): simplificar lógica de validación

test(login): agregar pruebas para casos de error
```

### Notas Especiales
- **Breaking Changes**: Se indican con `!` después del tipo o con `BREAKING CHANGE:` en el cuerpo
  ```
  feat!: cambiar API de autenticación
  
  BREAKING CHANGE: El endpoint de autenticación ahora requiere API key
  ```

## 3. Integración de Git Flow y Conventional Commits

La combinación de ambos enfoques crea un sistema robusto de gestión de código:

### Nomenclatura de Ramas + Conventional Commits

1. **Feature Branch**:
   - Nombrado: `feature/auth-google`
   - Commits: 
     ```
     feat(auth): agregar botón de login con Google
     feat(auth): implementar callback OAuth
     test(auth): agregar pruebas para login de Google
     ```
   - PR/Merge: `feat: implementar autenticación con Google`

2. **Hotfix Branch**:
   - Nombrado: `hotfix/login-500-error`
   - Commits:
     ```
     fix(auth): corregir error 500 en validación de credenciales
     test(auth): agregar caso de prueba para credenciales inválidas
     ```
   - PR/Merge: `fix: corregir error 500 en sistema de login`

3. **Release Branch**:
   - Nombrado: `release/1.2.0`
   - Commits:
     ```
     chore(release): actualizar versión a 1.2.0
     docs: actualizar changelog para v1.2.0
     fix(release): corregir error en manifiesto
     ```
   - PR/Merge: `chore(release): versión 1.2.0`

## 4. Automatización con GitHub Actions

GitHub Actions permite automatizar diversos aspectos del ciclo de desarrollo:

### Verificación de Conventional Commits

Archivo: `.github/workflows/commit-lint.yml`
```yaml
name: Lint Commit Messages
on: [pull_request]

jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v5
```

### Generación Automática de Changelog

Archivo: `.github/workflows/changelog.yml`
```yaml
name: Generate Changelog
on:
  push:
    tags:
      - 'v*'

jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - name: Generate changelog
        id: changelog
        uses: metcalfc/changelog-generator@v4.0.1
        with:
          myToken: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: ${{ steps.changelog.outputs.changelog }}
```

### Generación de Versiones Automáticas

Archivo: `.github/workflows/release.yml`
```yaml
name: Release
on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - name: Semantic Release
        uses: semantic-release/semantic-release@v19
        with:
          branches: ['main']
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 5. Etiquetas y Versionado Semántico

El versionado semántico (SemVer) es un sistema de numeración que refleja la naturaleza de los cambios:

### Formato
```
MAJOR.MINOR.PATCH[-prerelease][+metadata]
```

- **MAJOR**: Cambios incompatibles con versiones anteriores
- **MINOR**: Nuevas funcionalidades compatibles con versiones anteriores
- **PATCH**: Correcciones de errores compatibles con versiones anteriores

### Relación con Conventional Commits
- `fix:` incrementa PATCH (1.0.0 → 1.0.1)
- `feat:` incrementa MINOR (1.0.0 → 1.1.0)
- `feat!:` o `BREAKING CHANGE` incrementa MAJOR (1.0.0 → 2.0.0)

### Etiquetas en GitHub
Las etiquetas (labels) ayudan a categorizar issues y pull requests:

- **bug**: Error que necesita ser corregido
- **enhancement**: Nueva funcionalidad o mejora
- **documentation**: Cambios en documentación
- **good first issue**: Problemas adecuados para contribuyentes nuevos
- **help wanted**: Se busca ayuda externa
- **breaking-change**: Indica cambios que romperán compatibilidad

## 6. Herramientas de Apoyo

Estas herramientas te ayudarán a mantener estas prácticas:

### Herramientas de Línea de Comandos
- **[gitflow-avh](https://github.com/petervanderdoes/gitflow-avh)**: Implementación mejorada de Git Flow
  ```bash
  # Instalación en macOS
  brew install git-flow-avh
  
  # Inicialización
  git flow init
  
  # Uso
  git flow feature start nueva-funcionalidad
  git flow feature finish nueva-funcionalidad
  ```

- **[Commitizen](https://github.com/commitizen/cz-cli)**: Asistente interactivo para crear commits
  ```bash
  # Instalación
  npm install -g commitizen cz-conventional-changelog
  echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
  
  # Uso (en lugar de git commit)
  git add .
  cz
  ```

### Herramientas de Validación
- **[commitlint](https://commitlint.js.org/)**: Verifica que los commits sigan la convención
  ```bash
  # Instalación
  npm install -g @commitlint/cli @commitlint/config-conventional
  echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
  
  # Uso con husky (git hooks)
  npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
  ```

- **[husky](https://typicode.github.io/husky/)**: Hooks de Git para validación automática
  ```bash
  # Instalación
  npm install husky --save-dev
  npx husky install
  ```

### Herramientas de Generación
- **[standard-version](https://github.com/conventional-changelog/standard-version)**: Automatiza versiones y changelog
  ```bash
  # Instalación
  npm install -g standard-version
  
  # Uso
  standard-version
  ```

## 7. Plantillas y Documentación

### Plantilla de Pull Request
Archivo: `.github/PULL_REQUEST_TEMPLATE.md`
```markdown
## Descripción
<!-- Describa brevemente los cambios que incluye este PR -->

## Tipo de Cambio
<!-- Marque las opciones que aplican -->
- [ ] 🚀 Nueva funcionalidad (feat)
- [ ] 🐛 Corrección de error (fix)
- [ ] 📚 Cambios en documentación (docs)
- [ ] ♻️ Refactorización (refactor)
- [ ] ✅ Cambios en tests (test)
- [ ] 🔧 Cambios en configuración (chore)

## Breaking Changes
<!-- ¿Este PR introduce cambios que rompen compatibilidad? -->
- [ ] Sí
- [ ] No

## Checklist
- [ ] He seguido las convenciones de código del proyecto
- [ ] He agregado pruebas para mis cambios
- [ ] He actualizado la documentación correspondiente
```

### Documentación de Flujo de Trabajo
Crea un archivo `CONTRIBUTING.md` en la raíz del proyecto explicando:

1. El flujo de trabajo de Git Flow
2. Las convenciones de commit
3. El proceso de revisión de código
4. Las convenciones de nombrado
5. El uso de herramientas automatizadas

### README del Proyecto
Incluye en tu README.md secciones sobre:

- Convenciones de desarrollo
- Enlaces a la documentación de contribución
- Badges de estado del proyecto (CI/CD, versión, etc.)

## Implementación Progresiva

Para implementar estas prácticas gradualmente:

### Fase 1: Fundamentos
1. Establecer la estructura de ramas (Git Flow)
2. Adoptar Conventional Commits
3. Crear plantillas de PR e issues

### Fase 2: Automatización Básica
1. Implementar linting de commits
2. Configurar hooks de pre-commit
3. Iniciar versionado semántico manual

### Fase 3: CI/CD Avanzado
1. Automatizar generación de changelog
2. Implementar releases automáticas
3. Configurar despliegue continuo

## Ejemplo de Implementación Práctica

Para un proyecto típico, podrías comenzar con este conjunto de comandos:

```bash
# Inicializar Git Flow
git flow init -d

# Configurar herramientas de commit
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

# Configurar validación de commits (en el proyecto)
npm init -y
npm install --save-dev @commitlint/cli @commitlint/config-conventional husky
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

# Crear carpetas para plantillas
mkdir -p .github
wget -O .github/PULL_REQUEST_TEMPLATE.md https://example.com/template.md

# Comenzar una nueva funcionalidad
git flow feature start mi-funcionalidad
# Hacer cambios...
git add .
cz  # Usar Commitizen en lugar de git commit
git flow feature finish mi-funcionalidad
```

Esta guía te proporcionará una base sólida para implementar prácticas profesionales de gestión de código, facilitando la colaboración, el mantenimiento y la automatización de tu proyecto a medida que crece.
