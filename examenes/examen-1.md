# Examen 1 - Simulacro de certificacion GitHub Actions

Este simulacro esta alineado con la guia `guia-github-actions-copilot.md` y respeta exactamente la distribucion por dominios del temario:

- Domain 1 - Author and Maintain Workflows: 16 preguntas
- Domain 2 - Consume Workflows: 8 preguntas
- Domain 3 - Author and Maintain Actions: 10 preguntas
- Domain 4 - Manage GitHub Actions in the Enterprise: 6 preguntas

Total: 40 preguntas

## Instrucciones

- Elige una unica respuesta correcta en cada pregunta salvo que se indique lo contrario.
- No consultes la guia durante el intento si quieres medir tu nivel real.
- Cuando termines, puedes corregirlo manualmente o pedirme una plantilla de respuestas y solucionario aparte.

---

# Domain 1 - Author and Maintain Workflows (16 preguntas)

## 1.

Quieres que un workflow se dispare por `push` y por ejecucion manual. Cual de estas definiciones de `on` es valida.

A. `on: push, workflow_dispatch`

B.

```yaml
on:
  - push
  - workflow_dispatch
```

C.

```yaml
on:
  push:
  workflow_dispatch:
```

D. B y C son correctas

## 2.

Que afirmacion sobre el evento `push` es correcta.

A. Admite `types` como `opened` y `closed`

B. Puede filtrar por `branches`, `tags` y `paths`

C. Solo funciona sobre ramas, no sobre tags

D. No puede combinarse con otros eventos en el mismo workflow

## 3.

En un workflow de CI para pull requests, que valor de `pull_request.types` representa la llegada de nuevos commits a la rama de la PR.

A. `edited`

B. `reopened`

C. `synchronize`

D. `review_requested`

## 4.

Observa este fragmento:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Entorno
        required: true
        type: choice
        options: [dev, test, prod]
      dry_run:
        description: Simular
        default: false
        type: boolean
```

Que afirmacion es correcta.

A. `environment` y `dry_run` deben declararse dentro de `jobs`

B. `choice` no admite `options`

C. `workflow_dispatch` admite inputs tipados como `choice` y `boolean`

D. `default` no se puede usar con `boolean`

## 5.

Que afirmacion describe correctamente `schedule`.

A. Usa cron con zona horaria local del repositorio

B. La frecuencia minima es de un minuto

C. Usa cron en UTC y su frecuencia minima habitual es de 5 minutos

D. Solo puede declararse una expresion cron por workflow

## 6.

Que evento es el mas apropiado para disparar un workflow desde un sistema externo enviando un payload personalizado por API.

A. `workflow_run`

B. `repository_dispatch`

C. `workflow_call`

D. `deployment_status`

## 7.

Tienes este workflow:

```yaml
on:
  workflow_run:
    workflows: ['CI']
    types: [completed]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo deploy
```

Cuando se ejecuta el job `deploy`.

A. Siempre que el workflow `CI` comience

B. Solo cuando el workflow `CI` termine y su conclusion sea `success`

C. Solo cuando el workflow `CI` falle

D. Cuando se haga push a `main`

## 8.

Que opcion describe mejor `workflow_call`.

A. Permite ejecutar manualmente un workflow desde la interfaz

B. Convierte un workflow en reutilizable y puede definir `inputs` y `secrets`

C. Es una variante de `repository_dispatch`

D. Solo sirve para workflows dentro del mismo fichero YAML

## 9.

Necesitas publicar un paquete en GitHub Packages desde un workflow. Cual es la configuracion de permisos mas apropiada segun minimo privilegio.

A.

```yaml
permissions:
  write-all: true
```

B.

```yaml
permissions:
  contents: read
  packages: write
```

C.

```yaml
permissions:
  contents: write
```

D. No hace falta declarar permisos nunca

## 10.

Que hace esta configuracion.

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./app
```

A. Obliga a que todas las actions usen bash internamente

B. Define valores por defecto para los pasos `run`

C. Cambia el directorio del repositorio para todo el runner

D. Solo aplica a los jobs con matriz

## 11.

Que ocurre por defecto con dos jobs sin `needs` entre ellos.

A. Se ejecutan siempre en orden alfabetico

B. Se ejecutan en paralelo cuando es posible

C. El segundo espera al primero automaticamente

D. Solo uno se ejecuta y el otro queda omitido

## 12.

Observa esta matriz:

```yaml
strategy:
  fail-fast: false
  max-parallel: 2
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]
    exclude:
      - os: windows-latest
        node: 18
```

Cual es la interpretacion correcta.

A. Se ejecutan 4 combinaciones siempre

B. Se ejecutan 3 combinaciones y como maximo 2 a la vez

C. `fail-fast: false` cancela el resto cuando falla una combinacion

D. `exclude` solo funciona con `include`

## 13.

Quieres lanzar tests de integracion contra Postgres desde el mismo job. Que mecanismo de GitHub Actions esta pensado especificamente para eso.

A. `environment`

B. `services`

C. `concurrency`

D. `run-name`

## 14.

Que efecto practico tiene asociar un job a este bloque.

```yaml
environment:
  name: production
  url: https://app.example.com
```

A. Convierte el job automaticamente en reusable workflow

B. Puede activar protecciones de entorno, approvals y secrets del environment

C. Obliga al job a ejecutarse en un runner self-hosted

D. Publica automaticamente una release

## 15.

Como se define de forma moderna un output de step para que luego pueda usarlo otro paso o un output de job.

A. `echo "value=1" >> $GITHUB_ENV`

B. `echo "value=1" >> $GITHUB_OUTPUT`

C. `export value=1`

D. `echo "::set-env name=value::1"`

## 16.

Que hace esta configuracion.

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

A. Reintenta automaticamente los despliegues fallidos

B. Evita ejecuciones concurrentes del mismo grupo y cancela la anterior en curso

C. Impide que el workflow pueda ejecutarse manualmente

D. Fuerza a todos los jobs a usar el mismo runner

---

# Domain 2 - Consume Workflows (8 preguntas)

## 17.

Que contexto debes inspeccionar primero para saber que evento disparo una ejecucion.

A. `runner.os`

B. `github.event_name`

C. `job.status`

D. `matrix.event`

## 18.

En una pull request, que contexto contiene la rama origen del cambio.

A. `github.base_ref`

B. `github.head_ref`

C. `github.ref_name`

D. `runner.head_ref`

## 19.

Si un step aparece como `skipped` en la interfaz, cual es la causa mas tipica.

A. El runner perdio conectividad

B. El step fue eliminado del YAML durante la ejecucion

C. La condicion `if` no se cumplio

D. El job estaba deshabilitado

## 20.

Que tipo de problema suele impedir que un workflow arranque siquiera.

A. Un `npm test` con exit code 1

B. Un error de sintaxis YAML o indentacion

C. Un `continue-on-error: true`

D. Un artifact inexistente

## 21.

Que opcion permite inspeccionar una ejecucion desde la linea de comandos oficial de GitHub.

A. `gh actions inspect`

B. `gh run view`

C. `git run show`

D. `gh workflow debug`

## 22.

Que ajuste se usa para obtener logs mas detallados a nivel de pasos.

A. `ACTIONS_DEBUG_VERBOSE`

B. `RUNNER_TRACE`

C. `ACTIONS_STEP_DEBUG`

D. `GITHUB_LOG_DEBUG`

## 23.

Que afirmacion diferencia correctamente un artifact de una cache.

A. Ambos son identicos; solo cambia el nombre de la action

B. La cache esta pensada para acelerar futuras ejecuciones y el artifact para conservar o transferir archivos de una ejecucion

C. El artifact solo puede usarse en workflows manuales

D. La cache no depende nunca de una clave

## 24.

Que afirmacion sobre workflows deshabilitados y plantillas es correcta.

A. Un workflow deshabilitado se elimina del repositorio

B. Un starter workflow se invoca igual que un reusable workflow con `uses`

C. Un workflow deshabilitado sigue existiendo y un starter workflow se copia al repositorio

D. Un reusable workflow siempre desaparece tras ejecutarse

---

# Domain 3 - Author and Maintain Actions (10 preguntas)

## 25.

Que bloque `runs` corresponde a una JavaScript action moderna.

A.

```yaml
runs:
  using: bash
  main: index.sh
```

B.

```yaml
runs:
  using: node20
  main: dist/index.js
```

C.

```yaml
runs:
  using: dockerfile
  main: dist/index.js
```

D.

```yaml
runs:
  using: composite
  main: dist/index.js
```

## 26.

En una Docker action, cual es la definicion mas tipica en `action.yml`.

A.

```yaml
runs:
  using: docker
  image: Dockerfile
```

B.

```yaml
runs:
  using: container
  file: Dockerfile
```

C.

```yaml
runs:
  using: node20
  image: Dockerfile
```

D.

```yaml
runs:
  using: docker
  main: Dockerfile
```

## 27.

Que detalle suele ser obligatorio en los pasos `run` de una composite action.

A. `permissions`

B. `shell`

C. `timeout-minutes`

D. `continue-on-error`

## 28.

Que archivo es imprescindible para definir oficialmente una action.

A. `README.md`

B. `package.json`

C. `action.yml`

D. `index.js`

## 29.

Que campo de `inputs` permite avisar de que una entrada sigue existiendo pero deberia dejar de usarse.

A. `legacyWarning`

B. `deprecationMessage`

C. `obsolete`

D. `notice`

## 30.

Como debe exponer una action un output de forma moderna desde shell.

A. `echo "name=value" >> "$GITHUB_OUTPUT"`

B. `echo "::set-output name=name::value"`

C. `export name=value`

D. `echo "name=value" >> "$GITHUB_ENV"`

## 31.

Has creado una JavaScript action en TypeScript, pero al ejecutarla falla porque no encuentra `dist/index.js`. Cual es la causa mas probable.

A. Has usado `branding`

B. No se ha generado o incluido el artefacto compilado que apunta `main`

C. La action deberia ser obligatoriamente Docker

D. `inputs` no admite strings

## 32.

Que problema es caracteristico de una Docker action mal preparada.

A. Faltan labels en `runs-on`

B. El `ENTRYPOINT` o el fichero copiado al contenedor no es correcto

C. No existe `workflow_dispatch`

D. La action no puede recibir `args`

## 33.

Que conjunto describe mejor los requisitos habituales para publicar una action en Marketplace.

A. Repositorio publico, `action.yml`, `README.md` y versionado con tags

B. Repositorio privado y `Dockerfile` obligatorio

C. Solo hace falta que exista `package.json`

D. Solo se publican reusable workflows, no actions

## 34.

Que referencia es la mas segura al consumir una action de terceros.

A. `@main`

B. `@latest`

C. `@v1`

D. Un commit SHA fijo

---

# Domain 4 - Manage GitHub Actions in the Enterprise (6 preguntas)

## 35.

Una organizacion quiere centralizar su pipeline Node para que decenas de repositorios la reutilicen sin copiar el YAML completo. Que mecanismo encaja mejor.

A. Starter workflows

B. Reusable workflows con `workflow_call`

C. Badges de workflow

D. Issue templates

## 36.

En que ruta se suelen publicar las plantillas organizacionales de workflows.

A. `.github/actions`

B. `.github/workflows-templates`

C. `.github/workflow-templates`

D. `.github/templates/workflows`

## 37.

Que puede ocurrir aunque un workflow este correctamente escrito, si la organizacion tiene politicas restrictivas sobre actions.

A. Nada, las politicas no afectan a workflows validos

B. El workflow puede quedar bloqueado por usar actions no permitidas

C. GitHub cambia automaticamente las actions externas por acciones oficiales

D. El runner ignora la politica si el repositorio es privado

## 38.

Para que sirven principalmente los runner groups.

A. Para comprimir artifacts

B. Para controlar que repositorios pueden usar determinados runners self-hosted

C. Para compartir variables `env` entre workflows

D. Para habilitar `workflow_dispatch`

## 39.

En que caso es mas razonable usar un runner self-hosted.

A. Cuando quieres cero mantenimiento y entorno estandarizado

B. Cuando necesitas acceso a red interna o herramientas corporativas especificas

C. Cuando quieres evitar definir `permissions`

D. Cuando necesitas usar `actions/checkout`

## 40.

Que afirmacion sobre scopes de secrets es correcta.

A. Un environment secret esta disponible aunque el job no declare `environment`

B. Un organization secret puede limitarse a repositorios concretos

C. Los repository secrets siempre tienen mas alcance que los organization secrets

D. Los secrets se comportan igual que `vars` y no van enmascarados

---

# Hoja de respuestas

Rellena aqui tu solucion si quieres autocorregirte despues.

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

---

# Respuestas correctas

1 - D
2 - B
3 - C
4 - C
5 - C
6 - B
7 - B
8 - B
9 - B
10 - B
11 - B
12 - B
13 - B
14 - B
15 - B
16 - B
17 - B
18 - B
19 - C
20 - B
21 - B
22 - C
23 - B
24 - C
25 - B
26 - A
27 - B
28 - C
29 - B
30 - A
31 - B
32 - B
33 - A
34 - D
35 - B
36 - C
37 - B
38 - B
39 - B
40 - B

---