# Examen práctico: Certificación GitHub Actions (2026) — Examen 3

Instrucciones:
- Tiempo recomendado: 90 minutos.
- Responde en español.
- No se proporciona la clave en este documento.
- Señala claramente el número de la pregunta en tus respuestas.

## Estructura
- Parte A — Teoría y conceptos (10 preguntas, 40%)
- Parte B — Lectura y diagnóstico (6 preguntas, 30%)
- Parte C — Diseño y autoría (6 preguntas, 30%)

---

## Parte A — Teoría y conceptos (10 puntos, 40%)

1. (MC) ¿Cuál de estas afirmaciones es correcta respecto a `pull_request` vs `pull_request_target`?
   A) Ambos ejecutan el código de la rama origen con los secrets del repo destino.  
   B) `pull_request_target` se ejecuta en el contexto del repo destino y puede exponer secrets si no se controla.  
   C) `pull_request` permite `types`, `pull_request_target` no.  
   D) Ninguna es correcta.

2. (V/F) `permissions: contents: read` en el workflow evita que el job pueda publicar paquetes en el repositorio. (V o F)

3. (MC) ¿Qué contexto debes usar para leer el payload completo del evento dentro de un step?
   A) env  
   B) github.event  
   C) secrets  
   D) runner

4. (MC) En `workflow_dispatch` ¿qué tipo de input conviene usar para elegir entre entornos predefinidos desde la UI?
   A) string  
   B) choice  
   C) boolean  
   D) number

5. (MC) ¿Cuál es la diferencia práctica entre `cache` y `artifact`?
   A) Ninguna, son sinónimos.  
   B) Cache almacena dependencias para acelerar runs; artifact guarda archivos de una ejecución para descarga posterior.  
   C) Artifact solo funciona en self-hosted runners.  
   D) Cache solo se usa con Docker.

6. (MC) ¿Qué permiso mínimo necesitas declarar si tu workflow solo ejecuta tests que leen el código fuente?
   A) contents: write  
   B) contents: read  
   C) packages: write  
   D) id-token: write

7. (Short) Define brevemente qué hace `concurrency.group` con `cancel-in-progress: true`.

8. (MC) ¿Cuál es la forma más segura de referenciar una action publicada para evitar cambios inesperados?
   A) @main  
   B) @v1  
   C) @<commit-sha>  
   D) @latest

9. (MC) En una matriz (`strategy.matrix`) con `fail-fast: true`, ¿qué ocurre cuando una combinación falla?
   A) Solo la fila falla, las demás continúan.  
   B) Se cancelan las demás combinaciones pendientes del mismo job.  
   C) Se reintenta automáticamente la combinación fallida.  
   D) El workflow entero se pausa hasta intervención manual.

10. (Short) Menciona dos riesgos de seguridad al ejecutar workflows desde forks y cómo mitigarlos.

---

## Parte B — Lectura y diagnóstico (6 puntos, 30%)

11. (Diagnóstico) Observa el siguiente fragmento y explica por qué el job no publica la imagen en GHCR:

```yaml
name: Publish
on:
  push:
    branches: [main]

permissions:
  contents: read

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

12. (YAML debug) Identifica el error en este workflow que impide que el job corra:

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

13. (Interpretación) ¿Qué valor tendrá `${{ github.ref }}` si el workflow se dispara por `workflow_dispatch` manual apuntando a la rama `feature/x`?

14. (Logs / depuración) Indica el par de variables de entorno que activarías temporalmente para obtener logs detallados del runner y de los steps.

15. (Artifacts) Explica brevemente cómo pasar un directorio `dist/` generado en el job `build` para que lo use el job `deploy` en otro runner del mismo workflow.

16. (Security) Tienes un reusable workflow público usado por muchos repositorios. ¿Qué práctica aplicarías para evitar que una actualización rompa consumidores?

---

## Parte C — Diseño y autoría (6 puntos, 30%)

17. (Práctica) Escribe un workflow mínimo (yaml) llamado `ci-node.yml` que:
- Se dispare en push a `main` y `pull_request`.
- Use `runs-on: ubuntu-latest`.
- Instale Node 20 con `actions/setup-node@v4`.
- Corra `npm ci` y `npm test`.
(Respuesta: pega el YAML.)

18. (Reutilizable) Escribe las secciones `on:` y `inputs` de un workflow reusable que reciba `node-version` (string, requerido) y `run-tests` (boolean, opcional, default true).

19. (Matrix) Diseña el `strategy.matrix` para ejecutar pruebas en Node 18 y 20 sobre `ubuntu-latest` y `windows-latest`, excluyendo la combinación Windows+Node18.

20. (Permissions) Proporciona el bloque `permissions:` mínimo para un workflow que:
- Solo necesita leer el repositorio.
- Necesita emitir un token OIDC para despliegues cloud.

21. (Action authoring) Indica qué tipo de action (JS, Docker, Composite) escogerías para: a) una utilidad que parsea y modifica archivos del repo en Node; b) un paso que requiere binarios Linux nativos muy específicos y pesado ambiente de build.

22. (Scenario — chaining) Quieres que un workflow `deploy` se ejecute solo cuando el workflow `CI` termine con éxito sobre `main`. Escribe el `on:` necesario para `deploy` y la condición `if:` a nivel de job que garantiza ejecución solo cuando la conclusión fue success.

---

Fin del examen.