# Prácticas — Domain 3: Author and Maintain Actions

1. Crea `action.yml` para una JavaScript action que reciba `name` (required) y devuelva `greeting`.  
   - Entrega: `action.yml` mínimo y una línea de ejemplo de uso en workflow.

2. Escribe una Composite action (`action.yml`) que instale Node y ejecute `npm ci` y `npm test`.  
   - Entrega: `action.yml` del composite.

3. Docker action: crea un `Dockerfile` y `action.yml` que ejecuten un script `entrypoint.sh` que reciba un input `target`.  
   - Entrega: `Dockerfile` y `action.yml` mínimos.

4. Troubleshoot JS action: te dan un repo donde `main: dist/index.js` falla porque no existe `dist/`. Describe los pasos para reproducir localmente y corregir (comandos de build y cambio en `action.yml` si aplica).

5. Versionado seguro: muestra cómo referenciar una action en un workflow usando:
   - etiqueta mayor (`@v1`)
   - commit SHA (`@<sha>`)
   - Entrega: dos líneas `uses:` de ejemplo.

6. Inputs/Outputs y GITHUB_OUTPUT: crea un step `id: meta` que escriba `image_tag=1.2.3` en `$GITHUB_OUTPUT` y luego muestre cómo leerlo desde otro job con `needs`.  
   - Entrega: pasos YAML.

7. Publicación en Marketplace: lista mínima de requisitos y checklist para publicar una action pública (5 ítems).

8. Composite vs JS vs Docker: para cada caso indica en una línea cuándo elegirías ese tipo de action.

9. Seguridad: escribe un ejemplo de `action.yml` donde declares `runs.using: node20` y `main: dist/index.js` y añade una nota sobre empaquetado de dependencias.

10. Test local: comandos para probar una action Docker y una JavaScript action localmente (ejemplos con `act` o `docker run`).
