## Objetivo
Explorar cómo compartir salidas de construcción entre trabajos y cómo combinar el uso de caché y artefactos en el mismo flujo de trabajo.
Para lograr esto, crearemos un flujo de trabajo que construirá una aplicación React y almacenará los archivos de construcción como artefactos. Además, generaremos un informe de cobertura de pruebas unitarias y lo almacenaremos como un artefacto adicional.

## Tareas

### Trabajando con artefactos y caché:
0.  Desencadenar el flujo de trabajo desde la IU y observar los tiempos de ejecución de cada paso. ¿Cuánto tiempo se tarda en instalar las dependencias? ¿Cuánto tiempo tardaría si el flujo de trabajo se ejecutara 1000 veces?

1. Modificar el archivo llamado artifacts.yaml en la carpeta .github/workflows en la raíz de su repositorio.  Los datos del flujo de trabajo son:
  - Nombre:  Working with Artifacts
  - desencadenantes:
    - workflow_dispatch: 
  - Trabajos:
    - **test-build**:
      - se ejecuta en ubuntu-latest.    
      - tiene cuatro steps:
        - Checkout, que descarga el código del repositorio en el directorio de trabajo predeterminado.
        - Setup Node,que configura ode con la versión 20.x.     
        - Install dependencies, que ejecutar el comando npm ci --, y debería ejecutarse solo si no hubo un cache hit en el step Download cached dependencies.
        - Build code, que ejecuta el comando npm run build.
      - Realizar las moficaciones para añadir una caché
            -  Añadir un step, llamado "Download cached dependencies", que debería:
                - Tener un id  'cache'.
                - Usar la third-party action actions/cache@v4.
                - Pasar los siguientes inputs al action:
                  - key: debería ser deps-node-modules-${{ hashFiles('my-app/package-lock.json') }}
                  - path: la ruta donde restaurar la caché, que debería ser my-app/node_modules.
            -  Añadir otro step, llamado Unit tests, que debería ejecutar el comando npm run test.
            -  Modificar el step "Install dependencies", para que se ejecute solo si no hubo un cache hit en el step Download cached dependencies
 
2. Confirmar los cambios y hacer push del código. Desencadenar el flujo de trabajo desde la IU y observar los tiempos de ejecución de cada paso. ¿Cuánto tiempo se tarda en instalar las dependencias? ¿Cuánto tiempo tardaría si el flujo de trabajo se ejecutara 1000 veces? ¿Cuál es la diferencia en comparación con la versión sin caching?
3. Añadir 1 step más al job "test-build", llamado Upload build files,que debería:
          - Usar la third-party action actions/upload-artifact@v4.
          - Pasar dos inputs al action: 
            - name, con un valor de 'app', que se usa para la referenciar el artefacto en una descarga posterior,
            - path, con valor 'my-app/build', el cual se usa para determinar qué carpetas y/o archivos deben cargarse como un artefacto.
   
4. Modificar el job llamado deploy del flujo de trabajo, añadiendo un step llamado Download build files, que debería:
        - Usar la third-party action actions/download-artifact@v4.
        - Pasar dos inputs al action: 
          - name, con un valor de 'app', que debería ser el mismo que el valor del input name del action upload-artifact
          - path, con el valor  'build', que se usa para determinar en qué directorio se deben colocar los archivos del artefacto.

5. Confirmar los cambios y hacer push del código. Desencadenar el flujo de trabajo desde la IU y observar los tiempos de ejecución de cada paso. ¿Cómo se ven los artefactos cargados? 
6. Extender el flujo de trabajo para cargar informes de cobertura de pruebas unitarias como artefactos:
    - Reemplace el nombre del input en los actions upload-artifact y download-artifact existentes por una variable 'env' a nivel de flujo de trabajo llamada build-artifact-key. El valor de esta variable debería ser my-app-${{ github.sha }}.
    - Agregue una segunda variable 'env' a nivel de flujo de trabajo llamada test-coverage-key y con valor test-coverage-${{ github.sha }}.
    - Cambie el script del step "Unit tests" para generar un informe de cobertura. Si no está seguro de cómo hacerlo, consulte la sección de Tips a continuación.
    - Agregue un nuevo step después del step Unit tests. El step debería:
      - Llamarse Upload test results.
      - Cargar el contenido de la carpeta my-app/coverage, que se genera al ejecutar el comando test con la opción de cobertura activada. Use el valor de la variable test-coverage-key como el valor del input name.
5. Confirmar los cambios y hacer push del código. Desencadenar el flujo de trabajo desde la IU y observar los tiempos de ejecución de cada paso. ¿Cómo se ven los artefactos cargados? ¿Cómo se ven los informes de cobertura de pruebas unitarias?

## Tips

Para generar un informe de cobertura de pruebas unitarias, simplemente pase la opción --coverage al comando de prueba. 
Por ejemplo, si su comando de prueba es npm run test, el comando para generar un informe de cobertura sería npm run test -- --coverage, con un -- adicional antes de la opción --coverage. 
Por lo tanto, el script correcto para ejecutar es 
```shell
npm run test -- --coverage
```
