# Examen práctico — Certificación GitHub Actions (2026)

Instrucciones generales
- Duración recomendada: 120 minutos.
- Responde en español. Señala el número de la pregunta.
- Puntuación total: 100 puntos. Marca la respuesta en cada apartado donde aplique.
- No se incluyen claves en este documento.

Estructura
- Parte A — Conceptos y teoría (40 pts)
- Parte B — Diagnóstico y lectura (25 pts)
- Parte C — Autoría y práctica (25 pts)
- Parte D — Empresa y seguridad (10 pts)

---

Parte A — Conceptos y teoría (40 pts)
1. (2 pts — MC) ¿Cuál es la diferencia principal entre `pull_request` y `pull_request_target`?
   A) Ambos ejecutan código de la rama origen con los mismos permisos.  
   B) `pull_request_target` se ejecuta en el contexto del repo destino y puede acceder a secrets del destino.  
   C) `pull_request` permite `types`, `pull_request_target` no.  
   D) Ninguna de las anteriores.

2. (2 pts — V/F) `permissions: contents: read` impide usar el token para publicar paquetes. (V/F)

3. (3 pts — MC) ¿Cuál es la forma más segura de referenciar una action externa para evitar cambios inesperados?
   A) @main  
   B) @v1  
   C) @<commit-sha>  
   D) @latest

4. (3 pts — Short) Explica en una línea qué hace `concurrency.group` con `cancel-in-progress: true`.

5. (4 pts — MC) Selecciona el enunciado correcto sobre `workflow_dispatch` inputs:
   A) `choice` no existe, se usa `string` para todo.  
   B) `type: environment` permite seleccionar un environment existente desde la UI.  
   C) `boolean` siempre se muestra como checkbox, pero no puede tener default.  
   D) `number` no es soportado.

6. (4 pts — MC) En una matriz con `fail-fast: true`, qué comportamiento se observa al fallar una combinación:
   A) Se cancelan las demás combinaciones pendientes del mismo job.  
   B) Las demás combinaciones siguen normalmente.  
   C) El workflow se marca como neutral.  
   D) El job se reintenta automáticamente.

7. (4 pts — Short) Menciona dos diferencias prácticas entre `cache` y `artifact`.

8. (4 pts — MC) ¿Qué contexto usarías para leer el payload JSON completo del evento?
   A) env  
   B) github.event  
   C) github.payload  
   D) runner.event

9. (4 pts — Short) Enumera tres eventos que pueden usarse en `on:` para encadenar workflows (pista: uno es específico para encadenar por nombre de workflow).

10. (4 pts — MC) ¿Qué permiso se requiere para emitir un token OIDC (`id-token`) desde un workflow?
    A) id-token: read  
    B) id-token: write  
    C) contents: write  
    D) packages: write

---

Parte B — Diagnóstico y lectura (25 pts)
11. (6 pts — Diagnóstico práctico) Se proporciona este workflow que pretende publicar en GHCR pero falla con "permission denied":

```yaml
name: Publish image
on: push
permissions:
  contents: read

jobs:
  publish:
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
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

- Identifica la causa y escribe la corrección mínima en el bloque `permissions:`.

12. (5 pts — YAML debug) Corrige el siguiente fragmento que no se ejecuta y explica brevemente por qué fallaba (una línea):

```yaml
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - run: echo "Start"
    - name: Install
      run: npm ci
```

13. (4 pts — Logs) Indica las dos variables de entorno principales que habilitarías temporalmente para obtener logs detallados del runner y de las actions. Explica cómo definirías una de ellas en un workflow (línea de ejemplo).

14. (5 pts — Artifacts) Describe y muestra en YAML los pasos mínimos para que un job `build` suba `dist/` como artifact y el job `deploy` lo recupere (usa `upload-artifact` y `download-artifact`).

15. (5 pts — Interpretación) ¿Qué valor tendrá `${{ github.ref }}` si el workflow se ejecuta por `workflow_dispatch` seleccionando la rama `feature/login`? (respuesta breve)

---

Parte C — Autoría y práctica (25 pts)
16. (6 pts — Práctica) Escribe un workflow YAML llamado `ci-node.yml` que:
- Se dispare en push a `main` y en pull_request.
- Use `ubuntu-latest`.
- Instale Node 20 con `actions/setup-node@v4`.
- Ejecute `npm ci` y `npm test`.
Adjunta el YAML completo.

17. (4 pts — Reusable) Escribe la sección `on:` y `inputs:` de un workflow reusable que reciba:
- `node-version` (string, requerido)
- `run-tests` (boolean, opcional, default true)

18. (4 pts — Matrix) Diseña `strategy.matrix` para ejecutar Node 18 y 20 en `ubuntu-latest` y `windows-latest`, excluyendo la combinación Windows+Node18. Incluye `fail-fast: false`.

19. (5 pts — Seguridad en PRs) Explica en 2-3 líneas por qué ejecutar `pull_request_target` sin restricciones puede exponer secrets y da dos mitigaciones prácticas.

20. (6 pts — Fix práctico) Tienes este step que retorna un output para otro job, pero `needs.other.outputs.version` viene vacío:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.set-version.outputs.version }}
    steps:
      - id: set-version
        run: echo "::set-output name=version::1.0.0"
```

- Indica el problema y proporciona la corrección mínima necesaria en el step `set-version` (usa la sintaxis moderna).

---

Parte D — Empresa y seguridad (10 pts)
21. (3 pts — Política) Propón en 2 líneas una política org-wide para controlar el uso de actions públicas (p.ej. allow-list/deny-list).

22. (3 pts — Runners) Muestra el bloque `runs-on` para usar un runner self-hosted con labels `self-hosted`, `linux`, `gpu`. Menciona un riesgo operativo en una línea.

23. (2 pts — Environments) Escribe un job `deploy` que use `environment: production` y sólo se ejecute si `needs.ci.result == 'success'`. Adjunta `on:` o `needs` mínimos necesarios.

24. (2 pts — Versionado) Indica una estrategia de versionado para workflows y reusable workflows que minimice roturas en consumidores (3 elementos, puntos cortos).

---

Instrucciones de entrega
- Guarda tus respuestas y archivos en la carpeta del repo. Para las preguntas que pidan YAML, añade los archivos en `.github/workflows/` o en la carpeta `practicas/` y referencia el nombre del archivo en tus respuestas.
- Tiempo sugerido por sección: A 30 min, B 30 min, C 45 min, D 15 min.

Buen examen.