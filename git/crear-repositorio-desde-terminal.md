# Crear y Conectar un Repositorio GitHub desde la Terminal

## Pasos Completos Después de `git init`

### 1. Configurar Credenciales (si aún no lo has hecho)

```bash
# Configurar nombre de usuario y email
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"
```

### 2. Crear el Repositorio Remoto en GitHub

**Opción A: Usando GitHub CLI (`gh`)**

```bash
# Instalar GitHub CLI si no está instalado
# macOS
brew install gh

# Autenticarse
gh auth login

# Crear un nuevo repositorio remoto
gh repo create nombre-repositorio --public
# O para hacerlo privado:
# gh repo create nombre-repositorio --private
```

**Opción B: Usando curl y la API de GitHub (necesita token)**

```bash
# Crear un repositorio vía API (requiere token personal)
curl -u username:token https://api.github.com/user/repos -d '{"name":"nombre-repositorio", "private": false}'
```

### 3. Agregar Archivos al Repositorio Local

```bash
# Crear un archivo README.md inicial
echo "# Mi Proyecto" > README.md

# Agregar todos los archivos al staging
git add .

# Hacer el primer commit
git commit -m "Commit inicial"
```

### 4. Conectar el Repositorio Local con el Remoto

```bash
# Agregar el repositorio remoto
git remote add origin https://github.com/tu-usuario/nombre-repositorio.git

# Verificar que se agregó correctamente
git remote -v
```

### 5. Subir el Código al Repositorio Remoto

```bash
# Establecer la rama principal (para Git 2.28+)
git branch -M main

# Subir los cambios
git push -u origin main
```

## Autenticación

### Métodos de Autenticación Recomendados

**1. Token de Acceso Personal (PAT)**

```bash
# Cuando uses HTTPS, en lugar de contraseña, usa un PAT
# URL con PAT incorporado
git remote add origin https://username:token@github.com/username/repo.git

# O configura las credenciales para que las almacene
git config credential.helper store
# Luego al hacer push, introduce tu PAT como contraseña
```

**2. SSH**

```bash
# Generar una clave SSH (si no tienes una)
ssh-keygen -t ed25519 -C "tu.email@ejemplo.com"

# Iniciar ssh-agent
eval "$(ssh-agent -s)"

# Agregar la clave al agente
ssh-add ~/.ssh/id_ed25519

# Mostrar la clave pública para agregarla en GitHub
cat ~/.ssh/id_ed25519.pub
# Copia esta clave y agrégala en GitHub > Settings > SSH and GPG keys

# Cambiar el remote a SSH
git remote set-url origin git@github.com:tu-usuario/nombre-repositorio.git
```

## Flujo Completo en una Secuencia

```bash
# Iniciar repositorio local
git init

# Configurar credenciales (si es la primera vez)
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"

# Crear repo en GitHub usando GitHub CLI
gh repo create nombre-repositorio --public

# Crear archivos iniciales
echo "# Nombre del Proyecto" > README.md

# Hacer commit inicial
git add .
git commit -m "Commit inicial"

# Conectar con el remoto
git remote add origin git@github.com:tu-usuario/nombre-repositorio.git

# Establecer rama principal
git branch -M main

# Subir al remoto
git push -u origin main
```

## Consejos Adicionales

### Si quieres crear un `.gitignore` automáticamente

```bash
# Para un proyecto Node.js
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/master/Node.gitignore

# Para un proyecto Python
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/master/Python.gitignore

# Para otros tipos, reemplaza el nombre del archivo en la URL
```

### Si deseas crear una licencia

```bash
# Crear una licencia MIT
curl -o LICENSE https://raw.githubusercontent.com/licenses/license-templates/master/templates/mit.txt
```

### Para trabajar con ramas

```bash
# Crear y cambiar a una nueva rama
git checkout -b feature/nueva-funcionalidad

# Subir la nueva rama al remoto
git push -u origin feature/nueva-funcionalidad
```

### Para verificar el estado actual

```bash
# Ver estado de archivos
git status

# Ver historial de commits
git log --oneline
```

Con estos pasos, habrás creado un repositorio en GitHub y lo habrás conectado correctamente con tu repositorio local, todo desde la terminal.
