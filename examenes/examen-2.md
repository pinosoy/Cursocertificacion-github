# Examen 2 - Simulacro de certificación GitHub Actions

Este simulacro está alineado con la guía `guia-github-actions-copilot.md` y mantiene exactamente la misma distribución por dominios del temario:

- Domain 1 - Author and Maintain Workflows: 16 preguntas
- Domain 2 - Consume Workflows: 8 preguntas
- Domain 3 - Author and Maintain Actions: 10 preguntas
- Domain 4 - Manage GitHub Actions in the Enterprise: 6 preguntas

Total: 40 preguntas

## Instrucciones

- Elige una única respuesta correcta en cada pregunta, salvo que se indique lo contrario.
- No consultes la guía durante el intento si quieres medir tu nivel real.
- Cuando termines, puedes corregirlo manualmente o pedirme un solucionario razonado aparte.

---

# Domain 1 - Author and Maintain Workflows (16 preguntas)

## 1.

Quieres que un workflow se ejecute cuando se abra o se vuelva a abrir una pull request. ¿Qué configuración es la más adecuada?

A.

```yaml
on:
  pull_request:
    types: [opened, reopened]
```

B.

```yaml
on:
  push:
    types: [opened, reopened]
```

C.

```yaml
on:
  pull_request_target:
    branches-ignore: [opened, reopened]
```

D.

```yaml
on: pull_request.opened.reopened
```

## 2.

¿Qué afirmación sobre `paths-ignore` en el evento `push` es correcta?

A. Obliga a que al menos un archivo coincida para disparar el workflow.

B. Evita la ejecución si los cambios solo afectan a rutas excluidas.

C. Solo puede usarse junto a `tags`.

D. Funciona únicamente en `workflow_dispatch`.

## 3.

¿Cuál es el uso principal de `pull_request_target` frente a `pull_request`?

A. Ejecutarse con el contexto del repositorio base, con más cuidado de seguridad.

B. Ejecutarse únicamente cuando la PR está cerrada.

C. Permitir matrices de compilación más grandes.

D. Deshabilitar automáticamente el acceso a `GITHUB_TOKEN`.

## 4.

Observa este fragmento:

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: Versión a desplegar
        required: true
        type: string
      target:
        description: Entorno destino
        required: true
        type: environment
```

¿Qué afirmación es correcta?

A. `environment` no es un tipo válido en `workflow_dispatch`.

B. `type: environment` permite seleccionar un environment existente.

C. Los inputs manuales solo aceptan `string`.

D. `required` solo se puede usar con `boolean`.

## 5.

¿Qué expresión describe mejor un workflow programado todos los días a las 03:30 UTC?

A. `cron: '30 3 * * *'`

B. `cron: '3 30 * * *'`

C. `cron: '30 3 ? * *'`

D. `cron: '@daily 03:30'`

## 6.

¿Qué evento está diseñado para que un workflow llame a otro workflow reutilizable?

A. `workflow_run`

B. `workflow_dispatch`

C. `workflow_call`

D. `repository_dispatch`

## 7.

Tienes este bloque:

```yaml
on:
  release:
    types: [published]
```

¿Cuál es la interpretación correcta?

A. Se ejecuta cuando se crea cualquier tag.

B. Se ejecuta cuando una release queda publicada.

C. Se ejecuta antes de que exista la release.

D. Se ejecuta únicamente en prereleases.

## 8.

¿Qué declaración de permisos sigue mejor el principio de mínimo privilegio para un workflow que solo lee el código y sube resultados de CodeQL?

A.

```yaml
permissions:
  contents: read
  security-events: write
```

B.

```yaml
permissions:
  contents: write
  actions: write
```

C.

```yaml
permissions:
  write-all: true
```

D. No hay que declarar permisos para CodeQL.

## 9.

¿Qué efecto tiene `env` definido a nivel de workflow?

A. Solo aplica a actions de terceros.

B. Hace disponibles variables a jobs y steps, salvo que se sobrescriban.

C. Solo funciona dentro de `defaults`.

D. Sustituye automáticamente a los `secrets`.

## 10.

¿Qué hace `needs` en un job?

A. Duplica el job para cada elemento de una matriz.

B. Declara dependencias respecto a otros jobs.

C. Establece permisos del `GITHUB_TOKEN`.

D. Indica el runner requerido.

## 11.

Observa este fragmento:

```yaml
if: ${{ startsWith(github.ref, 'refs/tags/v') }}
```

¿Qué comportamiento expresa?

A. El job o step solo corre si la referencia es un tag que empieza por `v`.

B. El workflow solo corre en ramas `version/*`.

C. El job se ejecuta cuando existe cualquier variable `v`.

D. La condición solo es válida en `workflow_dispatch`.

## 12.

¿Qué afirmación sobre `strategy.matrix.include` es correcta?

A. Solo sirve para eliminar combinaciones.

B. Permite añadir combinaciones o campos extra a ciertas combinaciones.

C. Sustituye completamente a `matrix`.

D. Solo puede usarse con `fail-fast: true`.

## 13.

¿Para qué sirve `max-parallel` dentro de una matriz?

A. Para limitar cuántas combinaciones se ejecutan a la vez.

B. Para limitar el número total de jobs del workflow.

C. Para cancelar automáticamente la matriz si falla un step.

D. Para reducir el número de variables de `matrix`.

## 14.

¿Qué ventaja aporta `container` a nivel de job?

A. Ejecuta todos los pasos del job dentro de una imagen de contenedor concreta.

B. Convierte el workflow en una Docker action.

C. Obliga a usar runners self-hosted.

D. Hace que `services` deje de estar permitido.

## 15.

¿Qué salida moderna debe usar un step para definir un output consumible por otros pasos?

A. `echo "resultado=ok" >> $GITHUB_PATH`

B. `echo "resultado=ok" >> $GITHUB_ENV`

C. `echo "resultado=ok" >> $GITHUB_OUTPUT`

D. `echo "::set-env name=resultado::ok"`

## 16.

¿Qué describe mejor `cancel-in-progress: true` dentro de `concurrency`?

A. Reintenta automáticamente la ejecución actual.

B. Cancela ejecuciones anteriores del mismo grupo que aún siguen activas.

C. Evita que el workflow pueda cancelarse manualmente.

D. Bloquea la ejecución en todos los branches salvo `main`.

---

# Domain 2 - Consume Workflows (8 preguntas)

## 17.

Si quieres saber si una ejecución viene de una PR o de un push, ¿qué contexto debes revisar primero?

A. `github.event_name`

B. `runner.temp`

C. `strategy.event`

D. `job.container`

## 18.

En una pull request, ¿qué contexto representa normalmente la rama base?

A. `github.head_ref`

B. `github.base_ref`

C. `github.sha`

D. `runner.base_ref`

## 19.

Un workflow no aparece como iniciado y GitHub muestra un error al interpretar el archivo. ¿Cuál es la causa más probable?

A. Error de indentación o sintaxis YAML.

B. Un test ha devuelto código de salida 1.

C. Un artifact no se ha descargado.

D. El runner tiene demasiada memoria.

## 20.

¿Qué significa normalmente que un job aparezca como `cancelled`?

A. Que todos sus steps terminaron correctamente.

B. Que la ejecución fue detenida antes de completarse, por ejemplo por cancelación manual o concurrencia.

C. Que el YAML tenía un error de sintaxis.

D. Que el workflow estaba deshabilitado desde el repositorio.

## 21.

¿Qué comando de GitHub CLI se asocia de forma más directa con listar ejecuciones de workflows?

A. `gh run list`

B. `gh action ls`

C. `gh workflow logs`

D. `gh runs show-all`

## 22.

¿Para qué se usa `ACTIONS_RUNNER_DEBUG`?

A. Para activar logs más detallados del runner.

B. Para cifrar secretos en tiempo de ejecución.

C. Para habilitar matrices dinámicas.

D. Para desactivar `continue-on-error`.

## 23.

¿Qué escenario encaja mejor con artifacts y no con caché?

A. Reutilizar dependencias de Maven entre ejecuciones.

B. Conservar un binario compilado para descargarlo o pasarlo a otro job.

C. Guardar paquetes de npm para la siguiente ejecución automática.

D. Acelerar la resolución de dependencias de Python.

## 24.

¿Qué afirmación es correcta sobre starter workflows y reusable workflows?

A. Ambos se consumen siempre con `uses:`.

B. El starter workflow es una plantilla inicial; el reusable workflow se invoca en ejecución.

C. El reusable workflow se copia al repositorio y deja de estar conectado al original.

D. Ninguno de los dos admite parámetros.

---

# Domain 3 - Author and Maintain Actions (10 preguntas)

## 25.

¿Qué tipo de action permite componer varios pasos sin necesidad de JavaScript ni Docker?

A. Docker action

B. Composite action

C. Reusable workflow

D. Matrix action

## 26.

¿Qué propiedad identifica la versión de runtime en una JavaScript action moderna?

A. `shell: node20`

B. `runs.using: node20`

C. `engine: node20`

D. `runtime: node20`

## 27.

En una composite action, ¿qué suele faltar cuando los pasos `run` fallan antes de ejecutarse correctamente?

A. `shell`

B. `permissions`

C. `branding`

D. `concurrency`

## 28.

¿Qué archivo describe oficialmente inputs, outputs y modo de ejecución de una action?

A. `README.md`

B. `action.yml`

C. `.github/workflows/action.yml`

D. `manifest.json`

## 29.

¿Para qué sirve `deprecationMessage` en un input de una action?

A. Para ocultar por completo el input.

B. Para advertir a quien consume la action de que ese input debería dejar de usarse.

C. Para convertir el input en opcional.

D. Para forzar que el workflow falle siempre.

## 30.

¿Cuál es la forma moderna de escribir un output desde una action basada en shell o script?

A. Escribir en `$GITHUB_OUTPUT`

B. Escribir en `$GITHUB_PATH`

C. Escribir en `$GITHUB_STATE` exclusivamente

D. Exportar una variable del sistema operativo

## 31.

¿Qué problema suele causar que una JavaScript action publicada falle al consumirse desde otro repositorio?

A. No incluir el código compilado o empaquetado que referencia `main`.

B. Definir `inputs` con `required: true`.

C. Añadir un `README.md` demasiado largo.

D. Usar tags semánticos.

## 32.

En una Docker action, ¿qué problema es especialmente típico?

A. Usar demasiados `needs`.

B. Definir mal el `ENTRYPOINT` o no copiar el script necesario a la imagen.

C. No declarar `matrix`.

D. Añadir un `branding` con color no soportado.

## 33.

¿Qué práctica es recomendable al versionar una action pública?

A. Publicar solo en la rama `main` sin tags.

B. Mantener tags de versión mayor, como `v1`, además de tags más específicos.

C. Cambiar el nombre de la action en cada release.

D. Evitar el versionado para no romper compatibilidad.

## 34.

¿Qué referencia reduce más el riesgo de que una action de terceros cambie sin control?

A. Una rama mutable como `main`

B. Un tag ligero como `latest`

C. Un commit SHA fijo

D. Un nombre de release en texto libre

---

# Domain 4 - Manage GitHub Actions in the Enterprise (6 preguntas)

## 35.

Una organización quiere compartir lógica de CI obligatoria entre muchos repositorios sin duplicar workflows completos. ¿Qué opción encaja mejor?

A. Reusable workflows centralizados

B. Copiar y pegar el YAML en cada repositorio

C. Solo usar badges de estado

D. Convertir todos los workflows en Docker actions

## 36.

¿Qué ruta se asocia al almacenamiento de plantillas organizacionales de workflow?

A. `.github/workflow-templates`

B. `.github/workflows/templates`

C. `.github/actions/templates`

D. `.github/starter-workflows`

## 37.

¿Qué efecto puede tener una política organizacional que restrinja actions externas?

A. Ninguno, porque el YAML válido siempre prevalece.

B. Puede bloquear workflows que referencien actions no permitidas.

C. Solo afecta a repositorios públicos.

D. Solo afecta a reusable workflows, no a actions.

## 38.

¿Qué utilidad principal tienen los runner groups?

A. Clasificar artifacts por entorno.

B. Controlar qué repositorios pueden acceder a determinados runners.

C. Compartir outputs entre workflows.

D. Forzar `workflow_dispatch` en organizaciones grandes.

## 39.

¿Cuál es un caso típico para elegir un runner self-hosted?

A. Necesitar conectividad con red interna o herramientas corporativas instaladas.

B. Querer evitar mantenimiento operativo.

C. Ejecutar workflows simples y genéricos sin dependencias especiales.

D. Reducir permisos del `GITHUB_TOKEN` automáticamente.

## 40.

¿Qué afirmación sobre secretos a nivel organizacional es correcta?

A. Siempre están expuestos a todos los repositorios sin excepciones.

B. Pueden limitarse a repositorios seleccionados o a una política concreta de acceso.

C. Sustituyen automáticamente a los environment secrets.

D. No se enmascaran en logs.

---

# Hoja de respuestas

Rellena aquí tu solución si quieres autocorregirte después.

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


# Respuestas correctas

1 - A
2 - B
3 - A
4 - B
5 - A
6 - C
7 - B
8 - A
9 - B
10 - B
11 - A
12 - B
13 - A
14 - A
15 - C
16 - B
17 - A
18 - B
19 - A
20 - B
21 - A
22 - A
23 - B
24 - B
25 - B
26 - B
27 - A
28 - B
29 - B
30 - A
31 - A
32 - B
33 - B
34 - C
35 - A
36 - A
37 - B
38 - B
39 - A
40 - B
