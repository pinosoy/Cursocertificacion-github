# Prácticas — Domain 2: Consume Workflows

1. Lectura: Se proporciona un workflow (archivo). Identifica qué lo dispara, qué jobs crea y si publica artifacts.  
   - Entrega: lista de 4 puntos (evento, jobs, publicación, permisos requeridos).

2. Debug YAML: Recibe este fragmento que falla al parsear; corrígelo y explica el error en una línea.
   - Fragmento:
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
   - Entrega: YAML corregido.

3. Logs y depuración: Describe los pasos para habilitar logs detallados de runner y steps (variables de entorno a activar y cómo activarlas temporalmente en un repo).  
   - Entrega: comandos o fragmento env.

4. Artifacts entre jobs: Implementa dos jobs `build` y `deploy` donde `build` crea `dist/` y lo sube como artifact y `deploy` lo descarga.  
   - Entrega: YAML con ambos jobs y los pasos `upload`/`download`.

5. Diagnóstico de permisos: Workflow falla con `Resource not accessible by integration` al intentar publicar en Packages. Indica la causa probable y la corrección mínima en permisos YAML.

6. Interpretación de contexto: Qué valores devolvemos para:
   - `${{ github.event_name }}` cuando se lanza por `repository_dispatch`.
   - `${{ github.ref }}` en `workflow_dispatch` apuntando a `feature/x`.
   - Entrega: dos líneas con los valores esperados.

7. Uso de `needs.outputs`: Construye un ejemplo donde `build` genera `version` y `deploy` usa `needs.build.outputs.version`.  
   - Entrega: fragmento YAML de ambos jobs.

8. Simula un escenario de forks donde un step imprime un secret y explica por qué ese secret puede no estar disponible.  
   - Entrega: comando `run` y explicación (1 línea).

9. Diagnostica paso: un job no encuentra dependencias por culpa de `working-directory` global. Describe cómo comprobar y arreglarlo.

10. Descargar logs con CLI: Comandos `gh` para listar runs, ver un run específico y descargar artifacts de ese run.
# Prácticas — Domain 4: Manage GitHub Actions in the Enterprise

1. Política de actions: diseña un documento breve (3-4 líneas) que explique una política "allow list" de actions y cómo aplicarla en la organización.  
   - Entrega: texto breve.

2. Reusable workflows: crea un workflow reusable `node-ci.yml` que reciba `node-version` y exponga un output `tests_passed`. Luego muestra cómo invocarlo desde otro repo con `uses:`.  
   - Entrega: `node-ci.yml` (sección `on:` y job `uses`) y ejemplo de consumidor.

3. Runners self-hosted: escribe un ejemplo de `runs-on` que seleccione un runner con labels `self-hosted`, `linux`, `gpu`. Explica un riesgo operativo de 1 línea.

4. Organization secrets: muestra cómo declarar en un workflow que hereda secretos de la organización para `npm publish` y menciona la precaución clave con forks.

5. Environments y approvals: crea un job `deploy` que use `environment: production` y que solo se ejecute si `needs.ci` fue successful usando `if:`.  
   - Entrega: bloque `jobs.deploy` con `needs`, `if` y `environment`.

6. Strategy de migración de workflows centralizados: plan en 4 pasos para actualizar un reusable workflow sin romper consumidores (versionado semántico, branch protection, pruebas, comunicación).

7. Control de costos: diseño breve (3 puntos) para limitar concurrencia y paralelismo en workflows globales de la organización.

8. Auditoría y logs: lista de comandos o pasos para obtener lista de runs que fallan más frecuentemente y enlazar con repositorios consumidores (uso de `gh` o queries de API).

9. Gestión de approvals automáticos: cómo configurar un environment con reviewers requeridos (breve YAML o pasos UI).

10. Recovery: pasos rápidos para aislar un workflow malicioso detectado en un repositorio de plataforma (quitar permisos, deshabilitar workflow, revert tag).
