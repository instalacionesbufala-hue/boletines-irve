# Traspaso del proyecto

Esto es lo que hay que saber para seguir con la aplicación en otro ordenador, o
para que la coja otra persona. El detalle técnico está en `README.md`; cómo se
usa, en `LEEME.md`; qué comprobar antes de subir un cambio, en `COMPROBAR.md`.

Puesto al día el 19 de noviembre de 2026.

## Qué es

Una aplicación que genera el expediente completo de un punto de recarga de
vehículo eléctrico para la Comunidad de Madrid a partir de un solo formulario:
CIE, MTD, anexo de la ITC-BT-52, esquema unifilar, solicitud BT-1134F1,
autorización del titular y, si hace falta, el anexo de inspección del garaje.

Funciona **entera dentro del navegador** (`docs/`, publicada en GitHub Pages).
`nucleo.py` es el mismo motor escrito en Python: no se usa para trabajar, sirve
para comprobar que el del navegador sigue dando exactamente lo mismo.

## Dónde se publica

Hay dos repositorios y **no son iguales**:

| Rama de aquí | Repositorio | Qué lleva |
|---|---|---|
| `main` | `instalacionesbufala-hue/boletines-irve` | Todo. Es **la publicación por defecto**: lo que sube `publicar.bat`. |
| `cie` | `chargebycesar/cie` | La aplicación **sin** el enlace con el panel de Búfala. Se quedó en el esquema 4a y solo se actualiza a propósito. |

La página que se usa es
`https://instalacionesbufala-hue.github.io/boletines-irve/`.

Para llevar algo a `chargebycesar/cie`: `git checkout cie`, `git cherry-pick`
de los cambios que se quieran —ninguno del panel—, pasar las comprobaciones,
`git push cie cie:main` y volver a `main`. `publicar.py` no se usa desde esa
rama.

## No se publica solo desde aquí

En `instalacionesbufala-hue/boletines-irve` escribe también **la sesión del
panel de Búfala**, por *pull request*. Suyo es el enlace con el panel: abrir una
obra con sus datos (`#obra=`) y avisar al panel cuando se generan los
documentos. Está contado en el `README.md`, «El enlace con el panel de Búfala».

Tres cosas que salen de ahí:

- **Nunca se fuerza un envío.** Si la otra sesión ha subido algo, `publicar.py`
  lo baja y lo junta. El sello de versión choca siempre y lo resuelve solo; si
  choca algo más, pregunta, con el no por defecto.
- **Lo que cambie en el formulario le afecta.** El panel rellena los campos por
  su nombre. Si se renombra o se quita uno, o cambia lo que la página hace al
  cargarlos, hay que avisar: con una *issue* etiquetada `para-panel` en
  `instalacionesbufala-hue/panel-src`, que es como se hablan esas sesiones.
- **Tras publicar, mirar que el despliegue termina.** Ya se quedó colgado una
  vez y bloqueó los dos siguientes.

## Los dos paquetes

| Archivo | Qué lleva | Para qué |
|---|---|---|
| `boletines-madrid-completo.zip` | El proyecto entero, con su historial de Git, tu configuración y los impresos originales | Seguir trabajando tú, en otro ordenador |
| `boletines-madrid-publico.zip` | Solo lo que ya está publicado en GitHub | Dárselo a alguien |

### Lo que NO va en ninguno de los dos

- `salida/` — expedientes ya generados. Llevan datos de clientes reales.
- `node_modules/` — 25 MB que se rehacen con `npm install`.
- `__pycache__/` — lo rehace Python solo.

### Lo que va SOLO en el completo

- `config.json` y `prueba.json` — los datos de tu empresa y del técnico.
- `fuente/` — los impresos tal como llegaron de Industria, antes de limpiarlos.
  Llevan dentro datos de otros trabajos: otro instalador, otros clientes. Por
  eso están fuera del repositorio y por eso no van en el paquete público.
- `.git` — el historial, con las dos ramas y los dos destinos. Con él,
  `publicar.bat` sigue funcionando desde el ordenador nuevo sin volver a
  configurar nada.
- `.specify/`, `specs/`, `.claude/` y `CLAUDE.md` — las especificaciones de cada
  cambio y las instrucciones para el asistente. Están en el `.gitignore`: no se
  suben a ningún repositorio.

## Para arrancarlo en otro ordenador

1. Descomprimir donde sea (no en una carpeta sincronizada con OneDrive: Git y
   OneDrive se pelean).
2. Instalar Python 3.11 o más nuevo y Node 18 o más nuevo.
3. Dentro de la carpeta:

   ```bash
   pip install -r requisitos.txt
   npm install
   ```

4. Para trabajar, doble clic en `iniciar.bat`. Para publicar un cambio, doble
   clic en `publicar.bat`.

Si has cogido el paquete completo, el `git push` pedirá las credenciales de
GitHub la primera vez. Eso lo hace el usuario; yo no manejo sus claves.

## Antes de dar nada por bueno

```bash
npm run comprobar   # 22 casos límite, ninguno debe romper
npm run comparar    # los dos motores tienen que dar lo mismo, campo por campo
npm run formato     # el CIE lleva todo lo del impreso oficial, el aviso y el identificador
python herramientas/rastrear.py   # que las plantillas no lleven datos de nadie
```

Las cuatro tienen que salir limpias. Están explicadas en `COMPROBAR.md`.

`publicar.py` revisa además, fichero por fichero, que no se cuele un DNI, un
CIF, un CUPS, un correo o un móvil que no toque, y si encuentra uno no sube
nada. Los datos de prueba tienen que ser inventados a la vista: `12345678Z`,
`B00000000`, `prueba@ejemplo.es`.

## Lo último que se ha añadido

De octubre y noviembre de 2026. Cada cosa tiene su apartado en el `README.md`.

- **Protecciones añadidas al cuadro** (DPN 2×32, 2×16 con base schuko): salen
  dibujadas en el unifilar y declaradas en el MTD, en la fila 2 de los
  dispositivos de mando y protección, con la sección de la fila 1 y 6 kA.
- **Titular empresa.** Se reconoce por el NIF o por el nombre. La razón social
  va en el nombre, los apellidos dejan de ser obligatorios y se pide un
  representante legal, que sale en el CIE y en la solicitud. En lo que firma el
  cliente —autorización y anexo del garaje— va solo el nombre del representante.
- **Ningún dato se recorta.** Si no cabe con la letra del impreso, se encoge.
- **Anexo del garaje:** la localidad, el código postal y la provincia de la
  comunidad salen de la dirección de la instalación, y se pueden cambiar allí.
  El domicilio no.
- **El CUPS se comprueba mientras se escribe**, y si está mal se pregunta antes
  de generar.
- **La marca y el modelo del cargador vienen escritos** y se guardan con los
  demás valores técnicos.
- **Empresa predeterminada**, elegida en Configuración: es la que firma al
  vaciar el formulario. El desplegable la cambia solo para ese expediente.
- **Los 179 municipios de Madrid** van dentro de la aplicación. El código
  postal se pone solo cuando el municipio tiene uno único. Para eso no se
  consulta nada fuera: el callejero del IGN sigue siendo aparte y con botón.
- **Esquema 4a, vivienda unifamiliar:** sin plaza ni planta, contador en CPM
  —también en el MTD—, garaje individual y línea desde el cuadro de la
  vivienda. Se declara en la configuración (`por_esquema`), no en el programa.
- **El enlace con el panel de Búfala**, hecho por la otra sesión. Trae consigo
  la única salida automática de la aplicación: al generar los documentos de
  una obra abierta desde el panel, se le manda el número de obra, la firma del
  enlace y el identificador del CIE. Ningún dato del cliente.
- **`publicar.py` junta lo que haya subido otra sesión** sin pisarlo.

## Las trampas que costaron tiempo

Están todas contadas en el `README.md`, pero conviene saber que existen antes de
tocar nada:

- **Los impresos oficiales no venían en blanco.** Traían datos de otros trabajos
  dentro. `herramientas/limpiar_plantillas.py` los deja limpios, en doce pasos
  que van en ese orden por algo.
- **Los impresos oficiales no se modifican.** Solo se rellenan los huecos que
  traen; lo que no tiene hueco no se pone.
- **El gris de las leyendas va solo en el dibujo, nunca en el `/DA`.** Si se
  pone en los dos, lo que escriba el cliente encima sale gris.
- **Hay recuadros guardados del revés** (`y1 < y0`). MuPDF los endereza solo y
  otros visores no: el campo sale en blanco solo en el ordenador del cliente.
- **El MTD tiene 1 631 campos y 1 611 sin apariencia**, así que `removeField()`
  de pdf-lib revienta. Hay que quitar los widgets a mano.
- **La página 4 del MTD está apaisada** y sus huecos vienen con el ancho y el
  alto cambiados: el tamaño de letra se mide contra el lado largo.
- **La casilla del CIE no se marca con `/Yes`**, sino con `/S#ED`.
- **GitHub Pages cachea.** Por eso `publicar.py` sella cada archivo con `?v=`
  antes de subirlo. Ante un «a mí no me sale», sospechar primero de la caché.
- **Vaciar el formulario no vacía un campo oculto**, y un desplegable vuelve a
  la opción con la que se creó, no a la primera. Las dos cosas dieron un fallo
  que no se veía en pantalla.
- **Escribir un valor desde el programa no avisa a nadie.** Lo que dependa de
  ese campo hay que llamarlo a mano: así se quedó la comunidad sin su código
  postal.
- **El mensaje del cambio se le da a `publicar.py` por la entrada, no como
  argumento.** Como argumento acabó de sello de versión, con espacios, y
  rompió el sellado.
