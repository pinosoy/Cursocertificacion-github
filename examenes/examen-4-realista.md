# Examen 4 - Simulacro realista de certificacion GitHub Actions

Este simulacro esta construido a partir de [manual-github-actions.md](/home/adelpino/GITHUB/pinosoy/Cursocertificacion-github/manual-github-actions.md) y de los temas cubiertos por la documentacion oficial de GitHub Actions.

Objetivo: acercarte a preguntas de estilo examen, con foco en interpretacion de YAML, permisos, seguridad, eventos, reusable workflows, runners, logs, cache, artifacts y authoring de actions.

## Formato del simulacro

- 40 preguntas
- 75 minutos recomendados
- Una sola respuesta correcta salvo que se indique expresamente otra cosa
- Nivel: intermedio a avanzado

## Distribucion por dominios

- Domain 1 - Author and Maintain Workflows: 16 preguntas
- Domain 2 - Consume Workflows: 8 preguntas
- Domain 3 - Author and Maintain Actions: 10 preguntas
- Domain 4 - Manage GitHub Actions in the Enterprise: 6 preguntas

## Instrucciones

- Lee con cuidado el contexto tecnico de cada pregunta.
- Si aparece YAML, piensa siempre en alcance: workflow, job, step, action.
- Si dudas entre varias respuestas, prioriza minimo privilegio, sintaxis valida y seguridad frente a codigo no confiable.
- No se incluye solucionario en este archivo para mantener el efecto de simulacro.

---

# Domain 1 - Author and Maintain Workflows

## 1.

Quieres ejecutar un workflow cuando haya push a `main`, cuando se abra o actualice una pull request contra `main`, y tambien manualmente. Cual de estas configuraciones es valida y cumple ese objetivo.

A.

```yaml
on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize]
    branches: [main]
  workflow_dispatch:
```

B.

```yaml
on:
  push: [main]
  pull_request: [opened, synchronize]
  workflow_dispatch: true
```

C.

```yaml
on: push, pull_request, workflow_dispatch
```

D.

```yaml
on:
  push:
    branches: main
  pull_request:
    branches: opened
```

## 2.

Que afirmacion sobre `push` es correcta.

A. Admite `types` como `opened`, `closed` y `reopened`

B. Puede filtrarse por `branches`, `tags`, `paths` y sus variantes `ignore`

C. Solo se ejecuta sobre ramas, nunca sobre tags

D. No puede convivir con `workflow_dispatch` en el mismo workflow

## 3.

Un equipo quiere que su CI de pull requests se lance cada vez que entren commits nuevos en la rama origen de la PR. Que tipo de actividad de `pull_request` representa ese caso.

A. `edited`

B. `review_requested`

C. `synchronize`

D. `converted_to_draft`

## 4.

Necesitas una ejecucion manual con dos inputs: `target` con opciones `dev`, `test`, `prod`; y `dry_run` verdadero o falso. Que definicion es correcta.

A.

```yaml
on:
  workflow_dispatch:
    inputs:
      target:
        type: choice
        required: true
        options: [dev, test, prod]
      dry_run:
        type: boolean
        required: false
        default: false
```

B.

```yaml
on:
  workflow_dispatch:
    vars:
      target: [dev, test, prod]
      dry_run: false
```

C.

```yaml
on:
  workflow_dispatch:
    inputs:
      target:
        type: environment
        options: [dev, test, prod]
      dry_run:
        type: string
        default: boolean
```

D.

```yaml
on:
  workflow_dispatch:
    parameters:
      target:
        choices: [dev, test, prod]
```

## 5.

Que expresion cron ejecuta un workflow todos los dias a las 02:15 UTC.

A. `15 2 * * *`

B. `2 15 * * *`

C. `0 15 2 * *`

D. `@daily 02:15`

## 6.

Quieres encadenar un workflow `deploy.yml` para que reaccione a la finalizacion del workflow `CI`. Que evento esta diseñado para eso.

A. `repository_dispatch`

B. `workflow_run`

C. `workflow_call`

D. `deployment_status`

## 7.

Observa este fragmento:

```yaml
on:
  workflow_run:
    workflows: ['CI']
    types: [completed]
    branches: [main]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy"
```

Que describe correctamente este workflow.

A. Se ejecuta siempre que se abra una PR contra `main`

B. Se ejecuta cuando el workflow `CI` termine sobre `main`, pero el job solo corre si la conclusion fue `success`

C. El job `deploy` corre incluso si `CI` fue cancelado

D. `branches` filtra la rama del propio workflow `deploy`, no la del workflow origen

## 8.

Que afirmacion sobre `workflow_call` es correcta.

A. Solo sirve para workflows dentro del mismo repositorio y misma rama

B. Convierte un workflow en reutilizable y permite declarar `inputs`, `outputs` y `secrets`

C. Es un alias de `workflow_dispatch`

D. Solo puede usarse desde GitHub Enterprise Server

## 9.

Quieres publicar un paquete en GitHub Packages usando `GITHUB_TOKEN`. Que bloque `permissions` sigue mejor el principio de minimo privilegio.

A.

```yaml
permissions:
  contents: read
  packages: write
```

B.

```yaml
permissions:
  contents: write
  packages: write
  actions: write
```

C.

```yaml
permissions:
  write-all: true
```

D.

```yaml
permissions:
  packages: read
```

## 10.

Que efecto tiene esta configuracion.

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./app
```

A. Obliga a todas las actions `uses` a ejecutarse dentro de `./app`

B. Define valores por defecto para pasos `run`, salvo que un step los sobrescriba

C. Cambia la rama por defecto del repositorio

D. Solo aplica si el workflow usa matrices

## 11.

Tienes dos jobs sin `needs`. Que ocurrira normalmente.

A. Se ejecutaran en paralelo si hay capacidad disponible

B. Se ejecutaran en orden alfabetico de identificador

C. El segundo esperara siempre al primero

D. Uno de los dos quedara omitido si comparten runner

## 12.

Que describe mejor `concurrency`.

A. Permite repetir automaticamente un step fallido

B. Evita ejecuciones solapadas dentro de un mismo grupo y puede cancelar la anterior en curso

C. Limita el numero de pasos dentro de un job

D. Solo funciona con `workflow_dispatch`

## 13.

Quieres ejecutar pruebas en `ubuntu-latest` y `windows-latest` con Node 18 y 20, pero excluir Windows con Node 18. Que herramienta de workflow esta pensada para ese patron.

A. `services`

B. `strategy.matrix` con `exclude`

C. `defaults.run`

D. `permissions`

## 14.

Que afirmacion sobre `services` es correcta.

A. Solo esta disponible en self-hosted runners

B. Permite levantar contenedores auxiliares como Postgres o Redis para un job

C. Sustituye a `container` y no puede combinarse con el

D. Solo sirve para publicar imagenes a GHCR

## 15.

Necesitas que un job se autentique contra un proveedor cloud sin almacenar secretos estaticos. Que permiso es clave para un flujo OIDC.

A. `packages: write`

B. `pull-requests: write`

C. `id-token: write`

D. `checks: write`

## 16.

Que respuesta describe mejor la diferencia entre `pull_request` y `pull_request_target`.

A. `pull_request_target` no puede usar secrets bajo ninguna circunstancia

B. `pull_request` siempre tiene mas permisos que `pull_request_target`

C. `pull_request_target` se ejecuta en el contexto del repositorio base y exige mas cuidado de seguridad

D. Son equivalentes salvo por el nombre

---

# Domain 2 - Consume Workflows

## 17.

Observa este job:

```yaml
jobs:
  publish:
    if: ${{ github.ref == 'refs/heads/main' && success() }}
    runs-on: ubuntu-latest
    steps:
      - run: echo publish
```

Cuando se ejecuta `publish`.

A. Siempre que el workflow se dispare desde cualquier rama

B. Solo si la referencia actual es `main` y la condicion previa del flujo es satisfactoria

C. Solo en pull requests

D. Solo si falla un job anterior

## 18.

En un workflow disparado por `pull_request`, que contexto te da acceso a la rama base de la PR.

A. `github.head_ref`

B. `github.base_ref`

C. `github.ref_name`

D. `runner.os`

## 19.

Quieres ver el nombre del evento que disparo una ejecucion. Que expresion es la correcta.

A. `${{ event.name }}`

B. `${{ github.trigger }}`

C. `${{ github.event_name }}`

D. `${{ runner.event }}`

## 20.

Observa este YAML:

```yaml
env:
  APP_ENV: dev

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      APP_ENV: test
    steps:
      - run: echo "$APP_ENV"
      - run: echo "$APP_ENV"
        env:
          APP_ENV: integration
```

Que salida tendran los dos pasos.

A. `dev` y `dev`

B. `test` y `test`

C. `test` y `integration`

D. `integration` y `integration`

## 21.

Necesitas pasar la carpeta `dist/` de un job `build` a un job `deploy` que corre en otro runner. Cual es la aproximacion correcta.

A. Definir `env: DIST=dist/` y reutilizarla en el segundo job

B. Usar `actions/cache` porque cache y artifact son equivalentes

C. Subir `dist/` con `actions/upload-artifact` y descargarlo luego con `actions/download-artifact`

D. Confiar en que ambos jobs comparten filesystem por defecto

## 22.

Que diferencia describe mejor `cache` frente a `artifact`.

A. Cache conserva archivos de una ejecucion para descarga manual; artifact acelera dependencias

B. Cache optimiza rendimiento entre ejecuciones; artifact conserva ficheros producidos por una ejecucion

C. Cache solo funciona en runners Windows; artifact solo en Linux

D. No existe diferencia funcional

## 23.

Quieres depurar un problema de ejecucion con mas detalle en steps y runner. Que variables de depuracion suelen activarse temporalmente.

A. `DEBUG=true` y `TRACE=true`

B. `ACTIONS_STEP_DEBUG` y `ACTIONS_RUNNER_DEBUG`

C. `GITHUB_DEBUG` y `RUNNER_TRACE`

D. `LOG_LEVEL=debug` y `RUNNER_LOG=1`

## 24.

Un workflow deja de ejecutarse automaticamente, pero el fichero YAML sigue existiendo y puede volver a activarse desde la interfaz. Que ha ocurrido.

A. El workflow fue eliminado

B. El workflow fue deshabilitado

C. El repositorio perdio la rama por defecto

D. El workflow fue convertido en reusable

---

# Domain 3 - Author and Maintain Actions

## 25.

Que tipo de action elegirias si necesitas escribir logica reutilizable en Node.js y exponer `inputs` y `outputs` de forma directa.

A. Composite action

B. JavaScript action

C. Docker action obligatoriamente

D. Reusable workflow

## 26.

Que bloque `runs` corresponde a una JavaScript action moderna.

A.

```yaml
runs:
  using: node20
  main: dist/index.js
```

B.

```yaml
runs:
  using: javascript
  entrypoint: index.js
```

C.

```yaml
runs:
  using: node
  script: index.js
```

D.

```yaml
runs:
  using: composite
  main: dist/index.js
```

## 27.

Que tipo de action suele ser mas adecuada cuando necesitas aislar un entorno complejo con dependencias del sistema operativo y binarios nativos.

A. Docker action

B. Composite action

C. Reusable workflow

D. Workflow command

## 28.

Que requisito suele olvidarse en muchas composite actions y provoca fallo en steps con `run`.

A. Declarar `timeout-minutes` en cada paso

B. Declarar `shell` en los pasos `run`

C. Definir `services`

D. Definir `permissions`

## 29.

Observa este fragmento de `action.yml`:

```yaml
inputs:
  token:
    description: Token de acceso
    required: true
outputs:
  image:
    description: Imagen generada
```

Que afirmacion es correcta.

A. `outputs` solo se permite en Docker actions

B. `required` obliga al consumidor a proporcionar `token` si quiere usar correctamente la action

C. `description` es obligatoria tambien dentro de `outputs` por validador YAML

D. `inputs` se define dentro del workflow consumidor, no en la action

## 30.

En una JavaScript action, que llamada del toolkit se usa normalmente para marcar el fallo de la action.

A. `core.notice('error')`

B. `core.stop('failed')`

C. `core.setFailed('mensaje')`

D. `core.errorExit('mensaje')`

## 31.

Que forma moderna se usa para establecer un output desde shell dentro de una action o workflow.

A. `echo "::set-output name=result::ok"`

B. `export GITHUB_OUTPUT=result=ok`

C. `echo "result=ok" >> "$GITHUB_OUTPUT"`

D. `set-output result ok`

## 32.

Quieres publicar una action para consumo estable por terceros. Que estrategia de versionado es la mas adecuada.

A. Pedir a los usuarios que consuman siempre `@main`

B. Publicar tags semanticos y mover un tag mayor estable como `v1`

C. No usar tags porque GitHub resuelve siempre a la ultima release

D. Usar solo nombres de ramas sin releases

## 33.

Que afirmacion sobre reusable workflow frente a action es correcta.

A. Ambos se invocan a nivel de `step` con `uses`

B. Una action se usa tipicamente a nivel de `step`; un reusable workflow se usa a nivel de `job`

C. Un reusable workflow no puede recibir `secrets`

D. Una action no puede recibir `inputs`

## 34.

Tienes una action en TypeScript cuyo `action.yml` apunta a `dist/index.js`, pero en el repositorio solo existe `src/index.ts`. Cual es el problema mas probable.

A. `dist/` debe declararse dentro de `permissions`

B. Falta compilar y publicar el artefacto generado que realmente ejecuta la action

C. Las JavaScript actions no admiten TypeScript como fuente

D. `main` solo acepta extensiones `.mjs`

---

# Domain 4 - Manage GitHub Actions in the Enterprise

## 35.

Una organizacion quiere estandarizar pipelines para decenas de repositorios y mantener la logica CI en un solo sitio. Que estrategia encaja mejor.

A. Copiar el mismo YAML manualmente en todos los repositorios y editarlo localmente

B. Mantener reusable workflows versionados en un repositorio compartido

C. Reemplazar todos los workflows por scripts Bash sin Actions

D. Forzar que todos usen solo `workflow_dispatch`

## 36.

Que diferencia describe mejor `starter workflows` frente a `reusable workflows`.

A. Un starter workflow se invoca en runtime; un reusable workflow se copia al repo

B. Un starter workflow es una plantilla que se copia; un reusable workflow se referencia y mantiene centralizado

C. Ambos son equivalentes

D. Los starter workflows solo existen en GitHub Enterprise Server

## 37.

La organizacion quiere impedir el uso de actions de terceros no aprobadas. Que capacidad de gobierno aplica a este caso.

A. `strategy.fail-fast`

B. Politicas organizacionales de allow list o bloqueo de actions

C. `concurrency.group`

D. `workflow_run`

## 38.

Que afirmacion sobre runner groups y labels es correcta.

A. Las labels reemplazan por completo a los runner groups

B. Los runner groups controlan que repos pueden usar determinados runners; las labels ayudan a enrutar jobs a capacidades concretas

C. Solo los GitHub-hosted runners pueden tener labels

D. Un job `runs-on: [self-hosted, linux]` se ejecuta en cualquier runner aunque no tenga ambas labels

## 39.

Quieres compartir un secret con varios repositorios de una organizacion, pero no con todos. Que opcion es la mas adecuada.

A. Repository secret en cada repo manualmente, porque no existe alternativa organizacional

B. Organization secret limitado a repositorios seleccionados

C. Environment secret global para toda la organizacion

D. Variable `vars` porque funciona igual que `secrets`

## 40.

Un equipo usa self-hosted runners para acceder a recursos internos. Cual es uno de los riesgos operativos mas relevantes frente a GitHub-hosted runners.

A. No pueden ejecutar jobs en paralelo nunca

B. Requieren mantenimiento, endurecimiento y limpieza del host entre ejecuciones

C. No admiten secrets

D. No pueden usar workflows con `on: push`

---

## Plantilla de respuestas

Puedes responder asi para autocorregirte despues o para pedirme el solucionario:

```text
1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.
15.
16.
17.
18.
19.
20.
21.
22.
23.
24.
25.
26.
27.
28.
29.
30.
31.
32.
33.
34.
35.
36.
37.
38.
39.
40.
```

## Recomendacion de uso

Haz primero el simulacro sin mirar el manual. Si fallas varias preguntas del mismo dominio, vuelve a [manual-github-actions.md](/home/adelpino/GITHUB/pinosoy/Cursocertificacion-github/manual-github-actions.md) y repasa especificamente eventos, permisos, reusable workflows, seguridad de `pull_request_target`, cache frente a artifacts y authoring de actions.