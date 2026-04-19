# Guía ampliada de estudio para la certificación de GitHub Actions

Esta guía toma como base el temario principal del examen y el resumen de alto nivel de `github-actions-exam-preparation-study-guide__3_.md`, pero lo convierte en un documento de estudio más completo. El objetivo es que puedas entender qué hace cada pieza de GitHub Actions, qué parámetros suelen aparecer en el examen y cómo reconocer la sintaxis correcta sin memorizar YAML de forma ciega.

Referencia principal de documentación oficial:

- https://docs.github.com/es/actions
- https://docs.github.com/es/actions/using-workflows/workflow-syntax-for-github-actions
- https://docs.github.com/es/actions/learn-github-actions/understanding-github-actions

## Cómo usar esta guía

- Lee cada dominio en orden. El examen mezcla conceptos, pero casi todo parte de entender bien workflows, actions, runners y seguridad.
- Fíjate en la diferencia entre lo que se define a nivel de workflow, job, step y action. Es una fuente constante de preguntas.
- En cada bloque hay ejemplos prácticos de código. No son decorativos: imitan patrones reales del examen.
- Cuando veas un parámetro, intenta responder tres preguntas: dónde se declara, a qué nivel aplica y qué efecto produce.

---

# Índice del temario

## Domain 1 - Author and Maintain Workflows (40%)

1. Eventos que disparan un workflow
2. Componentes de un workflow
3. Secrets, variables y workflow commands
4. Workflows para propósitos específicos
5. Gestión de workflow runs

## Domain 2 - Consume Workflows (20%)

1. Interpretación de efectos de un workflow
2. Identificación del evento que lo disparó
3. Lectura y análisis del archivo YAML
4. Diagnóstico de fallos
5. Acceso a logs y depuración
6. Uso de variables de entorno
7. Descarga de artifacts
8. Workflows deshabilitados frente a eliminados
9. Uso de workflows templados

## Domain 3 - Author and Maintain Actions (25%)

1. Tipos de actions
2. Troubleshooting de actions
3. Componentes de una action
4. Distribución, versionado y publicación

## Domain 4 - Manage GitHub Actions in the Enterprise (15%)

1. Distribución de actions y workflows
2. Reutilización de plantillas
3. Estrategias de repositorio y mantenimiento
4. Control de acceso a actions
5. Políticas organizacionales
6. Gestión de runners
7. Gestión de secrets a nivel empresa

---

# Domain 1 - Author and Maintain Workflows (40%)

Este dominio es el más importante del examen. Si entiendes bien cómo se definen, disparan, filtran y ejecutan los workflows, buena parte del resto del temario resulta más sencilla.

## 1. Eventos que disparan un workflow

Un workflow se define en `.github/workflows/*.yml` y se activa con la clave `on`. Esa clave acepta un evento único, una lista de eventos o un mapa con configuración avanzada.

### Formas válidas de `on`

```yaml
on: push
```

```yaml
on: [push, pull_request]
```

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

### Eventos más importantes para examen

#### `push`

Se dispara cuando se hace push a una rama o tag. No usa `types`.

Parámetros más importantes:

- `branches`: limita el disparo del workflow a una o varias ramas concretas. Se usa cuando quieres CI solo en ramas como `main`, `develop` o `release/*`.
- `branches-ignore`: excluye ramas del disparo. Es útil cuando el workflow debe ejecutarse casi siempre, salvo en ramas técnicas o temporales.
- `tags`: limita el disparo a pushes sobre tags. Se usa mucho en versionado y publicaciones, por ejemplo con patrones como `v*`.
- `tags-ignore`: evita que el workflow se lance para ciertos tags. Sirve para excluir tags internos o de prueba.
- `paths`: hace que el workflow solo se dispare si el push modifica archivos o carpetas concretas. Es una optimización habitual para monorepos o pipelines parciales.
- `paths-ignore`: evita el disparo si los únicos cambios pertenecen a rutas concretas, como documentación o ficheros no funcionales.

```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*']
    paths:
      - 'src/**'
      - '.github/workflows/**'
```

Regla de examen: `push` y `pull_request` no filtran igual sobre el mismo contenido. En `push` se analiza lo que se sube; en `pull_request`, la actividad de la PR.

#### `pull_request`

Se activa por actividad relacionada con una pull request. Admite `types`.

Tipos frecuentes:

- `opened`: la pull request se acaba de crear. Suele usarse para validaciones iniciales.
- `edited`: se han cambiado título, descripción u otros metadatos de la PR. No siempre interesa para CI pesada.
- `reopened`: una PR cerrada vuelve a abrirse. Sirve para relanzar validaciones.
- `synchronize`: han llegado nuevos commits a la rama origen. Es el tipo más importante para CI continua en PR.
- `closed`: la PR se ha cerrado, con merge o sin merge. Se usa en automatismos de cierre o limpieza.
- `ready_for_review`: una PR draft pasa a estar lista para revisión. Muy útil para empezar validaciones costosas solo cuando ya procede revisar.
- `review_requested`: se solicita revisión a una persona o equipo. Se usa más en automatización de gobierno que en compilación.
- `labeled`: se añade una etiqueta a la PR. Permite activar comportamientos por label, como despliegues o ejecuciones opcionales.
- `unlabeled`: se elimina una etiqueta. Suele usarse para revertir comportamientos condicionados por labels.

Filtros relevantes:

- `types`: define qué actividades de la PR interesan realmente. Evita lanzar el workflow en eventos irrelevantes.
- `branches`: filtra por rama base de la pull request, no por la rama origen. Normalmente se usa para validar PR que apuntan a `main` o `develop`.
- `branches-ignore`: excluye ramas base concretas cuando la regla general es ejecutar el workflow en casi todas.
- `paths`: restringe la ejecución a cambios en determinados archivos o carpetas afectados por la PR.
- `paths-ignore`: evita ejecuciones cuando la PR solo toca contenido que no debe activar la pipeline.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
    branches: [main]
    paths:
      - 'app/**'
      - 'package.json'
```

Punto importante: `synchronize` es el tipo más común para CI sobre una PR, porque representa nuevos commits sobre la rama del pull request.

#### `pull_request_target`

Es parecido a `pull_request`, pero se ejecuta en el contexto del repositorio destino. Tiene implicaciones de seguridad muy relevantes porque puede acceder a permisos y secrets del repo base si se configura mal.

```yaml
on:
  pull_request_target:
    types: [opened, synchronize]
```

Regla de examen: es más delicado que `pull_request`. Nunca se debe ejecutar código no confiable de forks con permisos elevados sin controles.

#### `workflow_dispatch`

Permite ejecución manual desde la UI, API o CLI. Admite `inputs`.

Tipos de input útiles para examen:

- `string`: valor de texto libre. Es el tipo más flexible y el más utilizado para nombres, versiones o rutas.
- `choice`: obliga al usuario a elegir entre opciones predefinidas. Reduce errores y estandariza ejecuciones manuales.
- `boolean`: representa verdadero o falso. Se usa para activar o desactivar comportamientos como `dry_run`.
- `number`: recibe valores numéricos. Puede usarse para repeticiones, lotes o umbrales.
- `environment`: permite seleccionar un environment existente. Es útil cuando quieres conectar la ejecución manual con entornos protegidos.

Propiedades comunes de un input:

- `description`: texto que explica al usuario para qué sirve el input. En la práctica mejora mucho la usabilidad del formulario manual.
- `required`: indica si ese valor debe proporcionarse obligatoriamente. Si es `true`, no puedes lanzar el workflow sin rellenarlo.
- `default`: valor por defecto usado cuando el usuario no informa uno. Es útil para simplificar ejecuciones frecuentes.
- `type`: define la clase de dato esperada y cómo se mostrará en la interfaz.
- `options`: lista de valores válidos para `choice`. Sirve para guiar la selección y evitar texto libre.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Entorno de despliegue
        required: true
        type: choice
        options:
          - dev
          - test
          - prod
      dry_run:
        description: Ejecutar sin desplegar
        required: false
        default: false
        type: boolean
      version:
        description: Versión a desplegar
        required: true
        type: string
```

Uso:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Entorno ${{ inputs.environment }}"
      - run: echo "Dry run ${{ inputs.dry_run }}"
      - run: echo "Versión ${{ inputs.version }}"
```

#### `schedule`

Permite ejecuciones periódicas mediante expresiones cron en UTC. La frecuencia mínima son 5 minutos.

```yaml
on:
  schedule:
    - cron: '0 2 * * *'
    - cron: '30 6 * * 1-5'
```

Reglas útiles:

- Usa sintaxis cron clásica de 5 campos.
- Puede sufrir retrasos en horas de alta carga.
- En repositorios públicos inactivos puede deshabilitarse tras largo tiempo sin actividad.

#### `repository_dispatch`

Sirve para lanzar workflows desde sistemas externos mediante API. Usa `types` personalizados.

```yaml
on:
  repository_dispatch:
    types: [deploy-request, nightly-validation]
```

Acceso al payload:

```yaml
steps:
  - run: echo "Servicio ${{ github.event.client_payload.service }}"
```

#### `workflow_run`

Se usa para encadenar workflows a partir del resultado de otro workflow.

Parámetros:

- `workflows`: lista de nombres de workflows cuya ejecución actúa como disparador. Deben coincidir con el `name` del workflow origen.
- `types`: define en qué momento del workflow origen quieres reaccionar. `requested` actúa al solicitarse, `in_progress` durante la ejecución y `completed` al terminar.
- `branches`: filtra por la rama sobre la que se ejecutó el workflow origen. Es muy útil para encadenar despliegues solo desde `main`.

```yaml
on:
  workflow_run:
    workflows: ['CI']
    types: [completed]
    branches: [main]
```

Caso típico: lanzar un despliegue solo si otro workflow de CI ha terminado correctamente.

```yaml
jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Despliegue tras CI correcta"
```

#### `workflow_call`

Convierte un workflow en reutilizable. Es clave en organizaciones grandes.

Parámetros permitidos:

- `inputs`: valores que el workflow reutilizable expone para ser configurado desde el workflow consumidor.
- `outputs`: datos que el workflow reutilizable devuelve al workflow que lo llama.
- `secrets`: secretos que el workflow reutilizable necesita y que deben proporcionarse de forma explícita o heredarse.

Tipos de input:

- `string`: adecuado para nombres, versiones, comandos o rutas.
- `boolean`: útil para activar o no ciertas ramas lógicas del workflow reutilizable.
- `number`: sirve para contadores, umbrales o cantidades configurables.

```yaml
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      run-tests:
        required: false
        type: boolean
        default: true
    secrets:
      npm_token:
        required: true
```

Workflow consumidor:

```yaml
jobs:
  use-reusable:
    uses: org/shared/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'
      run-tests: true
    secrets:
      npm_token: ${{ secrets.NPM_TOKEN }}
```

#### `release`

Se usa en automatizaciones de versionado, publicación y despliegue.

Tipos frecuentes:

- `published`: la release ha quedado publicada y suele ser el punto más común para automatizar despliegues o publicaciones.
- `released`: indica una release final publicada, normalmente sin carácter de prerelease.
- `prereleased`: se ha publicado una prerelease. Es útil para entornos de validación previos a producción.
- `created`: la release se ha creado, pero no necesariamente está publicada para consumo final.
- `edited`: se han modificado campos de la release, como notas o metadatos.
- `deleted`: la release se ha eliminado.
- `unpublished`: una release publicada ha sido retirada.

```yaml
on:
  release:
    types: [published]
```

#### Eventos complementarios que conviene reconocer

- `issues`
- `issue_comment`
- `discussion`
- `discussion_comment`
- `check_run`
- `check_suite`
- `create`
- `delete`
- `fork`
- `watch`
- `deployment`
- `deployment_status`

Ejemplo con `issues`:

```yaml
on:
  issues:
    types: [opened, edited, closed, labeled]
```

### Resumen de examen sobre eventos

- `push` no usa `types`.
- `workflow_dispatch` es manual.
- `schedule` usa cron en UTC.
- `workflow_run` encadena workflows.
- `workflow_call` reutiliza workflows.
- `repository_dispatch` integra sistemas externos.
- `pull_request_target` implica más riesgo de seguridad que `pull_request`.

## 2. Componentes de un workflow

Un workflow se compone de metadatos, permisos, disparadores y trabajos. La estructura mínima suele incluir `name`, `on` y `jobs`.

```yaml
name: CI Node

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

### Claves globales más importantes

#### `name`

Nombre legible del workflow.

```yaml
name: Build and test
```

#### `run-name`

Nombre dinámico visible en la ejecución. Muy útil para identificar runs manuales o despliegues.

```yaml
run-name: Deploy ${{ inputs.version }} to ${{ inputs.environment }} by @${{ github.actor }}
```

#### `permissions`

Define permisos del token `GITHUB_TOKEN`. Es muy preguntado en examen.

Permisos comunes:

- `actions`: permite operar sobre ejecuciones, cancelaciones u otros elementos relacionados con Actions.
- `attestations`: controla la generación o lectura de attestations cuando se trabaja con supply chain security.
- `checks`: da acceso a crear o actualizar check runs y resultados de validación.
- `contents`: permite leer o modificar contenido del repositorio. Es de los permisos más usados.
- `deployments`: habilita operaciones sobre despliegues y sus estados.
- `discussions`: da acceso a discusiones del repositorio si el workflow interactua con ellas.
- `id-token`: permite emitir un OIDC token para autenticación federada con proveedores cloud. Es clave en despliegues sin secretos fijos.
- `issues`: habilita lectura o escritura sobre incidencias.
- `packages`: necesario para publicar o leer paquetes en GitHub Packages o GHCR.
- `pages`: requerido en despliegues hacia GitHub Pages.
- `pull-requests`: permite comentar, actualizar o interactuar con pull requests.
- `security-events`: necesario para subir resultados de herramientas como CodeQL.
- `statuses`: permite publicar estados clásicos de commit.

Valores habituales:

- `read`: concede solo lectura. Debe ser la opción por defecto cuando no necesitas modificar nada.
- `write`: concede escritura además de lectura. Solo conviene usarlo si el workflow realmente crea, publica o actualiza recursos.
- `none`: revoca el permiso para ese alcance concreto, reforzando el principio de mínimo privilegio.

```yaml
permissions:
  contents: read
  packages: write
  id-token: write
```

Regla importante: cuanto menos permiso, mejor. El examen suele premiar mínimo privilegio.

#### `env`

Declara variables de entorno a nivel de workflow.

```yaml
env:
  NODE_ENV: test
  CI: true
```

#### `defaults`

Define configuración por defecto para comandos `run`.

Parámetros:

- `run.shell`: define la shell por defecto para todos los pasos `run`, por ejemplo `bash`, `pwsh` o `cmd`. Sirve para homogeneizar la ejecución.
- `run.working-directory`: establece el directorio base desde el que se lanzan los comandos `run`. Es útil en monorepos o subproyectos.

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./app
```

#### `concurrency`

Controla ejecuciones simultáneas. Muy útil para despliegues.

Parámetros:

- `group`: nombre lógico del grupo de concurrencia. Las ejecuciones con el mismo grupo se consideran incompatibles entre sí.
- `cancel-in-progress`: si está a `true`, cancela la ejecución anterior en curso del mismo grupo cuando llega una nueva.

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

#### `jobs`

Contiene los trabajos del workflow. Cada job corre en un runner independiente.

### Componentes de un job

#### `jobs.<id>.name`

Nombre legible del job.

```yaml
jobs:
  build:
    name: Build application
```

#### `runs-on`

Define el runner. Puede ser un string único o una lista de labels.

```yaml
runs-on: ubuntu-latest
```

```yaml
runs-on: [self-hosted, linux, x64, prod]
```

GitHub-hosted comunes:

- `ubuntu-latest`
- `windows-latest`
- `macos-latest`

Self-hosted:

- `self-hosted`
- labels personalizadas como `linux`, `gpu`, `prod`, `build-large`

#### `needs`

Define dependencias entre jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy"
```

Si `build` falla, `deploy` no corre salvo que uses una condición distinta con `always()`.

#### `if`

Condiciona la ejecución del job o step.

Funciones frecuentes:

- `success()`
- `failure()`
- `always()`
- `cancelled()`

```yaml
if: ${{ github.ref == 'refs/heads/main' && success() }}
```

#### `strategy.matrix`

Permite ejecutar combinaciones de configuraciones.

Parámetros:

- `matrix`: define los ejes de combinación, por ejemplo sistemas operativos, versiones de lenguaje o modos de ejecución.
- `include`: agrega combinaciones extra o completa propiedades adicionales en algunas filas de la matriz.
- `exclude`: elimina combinaciones que de otra forma se generarían automáticamente.
- `fail-fast`: si está a `true`, cancela el resto de combinaciones al fallar una. Si está a `false`, deja que todas terminen.
- `max-parallel`: limita cuántas combinaciones de la matriz pueden ejecutarse simultáneamente.

```yaml
strategy:
  fail-fast: false
  max-parallel: 3
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]
    include:
      - os: ubuntu-latest
        node: 22
        experimental: true
    exclude:
      - os: windows-latest
        node: 18
```

Uso:

```yaml
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node }}
```

#### `container`

Ejecuta el job dentro de un contenedor.

Parámetros:

- `image`: imagen contenedora usada para ejecutar el job.
- `credentials`: credenciales para autenticarse contra un registro privado antes de descargar la imagen.
- `env`: variables de entorno disponibles dentro del contenedor del job.
- `ports`: puertos que se exponen cuando el contenedor necesita comunicación de red específica.
- `volumes`: montajes adicionales entre host y contenedor para compartir datos persistentes o directorios.
- `options`: argumentos extra de ejecución de Docker, por ejemplo límites de CPU o memoria.

```yaml
container:
  image: node:20
  env:
    NODE_ENV: test
  options: --cpus 2
```

#### `services`

Define contenedores de servicio, por ejemplo una base de datos para test de integración.

Parámetros:

- `image`: imagen del servicio auxiliar, como una base de datos o Redis.
- `env`: variables de configuración que el servicio necesita para arrancar correctamente.
- `ports`: mapeo de puertos para que el job principal pueda acceder al servicio.
- `volumes`: volúmenes necesarios para persistencia o inicialización avanzada.
- `options`: argumentos extra del contenedor, a menudo usados para health checks.
- `credentials`: autenticación para descargar la imagen desde registros privados.

```yaml
services:
  postgres:
    image: postgres:16
    env:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    ports:
      - 5432:5432
    options: >-
      --health-cmd="pg_isready -U app"
      --health-interval=10s
      --health-timeout=5s
      --health-retries=5
```

#### `environment`

Asocia el job a un environment, por ejemplo `production`.

```yaml
environment:
  name: production
  url: https://app.example.com
```

Esto puede activar approvals, protection rules y environment secrets.

#### `outputs`

Expone datos del job para otros jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.meta.outputs.image_tag }}
    steps:
      - id: meta
        run: echo "image_tag=v1.2.3" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.image_tag }}"
```

### Componentes de un step

Un step puede ejecutar shell con `run` o invocar una action con `uses`.

#### `uses`

Usa una action existente.

```yaml
- uses: actions/checkout@v4
```

Buenas prácticas de versionado:

- aceptable: `@v4`
- más seguro: `@<sha>`
- evitar en producción: `@main`

#### `run`

Ejecuta comandos shell.

```yaml
- run: npm ci
```

#### `with`

Pasa parámetros a una action.

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: npm
```

#### `env`

Variables de entorno del step.

```yaml
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### `shell`

Sobrescribe la shell.

```yaml
- run: echo Hello
  shell: bash
```

#### `working-directory`

Directorio de trabajo del step.

```yaml
- run: npm test
  working-directory: ./frontend
```

#### `timeout-minutes`

Limita duración.

```yaml
- run: npm test
  timeout-minutes: 15
```

#### `continue-on-error`

Permite continuar si el step falla.

```yaml
- run: npm run lint
  continue-on-error: true
```

### GitHub-hosted vs self-hosted runners

#### GitHub-hosted

Ventajas:

- sin mantenimiento de infraestructura
- provisión rápida
- imágenes estandarizadas

Limitaciones:

- menos control de red
- menos personalización
- tiempos y limites segun plan

#### Self-hosted

Ventajas:

- acceso a red interna
- control total de herramientas
- hardware especializado

Riesgos:

- mantenimiento
- seguridad del host
- limpieza entre ejecuciones

Pregunta clásica: si necesitas acceso a recursos internos de empresa o software no disponible en runners gestionados, probablemente necesitas self-hosted.

## 3. Secrets, variables y workflow commands

### `secrets`

Los secrets son valores cifrados que GitHub enmascara en logs. Se usan como `secrets.NOMBRE`.

Scopes que debes dominar:

- repository secrets: secretos visibles solo dentro del repositorio actual. Son el alcance más habitual para pipelines de un solo repo.
- environment secrets: secretos ligados a un environment concreto, como `production` o `staging`. Solo se entregan a jobs que usan ese environment.
- organization secrets: secretos compartibles entre varios repositorios de la organización, con posibilidad de limitar a repos concretos.

Ejemplo:

```yaml
steps:
  - run: echo "Autenticando"
    env:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Puntos de examen:

- En forks, los secrets no suelen estar disponibles por seguridad.
- Los environment secrets solo están disponibles si el job usa ese environment.
- `GITHUB_TOKEN` es automático y no se configura como secret manual en la mayoría de casos.

### Variables: `env`, `vars` y variables por defecto

#### `env`

Variables definidas por el workflow, job o step.

```yaml
env:
  APP_NAME: my-service
```

#### `vars`

Configuration variables gestionadas desde GitHub. Son distintas de `secrets` porque no van cifradas.

```yaml
steps:
  - run: echo "Region ${{ vars.AZURE_REGION }}"
```

#### Variables por defecto frecuentes

- `GITHUB_REPOSITORY`
- `GITHUB_SHA`
- `GITHUB_REF`
- `GITHUB_REF_NAME`
- `GITHUB_WORKFLOW`
- `GITHUB_RUN_ID`
- `GITHUB_ACTOR`
- `RUNNER_OS`
- `RUNNER_ARCH`
- `CI`

Ejemplo:

```yaml
- run: |
    echo "Repo: $GITHUB_REPOSITORY"
    echo "SHA: $GITHUB_SHA"
    echo "Runner: $RUNNER_OS"
```

### Workflow commands

Son comandos especiales escritos a stdout que GitHub interpreta.

#### Crear variables de entorno para pasos posteriores

```yaml
- run: echo "APP_VERSION=1.2.3" >> "$GITHUB_ENV"
```

#### Definir output de step

```yaml
- id: meta
  run: echo "image=ghcr.io/acme/app:1.2.3" >> "$GITHUB_OUTPUT"
```

#### Enmascarar valores

```yaml
- run: echo "::add-mask::${{ secrets.API_KEY }}"
```

#### Agrupar logs

```yaml
- run: |
    echo "::group::Instalacion"
    npm ci
    echo "::endgroup::"
```

#### Anotaciones de log

```yaml
- run: echo "::warning::Dependencia obsoleta detectada"
- run: echo "::error file=src/app.js,line=10::Fallo de compilacion"
- run: echo "::notice::Compilacion completada"
```

### Diferencia entre contexts más preguntados

- `github`: informacion del evento y del repositorio.
- `env`: variables declaradas en YAML o generadas con `GITHUB_ENV`.
- `vars`: configuration variables definidas en GitHub.
- `secrets`: valores cifrados.
- `runner`: caracteristicas del runner.
- `job`: datos del job.
- `steps`: outputs de steps anteriores.
- `matrix`: valores generados por la matriz.
- `needs`: outputs y resultados de jobs dependientes.

## 4. Workflows para propósitos específicos

### Scripts y automatizacion basica

Caso comun: lanzar scripts internos.

```yaml
name: Run maintenance script

on:
  workflow_dispatch:

jobs:
  script:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./scripts/maintenance.sh
```

### Publicación en GitHub Packages

```yaml
name: Publish package

on:
  push:
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: https://npm.pkg.github.com
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Conceptos a recordar:

- Necesitas `packages: write`.
- `setup-node` puede configurar el registro.
- La autenticación puede hacerse con `GITHUB_TOKEN` o PAT según el escenario.

### Publicación en GHCR

```yaml
name: Publish container image

on:
  push:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

Parámetros de interés en `docker/build-push-action`:

- `context`: carpeta o contexto de build que Docker utiliza como origen de archivos.
- `file`: ruta del `Dockerfile` si no se usa el nombre y ubicación por defecto.
- `push`: si está a `true`, sube la imagen resultante al registro remoto.
- `load`: carga la imagen construida en el daemon Docker local del runner en lugar de subirla.
- `tags`: lista de tags que recibirá la imagen, por ejemplo `latest`, `1.0.0` o tags por rama.
- `build-args`: argumentos enviados al proceso de build para parametrizar el Dockerfile.
- `cache-from`: origen de caché de compilación para acelerar builds posteriores.
- `cache-to`: destino donde se guarda la caché generada por el build.
- `platforms`: plataformas objetivo de la imagen, como `linux/amd64` o `linux/arm64`, especialmente útil en builds multiarch.

### Service containers

Muy habitual para bases de datos, Redis o servicios auxiliares.

```yaml
jobs:
  integration-tests:
    runs-on: ubuntu-latest
    services:
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
        env:
          REDIS_URL: redis://localhost:6379
```

### Labels para runners

```yaml
jobs:
  secure-build:
    runs-on: [self-hosted, linux, x64, secure]
```

La lista indica que el runner debe tener todas esas labels.

### CodeQL

Patrón típico de seguridad.

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '15 3 * * 1'

permissions:
  actions: read
  contents: read
  security-events: write

jobs:
  analyze:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        language: [javascript, python]
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
      - uses: github/codeql-action/autobuild@v3
      - uses: github/codeql-action/analyze@v3
```

### Releases y despliegues

```yaml
name: Release deploy

on:
  release:
    types: [published]

concurrency:
  group: production
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - run: echo "Deploying release ${{ github.event.release.tag_name }}"
```

### Pages y despliegue estático

Conviene reconocer este patrón aunque no sea el centro del examen.

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

## 5. Gestión de workflow runs

### Cache

La caché acelera ejecuciones reutilizando dependencias o artefactos intermedios. No es lo mismo que artifacts.

Parámetros clave de `actions/cache`:

- `path`: rutas que quieres cachear, como dependencias o directorios de herramientas.
- `key`: identificador único de la caché. Debe cambiar cuando cambian las dependencias relevantes.
- `restore-keys`: prefijos alternativos para recuperar caches parciales cuando no existe coincidencia exacta.
- `enableCrossOsArchive`: permite reutilizar ciertos artefactos de caché entre sistemas operativos compatibles.
- `fail-on-cache-miss`: si esta activo, hace fallar el step cuando no se encuentra cache.
- `lookup-only`: consulta si existe caché sin descargarla, útil para lógicas avanzadas.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

Regla de examen:

- Cache: optimiza rendimiento entre ejecuciones.
- Artifact: guarda ficheros de una ejecución para descargar o pasar entre jobs.

### Pasar datos entre jobs

La forma correcta es mediante outputs de job, no confiando en variables `env` entre runners distintos.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - id: version
        run: echo "value=1.0.${GITHUB_RUN_NUMBER}" >> "$GITHUB_OUTPUT"

  publish:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Publishing ${{ needs.build.outputs.version }}"
```

### Artifacts

#### Subida

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: dist-files
    path: dist/
    if-no-files-found: error
    retention-days: 7
```

#### Descarga

```yaml
- uses: actions/download-artifact@v4
  with:
    name: dist-files
    path: ./downloaded-dist
```

Parámetros frecuentes:

- `name`: nombre lógico del artifact. Es el identificador que luego usarás para descargarlo.
- `path`: ruta o patrón de archivos que quieres subir o descargar.
- `retention-days`: número de días que GitHub conservará ese artifact antes de eliminarlo.
- `if-no-files-found`: comportamiento cuando no existe ningún fichero que coincida, por ejemplo `error`, `warn` o `ignore`.
- `compression-level`: nivel de compresión aplicado al artifact, equilibrando tamaño y velocidad.

### Badges

Permiten mostrar el estado de un workflow en el README. No suelen implicar YAML, pero si reconocer el concepto.

Ejemplo de URL de badge:

```text
https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg
```

### Protecciones de entorno y approval gates

Los environments pueden exigir:

- revisores requeridos
- tiempo de espera
- restricciones de ramas
- secrets específicos del entorno

```yaml
jobs:
  deploy:
    environment: production
    runs-on: ubuntu-latest
    steps:
      - run: echo "Esperando aprobacion si aplica"
```

### Matrices

Puntos frecuentes de examen:

- por defecto, los jobs de matrix se expanden en paralelo
- `fail-fast` cancela el resto de combinaciones tras un fallo si está a `true`
- `include` agrega filas nuevas o completa datos
- `exclude` elimina combinaciones concretas

### Reintentos, cancelaciones y control de flujo

GitHub Actions no tiene un `retry` general declarativo para cualquier step, por eso se suelen ver patrones shell o acciones externas para reintentos. Lo importante en examen es reconocer que:

- `continue-on-error` no reintenta, solo deja continuar
- `timeout-minutes` corta la ejecución
- `concurrency` evita solapamiento
- `cancel-in-progress` cancela ejecuciones previas del mismo grupo

---

# Domain 2 - Consume Workflows (20%)

Este dominio evalúa si sabes interpretar y operar workflows existentes. No basta con escribir YAML: hay que leerlo, diagnosticarlo y entender sus efectos.

## 1. Interpretación de efectos de un workflow

Cuando te muestren un YAML, debes identificar:

- qué lo dispara
- qué jobs crea
- en qué orden se ejecutan
- si alguna condición evita su ejecución
- si publica paquetes, artifacts, releases o despliegues

Ejemplo:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

Interpretación correcta: el job o step solo corre en la rama `main`.

## 2. Identificación del evento que lo disparó

La variable clave es `github.event_name`.

```yaml
- run: echo "Evento ${{ github.event_name }}"
```

Otros datos importantes:

- `github.event`: payload completo
- `github.ref`: referencia actual
- `github.ref_name`: nombre corto de rama o tag
- `github.base_ref`: rama base en PR
- `github.head_ref`: rama origen en PR
- `github.actor`: usuario que dispara

## 3. Lectura y análisis del archivo YAML

Checklist mental para leer un workflow:

1. Mira `on`.
2. Mira `permissions`.
3. Revisa `env` y `defaults`.
4. Identifica cada `job` y su `runs-on`.
5. Observa `needs`.
6. Busca `if`.
7. Busca `uses` y `run`.
8. Revisa `strategy.matrix`, `environment`, `services` y `container`.

Errores típicos que debes detectar:

- mala indentación
- step fuera de `steps`
- `runs-on` mal situado
- output referenciado con id incorrecto
- uso de secret no disponible
- `needs` apuntando a job inexistente

## 4. Diagnóstico de fallos

### Tipos de fallo

#### Sintaxis YAML

El workflow ni siquiera arranca.

Ejemplos:

- mala indentación
- mapa mal formado
- mezcla inválida de arrays y objetos

#### Error de ejecución

El workflow arranca, pero falla un comando o una action.

Ejemplos:

- `npm test` devuelve exit code 1
- action inexistente o versión inválida
- falta de permisos

#### Error de lógica

El workflow corre, pero no hace lo esperado.

Ejemplos:

- condición `if` falsa
- filtro de `paths` demasiado restrictivo
- output vacío

#### Error de seguridad o permisos

Ejemplos:

- `Resource not accessible by integration`
- `GITHUB_TOKEN` sin `packages: write`
- secreto no disponible en fork

## 5. Acceso a logs y depuración

### Logs desde UI

Ruta normal:

- pestaña Actions
- seleccion del workflow
- seleccion del run
- seleccion del job
- seleccion del step

### Logs desde CLI y API

Comandos conocidos:

```bash
gh run list
gh run view <run-id>
gh run download <run-id>
```

### Debug logging

Flags habituales:

- `ACTIONS_STEP_DEBUG`
- `ACTIONS_RUNNER_DEBUG`

Ejemplo:

```yaml
env:
  ACTIONS_STEP_DEBUG: true
```

Es más común activarlos como secret o variable del repositorio para no tocar el YAML en cada depuración.

## 6. Uso de variables de entorno

Recuerda el alcance:

- workflow level
- job level
- step level

El alcance más cercano sobrescribe al más general.

```yaml
env:
  APP_ENV: dev

jobs:
  test:
    env:
      APP_ENV: test
    steps:
      - run: echo "$APP_ENV"
      - run: echo "$APP_ENV"
        env:
          APP_ENV: integration
```

Salida esperada por pasos:

- primer step: `test`
- segundo step: `integration`

## 7. Descarga de artifacts

Un artifact pertenece a una ejecución concreta. Puede descargarse desde la UI o dentro de otro job del mismo workflow.

Ejemplo de uso entre jobs:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: mkdir dist && echo hello > dist/file.txt
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  consume:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: ls -R
```

## 8. Workflows deshabilitados frente a eliminados

### Deshabilitado

- el archivo sigue existiendo
- el workflow no se ejecuta automáticamente
- puede volver a habilitarse

### Eliminado

- el archivo desaparece del repositorio
- deja de existir como definición activa

Punto de examen: deshabilitar no equivale a borrar.

## 9. Workflows templados

### Starter workflows

Son plantillas iniciales que GitHub ofrece o que una organización pública pone a disposición para acelerar la creación de workflows.

### Reusable workflows

No son plantillas estaticas, sino workflows reales invocados con `workflow_call`.

Diferencia clave de examen:

- starter workflow: se copia al repo
- reusable workflow: se invoca en runtime

---

# Domain 3 - Author and Maintain Actions (25%)

El examen diferencia claramente entre usar una action y crear una action.

## 1. Tipos de actions

### JavaScript action

Se ejecuta directamente con Node.js en el runner.

Ventajas:

- rapida
- buen acceso a toolkit oficial
- ideal para lógica reusable

Ejemplo de `action.yml`:

```yaml
name: Hello JS Action
description: Saluda con un nombre de entrada
inputs:
  name:
    description: Nombre a saludar
    required: true
runs:
  using: node20
  main: dist/index.js
```

Uso típico del toolkit:

```javascript
const core = require('@actions/core');

try {
  const name = core.getInput('name');
  core.info(`Hola ${name}`);
  core.setOutput('message', `Hola ${name}`);
} catch (error) {
  core.setFailed(error.message);
}
```

### Docker action

Se ejecuta dentro de un contenedor. Aporta aislamiento y control del entorno.

Ejemplo de `action.yml`:

```yaml
name: Docker Action
description: Ejecuta lógica dentro de contenedor
inputs:
  target:
    description: Destino
    required: true
runs:
  using: docker
  image: Dockerfile
  args:
    - ${{ inputs.target }}
```

Ejemplo de `Dockerfile`:

```dockerfile
FROM alpine:3.20
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

### Composite action

Encapsula varios steps reutilizables sin escribir una action en JS o Docker.

```yaml
name: Composite Node Setup
description: Instala dependencias y ejecuta test
inputs:
  node-version:
    description: Version de Node
    required: true
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
      shell: bash
    - run: npm test
      shell: bash
```

Regla de examen:

- JavaScript: action con lógica en Node.
- Docker: action con entorno encapsulado.
- Composite: varios steps YAML reutilizables.

## 2. Troubleshooting de actions

### Problemas habituales en JavaScript actions

- `main` apunta a fichero inexistente
- dependencias no empaquetadas
- runtime `node16` obsoleto frente a `node20`
- inputs no declarados pero usados
- no se genera `dist/` tras compilar TypeScript

### Problemas habituales en Docker actions

- Dockerfile no copia el entrypoint
- permisos de ejecución
- `ENTRYPOINT` incorrecto
- imagen demasiado pesada
- paths relativos mal resueltos

### Problemas habituales en composite actions

- olvidar `shell` en steps `run`
- usar contexts no disponibles
- intentar hacer cosas no soportadas por el modelo de composite

## 3. Componentes de una action

### Estructura minima recomendada

- `action.yml`
- codigo fuente o `Dockerfile`
- `README.md`

### Campos principales de `action.yml`

#### `name`

Nombre visible.

#### `description`

Descripción corta del comportamiento.

#### `author`

Autor de la action.

#### `branding`

Pensado para Marketplace.

```yaml
branding:
  icon: package
  color: blue
```

#### `inputs`

Parámetros de entrada.

Campos útiles:

- `description`: explica al consumidor de la action para qué sirve el input y cómo debe rellenarlo.
- `required`: obliga a proporcionar el valor para poder usar correctamente la action.
- `default`: valor aplicado si el consumidor no indica uno.
- `deprecationMessage`: mensaje mostrado cuando el input sigue existiendo por compatibilidad, pero ya no debería usarse.

```yaml
inputs:
  token:
    description: Token de acceso
    required: true
  mode:
    description: Modo de operacion
    required: false
    default: safe
```

#### `outputs`

Valores devueltos.

```yaml
outputs:
  image:
    description: Imagen generada
```

#### `runs`

Seccion obligatoria para indicar tipo de action.

Variantes:

- `using: node20`
- `using: docker`
- `using: composite`

Ejemplo JS:

```yaml
runs:
  using: node20
  main: dist/index.js
  post: dist/post.js
```

Ejemplo Docker:

```yaml
runs:
  using: docker
  image: Dockerfile
  args:
    - ${{ inputs.mode }}
```

Ejemplo Composite:

```yaml
runs:
  using: composite
  steps:
    - run: echo hello
      shell: bash
```

### Workflow commands y exit codes

Salida correcta moderna:

```bash
echo "result=ok" >> "$GITHUB_OUTPUT"
```

Fallo de action desde shell:

```bash
echo "Fallo controlado" >&2
exit 1
```

En JS, lo típico es:

```javascript
core.setFailed('Error de validacion');
```

## 4. Distribución, versionado y publicación

### Visibilidad

- pública
- privada
- interna para organización, según plan y configuración

### Marketplace

Requisitos habituales:

- repositorio publico
- `action.yml`
- `README.md`
- versionado con tags

### Estrategia de versionado

Recomendado:

- tag exacto: `v1.2.3`
- major estable: `v1`

Uso:

```yaml
- uses: owner/my-action@v1
```

Más seguro:

```yaml
- uses: owner/my-action@a1b2c3d4e5f6g7h8i9
```

### Buenas prácticas de publicación

- no usar `main` como referencia estable
- documentar inputs y outputs
- publicar ejemplos reales
- minimizar permisos requeridos
- usar release notes y tags claros

---

# Domain 4 - Manage GitHub Actions in the Enterprise (15%)

Este dominio se centra en gobierno, seguridad, escalabilidad y estandarización.

## 1. Distribución de actions y workflows

Patrones sanos para empresa:

- repositorio dedicado de actions reutilizables
- repositorio dedicado de reusable workflows
- versionado por tags mayores y menores
- documentación de consumo

Ejemplo de reusable workflow compartido:

```yaml
jobs:
  ci:
    uses: org/platform-workflows/.github/workflows/node-ci.yml@v1
    with:
      node-version: '20'
    secrets: inherit
```

## 2. Reutilización de plantillas

Las plantillas organizacionales viven en `.github/workflow-templates`.

Diferencia fundamental:

- plantilla: se copia y luego diverge
- reusable workflow: se mantiene centralizado

## 3. Estrategias de repositorio y mantenimiento

Practicas recomendadas:

- nombres claros: `ci.yml`, `cd.yml`, `security.yml`
- ramas protegidas para repos de plataforma
- revisiones obligatorias en workflows compartidos
- escaneo de seguridad sobre actions internas
- plan de deprecación y migración de versiones

## 4. Control de acceso a actions

GitHub permite políticas para:

- permitir todas las actions
- permitir solo actions de GitHub
- permitir solo una allow list concreta
- bloquear actions no aprobadas

Regla de examen: una política organizacional puede impedir el uso de una action aunque el YAML esté bien escrito.

## 5. Políticas organizacionales

Temas a dominar:

- permisos por defecto del `GITHUB_TOKEN`
- aprobación para workflows desde forks
- uso de actions de terceros
- uso de runners self-hosted
- acceso a environments

Ejemplo de minimizacion de permisos:

```yaml
permissions:
  contents: read
```

## 6. Gestión de runners

### Tipos

- GitHub-hosted
- self-hosted
- grupos de runners a nivel org o enterprise
- runners efímeros, según arquitectura elegida

### Runner groups

Sirven para controlar qué repositorios pueden usar qué runners.

### Labels

Ayudan a enrutar jobs a capacidades concretas.

```yaml
runs-on: [self-hosted, linux, docker, prod]
```

### Networking, proxies e IP allow lists

Debes reconocer estos escenarios:

- necesidad de salida a internet limitada
- acceso a servicios internos
- configuración de proxy corporativo
- whitelist de IPs para recursos protegidos

### Monitorización y troubleshooting

Problemas típicos:

- runner offline
- runner ocupado o sin capacidad
- job en cola indefinidamente
- versión antigua del runner
- fallo de conectividad hacia GitHub o recursos internos

## 7. Gestión de secrets en la empresa

### Scopes

- repository secrets: secretos definidos solo para un repositorio concreto, adecuados para pipelines aisladas.
- environment secrets: secretos vinculados a un entorno protegido y entregados solo a jobs que declaran ese environment.
- organization secrets: secretos compartidos a nivel de organización, configurables para todos o solo algunos repositorios.

### Reglas importantes

- organization secrets pueden limitarse a repositorios concretos
- environment secrets requieren usar el environment correspondiente
- los secrets no deben imprimirse ni transformarse sin masking
- no se exponen por defecto a forks no confiables

Ejemplo de uso de un secret organizacional:

```yaml
- run: echo "Desplegando"
  env:
    API_TOKEN: ${{ secrets.ORG_DEPLOY_TOKEN }}
```

---

# Bloque final de memorización rápida

## Diferencias críticas que suelen caer en examen

- `action` frente a `reusable workflow`: una action se usa a nivel de step; un reusable workflow se usa a nivel de job.
- `cache` frente a `artifact`: la caché acelera futuras ejecuciones; el artifact conserva archivos de una ejecución.
- `env` frente a `vars` frente a `secrets`: `env` se define en YAML, `vars` son variables de configuración, `secrets` son valores cifrados.
- `pull_request` frente a `pull_request_target`: el segundo corre en el contexto del repo destino y exige más cuidado.
- `starter workflow` frente a `reusable workflow`: la plantilla se copia; el reusable workflow se referencia.
- `continue-on-error` no reintenta; solo deja continuar.
- `workflow_dispatch` es manual; `repository_dispatch` es externo; `workflow_run` encadena.
- `needs` crea dependencias; sin `needs`, los jobs van en paralelo.

## Preguntas mentales que debes saber resolver

1. Qué evento dispara esto.
2. En qué runner corre.
3. Qué permisos tiene el token.
4. De dónde salen los datos: `inputs`, `env`, `vars`, `secrets`, `matrix`, `needs`.
5. Qué ocurrirá si falla un step o un job.
6. Si el workflow publica, despliega, guarda artifacts o solo valida.
7. Si hay riesgo de seguridad en forks, secrets o actions externas.

## Rutas de documentación oficial que conviene revisar antes del examen

- https://docs.github.com/es/actions/using-workflows/events-that-trigger-workflows
- https://docs.github.com/es/actions/using-workflows/workflow-syntax-for-github-actions
- https://docs.github.com/es/actions/learn-github-actions/contexts
- https://docs.github.com/es/actions/learn-github-actions/variables
- https://docs.github.com/es/actions/using-workflows/workflow-commands-for-github-actions
- https://docs.github.com/es/actions/creating-actions
- https://docs.github.com/es/actions/using-workflows/reusing-workflows
- https://docs.github.com/es/actions/monitoring-and-troubleshooting-workflows
- https://docs.github.com/es/actions/hosting-your-own-runners
- https://docs.github.com/es/actions/security-for-github-actions/security-guides/automatic-token-authentication

## Consejo final de estudio

Si una pregunta te duda entre varias respuestas válidas, casi siempre la correcta en GitHub Actions es la que mejor respeta una de estas cuatro reglas: mínimo privilegio, reutilización correcta, alcance exacto de cada parámetro y seguridad frente a código no confiable.
