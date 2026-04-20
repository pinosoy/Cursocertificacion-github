# Prácticas — Domain 1: Author and Maintain Workflows

1. Escribe un workflow llamado `ci.yml` que:
   - Se dispare en `push` a `main` y en `pull_request`.
   - Use `permissions` con el mínimo privilegio para ejecutar tests que solo leen el repo.
   - Instale Node 20 y ejecute `npm ci` y `npm test`.
   - Entrega: YAML completo.

2. Diseña un workflow con `concurrency` que evite solapamiento por rama y cancele ejecuciones previas.  
   - Muestra la expresión de `concurrency.group` usando `github.ref_name`.
   - Entrega: fragmento YAML del bloque `concurrency` y explicación (1-2 líneas).

3. Crea un workflow que publique una imagen en GHCR desde `main`.  
   - Incluye `permissions` necesarios y pasos para login con `GITHUB_TOKEN`.  
   - Entrega: YAML mínimo que incluya `docker/login-action` y `docker/build-push-action`.

4. Implementa una matriz (`strategy.matrix`) para test en Node 18 y 20 sobre `ubuntu-latest` y `windows-latest`, excluyendo Windows+Node18, con `fail-fast: false`.  
   - Entrega: bloque `strategy` y ejemplo de uso de `matrix.node` en `uses` o `setup-node`.

5. Añade a un workflow `workflow_dispatch` inputs: `environment` (choice: dev,staging,prod) obligatorio y `dry_run` (boolean, default true).  
   - Entrega: sección `on:` completa.

6. Crea un job que use `defaults.run.working-directory` para ejecutar tests dentro de `./frontend`.  
   - Entrega: job YAML completo con `defaults` y pasos.

7. Escribe un ejemplo donde un job usa `if:` para correr solo si la rama es `refs/heads/main` y la variable `CI` es `true`.  
   - Entrega: línea `if:`.

8. Prepara un workflow con `workflow_call` que exponga `node-version` (string required) y `run-tests` (boolean, default true).  
   - Entrega: sección `on:` y `inputs`.

9. Diseña un `job` que corra dentro de un `container` usando `node:20` y exponga una variable de entorno al contenedor.  
   - Entrega: bloque `container` y paso que imprime la variable.

10. Diagnostica: te dan un workflow que no publica paquetes por falta de permiso. Indica qué permiso añadir y por qué (respuesta corta).
