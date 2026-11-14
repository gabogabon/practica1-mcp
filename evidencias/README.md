# Práctica 1: Conectar servidores MCP a un agente

Ingeniería en Sistemas Computacionales · Model Context Protocol (MCP)

## 1. Descripción

Esta práctica conecta **servidores MCP locales** a un agente de programación y resuelve con ellos tareas reales, comprobando qué herramienta se usó en cada paso y qué límites impone cada servidor.

- **Agente usado:** Google Antigravity (aplicación de escritorio), versión **v2.31.2** según la página de descarga (`TODO: confirmar en Settings o Acerca de`), con el modelo **Gemini 3.8 Flash**. No se usó Claude Code: se instaló, pero pide una suscripción o crédito de API que no tenía, y la guía permite Antigravity como alternativa equivalente.
- **Sistema operativo:** Windows 11, con terminal **Git Bash**.
- **Servidores MCP:** `filesystem` y `playwright` (obligatorios) y `memory` (Actividad D).
- **Modo de trabajo:** el agente pidió mi aprobación antes de cada llamada a una herramienta (*Allow using this MCP tool?*) y la aprobé una por una con *Yes, allow this time*.

### Estructura del repositorio

```
practica1-mcp/
├── README.md
├── .gitignore
├── .agents/mcp_config.json      (configuración de Antigravity)
├── datos/calificaciones.csv
├── salida/
│   ├── reporte.md               (Actividad A)
│   ├── entidades.csv            (Actividad B)
│   ├── captura.png              (Actividad B, captura de playwright, repetida en una segunda consulta)
│   └── memoria.jsonl            (Actividad D, grafo de memoria)
├── instrucciones/               (texto de cada instrucción dada al agente)
│   ├── actividad_a.txt
│   ├── actividad_b.txt
│   ├── actividad_c.txt
│   └── actividad_d.txt
└── evidencias/                  (capturas de pantalla)
```

## 2. Requisitos y reproducción

### Requisitos

| Requisito | Versión usada |
|---|---|
| Node.js (incluye `npx`) | v26.13.1 (npm 11.20.0). Se requiere 18 o superior |
| Git | 2.55.0 (Windows) |
| Google Chrome | Lo usa el servidor `playwright` |
| Google Antigravity | v2.31.2, con cuenta personal de Google (`TODO: confirmar versión`) |
| Cuenta de GitHub | Para publicar el repositorio |

Los servidores se ejecutan con `npx -y`, así que no hay que instalarlos de antemano. La primera ejecución tarda más porque descarga el paquete.

### Pasos para repetir la práctica desde cero

1. **Verificar herramientas** en Git Bash:

   ```
   node --version
   npx --version
   git --version
   ```

2. **Crear el espacio de trabajo:**

   ```
   mkdir practica1-mcp
   cd practica1-mcp
   mkdir datos salida instrucciones evidencias .agents
   git init
   ```

3. **Crear `datos/calificaciones.csv`** con este contenido exacto y `.gitignore` con la línea `.playwright-mcp/`:

   ```
   matricula,nombre,parcial1,parcial2,parcial3
   A001,Ana Torres,9.0,8.5,9.5
   A002,Bruno Salas,6.0,7.0,5.5
   A003,Carla Mendez,10.0,9.5,9.0
   A004,Diego Ruiz,5.0,6.5,5.0
   A005,Elena Castro,8.0,8.0,8.5
   A006,Fabian Ortiz,7.5,6.0,7.0
   A007,Gabriela Luna,9.5,10.0,9.0
   A008,Hector Vega,4.5,5.0,6.5
   ```

4. **Crear `.agents/mcp_config.json`** y **cambiar la ruta** por la de tu propia carpeta. Es el único valor que hay que modificar: sustituye `C:/Users/TU_USUARIO/practica1-mcp` en `filesystem` y en `MEMORY_FILE_PATH` de `memory`.

   ```json
   {
     "mcpServers": {
       "filesystem": {
         "command": "cmd",
         "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-filesystem", "C:/Users/TU_USUARIO/practica1-mcp"]
       },
       "playwright": {
         "command": "cmd",
         "args": ["/c", "npx", "-y", "@playwright/mcp@latest"]
       },
       "memory": {
         "command": "cmd",
         "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-memory"],
         "env": { "MEMORY_FILE_PATH": "C:/Users/TU_USUARIO/practica1-mcp/salida/memoria.jsonl" }
       }
     }
   }
   ```

   Las barras normales (`/`) son válidas en JSON y las entiende Node en Windows. En macOS o Linux se usa `"command": "npx"` sin `cmd /c`.

5. **Abrir la carpeta en Antigravity** y revisar los servidores en **Customizations > Installed > MCP Servers**. Deben aparecer los tres con punto verde y su lista de herramientas.

   > **Importante:** la aplicación puede leer un archivo de configuración distinto al del proyecto. El icono de copiar de la sección *MCP Servers* da la ruta del archivo que realmente lee. Si los servidores no aparecen, el mismo JSON debe estar también en ese archivo (con la ruta real escrita, sin variables).

6. **Dejar la aprobación de herramientas en modo "preguntar"** (no usar *always allow*), para ver cada llamada.
7. **Guardar cada instrucción** en `instrucciones/` y ejecutar las actividades A, B, C y D en conversaciones nuevas dentro del proyecto, con un commit por actividad.
8. **Publicar:**

   ```
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/practica1-mcp.git
   git push -u origin main
   ```

## 3. Servidores MCP

Cada servidor es un programa independiente que el agente (el *host*) lanza como proceso hijo y al que se conecta con un cliente MCP, mediante JSON-RPC sobre **stdio**. El modelo nunca toca el disco ni el navegador: pide una acción y el servidor la ejecuta.

| Servidor | Paquete de npm | Qué permite | Herramientas |
|---|---|---|---|
| `filesystem` | `@modelcontextprotocol/server-filesystem` | Leer, escribir, listar y buscar archivos solo dentro de la carpeta indicada al arrancar | 16 |
| `playwright` | `@playwright/mcp` | Controlar un navegador: abrir páginas, leer su contenido, ejecutar JavaScript y tomar capturas | 25 |
| `memory` | `@modelcontextprotocol/server-memory` | Memoria persistente en forma de grafo de conocimiento (Actividad D) | 9 |

**Cómo se configuraron:** los tres se declaran en `.agents/mcp_config.json` con el comando `cmd /c npx -y <paquete>` (ver sección 2). `filesystem` recibe como argumento la ruta de la carpeta de la práctica, que es el único directorio al que tiene acceso. `memory` recibe `MEMORY_FILE_PATH` para guardar su grafo en `salida/memoria.jsonl`.

**Herramientas expuestas** (según la pantalla de servidores de Antigravity):

- **`filesystem` (16):** `read_file`, `read_text_file`, `read_media_file`, `read_multiple_files`, `write_file`, `append_file`, `edit_file`, `create_directory`, `list_directory`, `list_directory_with_sizes`, `directory_tree`, `move_file`, `copy_file`, `search_files`, `get_file_info`, `list_allowed_directories`.
- **`playwright` (25):** `browser_close`, `browser_resize`, `browser_console_messages`, `browser_handle_dialog`, `browser_emulate_media`, `browser_evaluate`, `browser_file_upload`, `browser_drop`, `browser_find`, `browser_fill_form`, `browser_press_key`, `browser_type`, `browser_navigate`, `browser_navigate_back`, `browser_network_requests`, `browser_network_request`, `browser_run_code_unsafe`, `browser_take_screenshot`, `browser_snapshot`, `browser_click`, `browser_drag`, `browser_hover`, `browser_select_option`, `browser_tabs`, `browser_wait_for`.
- **`memory` (9):** `create_entities`, `create_relations`, `add_observations`, `delete_entities`, `delete_observations`, `delete_relations`, `read_graph`, `search_nodes`, `open_nodes`.

**Los dos servidores de la práctica conectados en Antigravity:**

![Servidores conectados](evidencias/01_servidores.png)

**Los tres servidores conectados, con todas sus herramientas:**

![Tres servidores conectados](evidencias/act_d_1.png)

## 4. Actividades A, B y C

### Actividad A: lectura y escritura de archivos

**Servidor usado:** `filesystem` (todas las llamadas fueron a herramientas de este servidor).
**Agente:** Google Antigravity App, modelo Gemini 3.8 Flash. Aprobación de herramientas: cada llamada MCP pidió confirmación ("Allow using this MCP tool?") y se aprobó con la opción 1, *Yes, allow this time*.

#### Instrucción dada al agente

Guardada en [`instrucciones/actividad_a.txt`](instrucciones/actividad_a.txt):

```
Usa unicamente las herramientas del servidor MCP filesystem.
1) Lista el contenido de la carpeta datos.
2) Lee datos/calificaciones.csv.
3) Calcula el promedio final de cada alumno (media de los tres
parciales, con dos decimales) e indica si aprueba (promedio >= 6.0).
4) Escribe el resultado en salida/reporte.md como una tabla Markdown
ordenada de mayor a menor promedio, y al final agrega el promedio
del grupo.
Responde con un resumen breve de lo que hiciste.
```

#### Secuencia de herramientas

En Windows, la ruta aparece con barras invertidas dobles (`C:\\Users\\...`) porque los argumentos se muestran en formato JSON.

| # | Herramienta | Argumentos (`path`) | Resultado |
|---|---|---|---|
| 1 | `list_allowed_directories` | (sin argumentos) | *Pendiente: no se capturó el resultado* |
| 2 | `list_directory` | *Pendiente: no se capturaron los argumentos (según el resumen del agente, la carpeta `datos`)* | Según el resumen del agente: encontró `calificaciones.csv` |
| 3 | `read_text_file` | `C:\\Users\\gabri\\practica1-mcp\\datos\\calificaciones.csv` | Contenido del CSV: encabezado y 8 alumnos (A001 a A008) |
| 4 | `list_directory` | `C:\\Users\\gabri\\practica1-mcp` | `[DIR] .agents`, `[DIR] .git`, `[FILE] .gitignore`, `[DIR] datos`, `[DIR] evidencias`, `[DIR] instrucciones`, `[DIR] salida` |
| 5 | `list_directory` | `C:\\Users\\gabri\\practica1-mcp\\instrucciones` | *Pendiente: no se capturó el resultado* |
| 6 | `read_text_file` | `C:\\Users\\gabri\\practica1-mcp\\instrucciones\\actividad_a.txt` | *Pendiente: no se capturó el resultado* |
| 7 | `list_directory` | `C:\\Users\\gabri\\practica1-mcp\\salida` | *Pendiente: no se capturó el resultado* |
| 8 | `write_file` | `path`: `C:\\Users\\gabri\\practica1-mcp\\salida\\reporte.md`; `content`: la tabla Markdown completa | Escribió `salida/reporte.md` (el archivo existe, ver abajo) |

> El orden se reconstruyó a partir de las capturas, tomadas en el orden en que aparecieron en pantalla.

#### Observaciones

- **Llamadas adicionales.** El agente hizo por su cuenta llamadas que la instrucción no pedía: `list_allowed_directories` al inicio (comprobar qué carpetas puede ver el servidor), y listados de la raíz del proyecto, `instrucciones` y `salida`, además de leer `instrucciones/actividad_a.txt`. Todas son de `filesystem` y de solo lectura, salvo la escritura final, que fue la pedida.
- **Aprobaciones.** La aplicación se detuvo con *Waiting for user input* antes de cada llamada. Se eligió siempre *Yes, allow this time*, para ver y aprobar cada una por separado.
- **Duración.** El resumen del agente indica "Worked for 40m"; ese tiempo incluye las esperas de aprobación.
- **Resumen del agente.** Dijo haber listado `datos`, leído el CSV con `read_text_file`, calculado los promedios (media de los tres parciales, dos decimales, aprueba con >= 6.0, promedio del grupo 7.50) y escrito el reporte con `write_file`.

#### Resultado: `salida/reporte.md`

```markdown
# Reporte de Calificaciones

| Matrícula | Nombre | Parcial 1 | Parcial 2 | Parcial 3 | Promedio | ¿Aprueba? |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| A003 | Carla Mendez | 10.0 | 9.5 | 9.0 | 9.50 | Sí |
| A007 | Gabriela Luna | 9.5 | 10.0 | 9.0 | 9.50 | Sí |
| A001 | Ana Torres | 9.0 | 8.5 | 9.5 | 9.00 | Sí |
| A005 | Elena Castro | 8.0 | 8.0 | 8.5 | 8.17 | Sí |
| A006 | Fabian Ortiz | 7.5 | 6.0 | 7.0 | 6.83 | Sí |
| A002 | Bruno Salas | 6.0 | 7.0 | 5.5 | 6.17 | Sí |
| A004 | Diego Ruiz | 5.0 | 6.5 | 5.0 | 5.50 | No |
| A008 | Hector Vega | 4.5 | 5.0 | 6.5 | 5.33 | No |

**Promedio del grupo:** 7.50
```

**Verificación manual:**

- 8 alumnos, ordenados de mayor a menor promedio.
- 6 aprueban y 2 no: Diego Ruiz (5.50) y Hector Vega (5.33).
- Recalculé cada promedio a mano y coinciden. La suma de los ocho promedios es 60.00, así que el promedio del grupo es 7.50.
- Carla Mendez y Gabriela Luna empatan en 9.50, por lo que cualquier orden entre ellas es correcto.

#### Capturas

**Aprobación de la primera herramienta (`list_allowed_directories`):**
![Aprobación list_allowed_directories](evidencias/02_actividad_a_aprobacion.png)

**Solicitud de aprobación de `list_directory`:**
![Aprobación list_directory](evidencias/02_actividad_a_1.png)

**`read_text_file` sobre `calificaciones.csv` (argumentos):**
![read_text_file argumentos](evidencias/03_actividad_a_csv_args.png)

**Aprobación de `read_text_file`:**
![Aprobación read_text_file](evidencias/04_actividad_a_2.png)

**`read_text_file` con su resultado (contenido del CSV):**
![read_text_file resultado](evidencias/04b_actividad_a_csv_resultado.png)

**Aprobación de `list_directory`:**
![Aprobación list_directory](evidencias/05_actividad_a_3.png)

**`list_directory` de la raíz del proyecto, con resultado:**
![list_directory raíz](evidencias/05b_actividad_a_raiz.png)

**`list_directory` de `instrucciones`:**
![list_directory instrucciones](evidencias/05c_actividad_a_instrucciones.png)

**Aprobación de `list_directory`:**
![Aprobación list_directory](evidencias/06_actividad_a_4.png)

**`read_text_file` sobre `actividad_a.txt`:**
![read_text_file actividad_a](evidencias/06b_actividad_a_lee_instruccion.png)

**Aprobación de `read_text_file`:**
![Aprobación read_text_file](evidencias/07_actividad_a_5.png)

**`list_directory` de `salida`:**
![list_directory salida](evidencias/07b_actividad_a_salida.png)

**Aprobación de `write_file`:**
![Aprobación write_file](evidencias/08_actividad_a_6.png)

**`write_file` (argumentos con el contenido del reporte):**
![write_file argumentos](evidencias/08b_actividad_a_write_args.png)

**Resumen final del agente:**
![Resumen final](evidencias/09_actividad_a_resumen.png)

### Actividad B: de una página web a un CSV

**Servidores usados:** `playwright` (navegador) y `filesystem` (guardar el archivo).
**Agente:** Google Antigravity App, modelo Gemini 3.8 Flash. Todas las llamadas pidieron aprobación ("Allow using this MCP tool?") y se aprobaron con *Yes, allow this time*.

#### Instrucción dada al agente

Guardada en [`instrucciones/actividad_b.txt`](instrucciones/actividad_b.txt):

```
Usa las herramientas MCP de playwright y filesystem.
1) Abre en el navegador la pagina
https://es.wikipedia.org/?curid=5897912
2) Localiza la tabla principal con las 32 entidades federativas y
extrae, para cada una: entidad, superficie en km2, poblacion y
densidad (hab/km2).
3) Guarda los datos en salida/entidades.csv con los encabezados
entidad,superficie_km2,poblacion,densidad_hab_km2, con numeros
sin separadores de miles y punto decimal.
4) Toma una captura de pantalla de la pagina.
5) Cierra el navegador.
Responde con un resumen breve: cuantas filas guardaste y que año de
poblacion usa la tabla.
```

#### Secuencia de herramientas

El orden se reconstruyó a partir de las capturas. De las llamadas de exploración (pasos 3 a 10) solo se capturaron los argumentos y la solicitud de aprobación, no el resultado que devolvió cada una.

| # | Servidor | Herramienta | Qué hizo | Capturas |
|---|---|---|---|---|
| 1 | filesystem | `list_allowed_directories` | Comprobó qué carpeta puede ver el servidor. Resultado: `C:\Users\gabri\practica1-mcp` | `act_b_1`, `act_b_2` |
| 2 | playwright | `browser_navigate` | Abrió `https://es.wikipedia.org/?curid=5897912` | `act_b_3` |
| 3 | playwright | `browser_evaluate` | JavaScript: listó las tablas `table.wikitable` con sus encabezados y número de filas | `act_b_4`, `act_b_5` |
| 4 | playwright | `browser_evaluate` | JavaScript: listó **todas** las tablas (clase, id, caption y primeras filas de encabezado) | `act_b_6`, `act_b_7` |
| 5 | playwright | `browser_evaluate` | JavaScript: tomó la primera tabla y leyó 7 filas de muestra con `tagName`, `colspan` y `rowspan` | `act_b_8`, `act_b_9` |
| 6 | playwright | `browser_evaluate` | JavaScript: leyó el HTML de las primeras 4 filas | `act_b_10`, `act_b_11` |
| 7 | playwright | `browser_evaluate` | JavaScript: leyó el texto de las últimas 5 filas | `act_b_12`, `act_b_13` |
| 8 | playwright | `browser_evaluate` | JavaScript: extrajo las columnas 3, 4, 6 y 7 de cada fila de datos (omitiendo 3 filas de encabezado) | `act_b_14`, `act_b_15` |
| 9 | playwright | `browser_evaluate` | JavaScript: repitió la extracción limpiando los números (quita espacios, incluidos los no separables, y cambia coma por punto) | `act_b_16`, `act_b_17` |
| 10 | playwright | `browser_evaluate` | JavaScript: leyó el texto de las notas al pie (`cite_note-Conteo_2015...`) para confirmar el año de cada columna | `act_b_18` |
| 11 | filesystem | `create_directory` | Creó `salida` (ya existía) | `act_b_19` |
| 12 | filesystem | `write_file` | Escribió `salida/entidades.csv` | `act_b_20` |
| 13 | playwright | `browser_take_screenshot` | Pidió guardar `salida/captura.png` (`fullPage: false`, `scale: css`). El archivo no apareció en el proyecto, por lo que la captura se repitió después en una consulta aparte (ver «Observaciones» y «Problemas encontrados») | `act_b_21` |
| 14 | playwright | `browser_close` | Cerró el navegador | `act_b_22` |
| 15 | filesystem | `read_file` | Leyó `salida/entidades.csv` | `act_b_23` |

#### En qué paso cambió de servidor

- Empezó con una llamada breve a `filesystem` (paso 1) para comprobar su carpeta permitida.
- El cambio principal fue del **paso 10 al 11**: tras terminar de leer y limpiar la tabla con `playwright`, pasó a `filesystem` (`create_directory`, y después `write_file`) para guardar el CSV. `playwright` solo ve la página, y `filesystem` es el único que puede escribir en la carpeta de la práctica.
- Después volvió a `playwright` para la captura de pantalla y para cerrar el navegador (pasos 13 y 14), y terminó con `filesystem` para leer el CSV (paso 15).

#### Cómo obtuvo los datos de la tabla

**Ejecutando JavaScript** con `browser_evaluate`. No usó `browser_snapshot`. El agente fue avanzando por tanteo:

1. Buscó las tablas con clase `wikitable` y luego listó todas las tablas de la página.
2. Eligió la primera tabla (`table[0]`) y examinó sus filas (texto, HTML, `colspan` y `rowspan`), tanto las primeras como las últimas.
3. Con esa estructura, tomó por posición las columnas de **entidad** (3), **superficie** (4), **población** (6) y **densidad** (7), y saltó la 5, que es el porcentaje de superficie, y las tres columnas de posición.
4. Limpió los números dentro del mismo script: quitó espacios (normales y no separables) y cambió la coma decimal por punto.

#### Resultado: `salida/entidades.csv`

Primeras líneas (el archivo completo está en el repositorio):

```csv
entidad,superficie_km2,poblacion,densidad_hab_km2
Chihuahua,247455,3741869,14
Sonora,179355,2944840,16
Coahuila,151562,3146771,19
Durango,123317,1832650,14
Oaxaca,93757,4132148,42
Jalisco,80295,8348151,100
```

**Resumen del agente:** guardó **32 filas** (más el encabezado) y la tabla usa la estimación de población de **2020** (columna «Estimación (2020)»). La columna de densidad está rotulada como **2015** en la página.

**Comprobaciones sobre el archivo:**

- 33 líneas: 1 encabezado y 32 entidades.
- Todas las filas tienen 4 campos, y los números no llevan separadores de miles ni comas.
- La primera entidad es Chihuahua con 247455 km², como se esperaba.
- La suma de la columna de población es 126 014 024, que coincide con el total nacional del Censo 2020.

#### Verificación manual contra la página

Comparé el CSV con la tabla de Wikipedia (capturas `act_b_25` y `act_b_26`). Revisé tres filas, elegidas por ser las que más fácil fallan (números largos, cifras de cuatro dígitos sin separador y una fila con una capital «N/A»):

| Entidad | En el CSV (superficie, población, densidad) | En la página | ¿Coincide? |
|---|---|---|---|
| Estado de México | 22351, 16992418, 724 | 22 351, 16 992 418, 724 | Sí |
| Colima | 5627, 731391, 126 | 5627, 731 391, 126 | Sí |
| Ciudad de México | 1495, 9209944, 5966 | 1495, 9 209 944, 5966 | Sí |

#### Observaciones

- **Aviso de Chrome:** al abrirse la ventana controlada por el agente apareció la barra «No se admite el indicador de línea de comandos (--disable-blink-features=AutomationControlled)...». Es normal cuando Playwright controla Chrome y no impide el funcionamiento.
- **Captura de pantalla:** en la secuencia original el agente pidió guardar `salida/captura.png` con `fullPage: false` (solo la parte visible de la página), pero el archivo no apareció en la carpeta del proyecto. Para cumplir el paso 4 de la actividad se repitió la captura en una **consulta aparte** con `playwright`, indicando la ruta de destino, y esta vez quedó en `salida/captura.png`. De esa segunda consulta **no tengo capturas de pantalla** (ni de la instrucción, ni de las aprobaciones, ni de los argumentos), por lo que la única evidencia de ella es el propio archivo `salida/captura.png` y el `act_b_21` de la primera llamada.
- **Llamadas extra:** el agente hizo exploraciones y comprobaciones que la instrucción no pedía (`list_allowed_directories`, lectura de notas al pie, `create_directory` y `read_file` del CSV). Todas eran de `playwright` o `filesystem`.

#### Capturas

**1. `list_allowed_directories`: aprobación y resultado**
![Aprobación list_allowed_directories](evidencias/act_b_1.png)
![Resultado list_allowed_directories](evidencias/act_b_2.png)

**2. `browser_navigate`: argumentos y aprobación**
![browser_navigate](evidencias/act_b_3.png)

**3 a 9. `browser_evaluate`: exploración y extracción de la tabla (argumentos y aprobaciones)**
![evaluate 3 args](evidencias/act_b_4.png)
![evaluate 3 aprobación](evidencias/act_b_5.png)
![evaluate 4 args](evidencias/act_b_6.png)
![evaluate 4 aprobación](evidencias/act_b_7.png)
![evaluate 5 args](evidencias/act_b_8.png)
![evaluate 5 aprobación](evidencias/act_b_9.png)
![evaluate 6 args](evidencias/act_b_10.png)
![evaluate 6 aprobación](evidencias/act_b_11.png)
![evaluate 7 args](evidencias/act_b_12.png)
![evaluate 7 aprobación](evidencias/act_b_13.png)
![evaluate 8 args](evidencias/act_b_14.png)
![evaluate 8 aprobación](evidencias/act_b_15.png)
![evaluate 9 args](evidencias/act_b_16.png)
![evaluate 9 aprobación](evidencias/act_b_17.png)

**10. `browser_evaluate` sobre las notas al pie**
![evaluate notas al pie](evidencias/act_b_18.png)

**11. `create_directory` (filesystem)**
![create_directory](evidencias/act_b_19.png)

**12. `write_file` (filesystem): cambio de servidor, escribe `entidades.csv`**
![write_file](evidencias/act_b_20.png)

**13. `browser_take_screenshot`**
![browser_take_screenshot](evidencias/act_b_21.png)

**14. `browser_close`**
![browser_close](evidencias/act_b_22.png)

**15. `read_file` sobre `entidades.csv`**
![read_file](evidencias/act_b_23.png)

**Resumen final del agente**
![Resumen del agente](evidencias/act_b_24.png)

**Tabla en Chrome, filas 1 a 17 (usada en la verificación)**
![Tabla en Chrome parte 1](evidencias/act_b_25.png)

**Tabla en Chrome, filas 18 a 32 (usada en la verificación)**
![Tabla en Chrome parte 2](evidencias/act_b_26.png)

**Captura tomada por `playwright`** (obtenida en la segunda consulta; ver «Observaciones»)
![Captura de playwright](salida/captura.png)

### Actividad C: los límites del servidor

**Servidor usado:** `filesystem`.
**Agente:** Google Antigravity App. Cada llamada pidió aprobación ("Allow using this MCP tool?") y se aprobó con *Yes, allow this time*, incluidas las dos que terminaron en error: el rechazo no lo provocó el usuario, llegó después de aprobar.

#### Instrucción dada al agente

Guardada en [`instrucciones/actividad_c.txt`](instrucciones/actividad_c.txt):

```
Usa unicamente las herramientas del servidor MCP filesystem.
1) Llama a list_allowed_directories.
2) Intenta leer un archivo que este fuera de la carpeta de la
practica (en Windows: C:/Windows/win.ini; en macOS o Linux:
/etc/hosts).
3) Intenta listar el directorio padre de tu carpeta permitida.
No uses ninguna otra via si falla. Reporta textualmente que
respondio el servidor en cada intento.
```

#### Secuencia de herramientas y respuestas del servidor

| # | Herramienta | Argumentos | Respuesta del servidor |
|---|---|---|---|
| 1 | `list_allowed_directories` | `{}` | `Allowed directories: C:\Users\gabri\practica1-mcp` |
| 2 | `read_text_file` | `{"path": "C:/Windows/win.ini"}` | `Encountered error in tool execution: Access denied - path outside allowed directories: C:\Windows\win.ini not in C:\Users\gabri\practica1-mcp` |
| 3 | `list_directory` | `{"path": "C:\\Users\\gabri"}` | `Encountered error in tool execution: Access denied - path outside allowed directories: C:\Users\gabri not in C:\Users\gabri\practica1-mcp` |

#### Mensajes de error exactos

**Paso 2, leer un archivo fuera de la carpeta:**

```
Encountered error in tool execution: Access denied - path outside allowed directories: C:\Windows\win.ini not in C:\Users\gabri\practica1-mcp
```

**Paso 3, listar el directorio padre:**

```
Encountered error in tool execution: Access denied - path outside allowed directories: C:\Users\gabri not in C:\Users\gabri\practica1-mcp
```

La parte `Access denied - path outside allowed directories: <ruta> not in <carpeta permitida>` es el mensaje del servidor `filesystem`. El prefijo `Encountered error in tool execution:` lo añade la aplicación al mostrar el error de la herramienta. Se observa además que el servidor normalizó la ruta del paso 2: se envió `C:/Windows/win.ini` y respondió con `C:\Windows\win.ini`.

#### ¿Quién impuso el límite: el modelo, el host o el servidor?

**El servidor.**

- **No fue el modelo.** El modelo sí intentó el acceso. En su razonamiento explicó que probaría leer `C:\Windows\win.ini`, fuera de la carpeta permitida, "to check for restricted access". Un modelo solo pide la acción, y por sí solo no puede leer ni bloquear nada.
- **No fue el host (Antigravity).** Antigravity mostró la solicitud de aprobación de cada llamada y yo la aprobé, así que no la bloqueó. Su papel fue pasar la petición al servidor y mostrar la respuesta.
- **Fue el servidor `filesystem`.** El error viene en la respuesta de la herramienta y cita la lista de carpetas permitidas (`C:\Users\gabri\practica1-mcp`), que se fijó al arrancar el servidor con la ruta del proyecto en `mcp_config.json`. Cualquier ruta fuera de esa lista se rechaza, sin importar qué pida el modelo.

Por eso es más seguro que el límite lo imponga el servidor que una instrucción escrita en el prompt: una instrucción es solo una petición al modelo, que puede equivocarse o ser engañado, mientras que la restricción del servidor se aplica siempre.

#### Observaciones

- El agente respetó "no uses ninguna otra vía": tras los dos errores no intentó leer el archivo ni listar la carpeta con otra herramienta, y su resumen solo menciona herramientas de `filesystem`.
- Entre las capturas, los resultados de los pasos 2 y 3 provienen del resumen que redactó el agente, donde reprodujo la respuesta del servidor como "respuesta textual".

#### Capturas

**1. `list_allowed_directories`: aprobación**
![Aprobación list_allowed_directories](evidencias/act_c_1.png)

**1. Resultado de `list_allowed_directories` y razonamiento del agente**
![Resultado list_allowed_directories](evidencias/act_c_2.png)

**2. `read_text_file` sobre `C:/Windows/win.ini`: argumentos**
![read_text_file argumentos](evidencias/act_c_3.png)

**2. `read_text_file`: aprobación**
![Aprobación read_text_file](evidencias/act_c_4.png)

**3. `list_directory` sobre el directorio padre: aprobación**
![Aprobación list_directory](evidencias/act_c_5.png)

**Resumen del agente, paso 1**
![Resumen paso 1](evidencias/act_c_6.png)

**Resumen del agente, paso 2 (error del servidor)**
![Resumen paso 2](evidencias/act_c_7.png)

**Resumen del agente, paso 3 (error del servidor)**
![Resumen paso 3](evidencias/act_c_8.png)

## 5. Actividad D

**Servidor elegido:** `memory` (paquete de npm `@modelcontextprotocol/server-memory`), un servidor de memoria persistente en forma de grafo de conocimiento.
**Agente:** Google Antigravity App, modelo Gemini 3.8 Flash. Todas las llamadas pidieron aprobación y se aprobaron con *Yes, allow this time*.

### Qué es y quién lo publica

- **Publicador:** los mantenedores del protocolo. Es uno de los servidores de referencia del repositorio oficial `modelcontextprotocol/servers` (carpeta `src/memory`), y se distribuye en npm con licencia MIT.
- **Qué hace:** guarda un grafo con **entidades** (nodos con nombre y tipo), **relaciones** (aristas dirigidas entre entidades) y **observaciones** (datos cortos asociados a una entidad). Ese grafo se conserva entre conversaciones.
- **Dónde guarda los datos:** en un único archivo de texto (JSON Lines) en mi equipo. Su ruta se define con la variable de entorno `MEMORY_FILE_PATH`; configuré `salida/memoria.jsonl` dentro del proyecto.

### Configuración

Se agregó al mismo archivo `.agents/mcp_config.json` de los otros dos servidores (en Windows, con `cmd /c`):

```json
"memory": {
  "command": "cmd",
  "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-memory"],
  "env": { "MEMORY_FILE_PATH": "C:/Users/TU_USUARIO/practica1-mcp/salida/memoria.jsonl" }
}
```

Tras recargar los servidores, los tres aparecen conectados (`filesystem`: 16 herramientas, `memory`: 9, `playwright`: 25).

### Herramientas que expone

Según la pantalla de servidores MCP, `memory` expone 9 herramientas:

| Herramienta | Para qué sirve |
|---|---|
| `create_entities` | Crear nodos (entidades) |
| `create_relations` | Crear relaciones dirigidas entre entidades |
| `add_observations` | Agregar datos a una entidad existente |
| `delete_entities` | Borrar entidades |
| `delete_relations` | Borrar relaciones |
| `delete_observations` | Borrar observaciones |
| `read_graph` | Leer todo el grafo |
| `search_nodes` | Buscar nodos por texto |
| `open_nodes` | Recuperar nodos concretos por nombre |

### Tarea que diseñé

Guardar en el grafo los resultados del grupo de la Actividad A y comprobar que la memoria **persiste entre conversaciones**:

- **Sesión 1:** el agente crea una entidad `Alumno` por cada estudiante, con observaciones de promedio final y si aprueba, y una entidad `Grupo` llamada `Practica1`, con una relación `pertenece_a` de cada alumno hacia el grupo.
- **Sesión 2 (conversación nueva):** sin entregarle los datos de nuevo, se le pregunta quiénes no aprobaron y quién tiene el promedio más alto, y debe contestar desde la memoria.

Instrucción completa, guardada en [`instrucciones/actividad_d.txt`](instrucciones/actividad_d.txt):

```
SESION 1
Usa unicamente las herramientas del servidor MCP memory.
Crea una entidad de tipo Alumno por cada fila, con observaciones
"promedio final: X" y "aprueba: si/no", y una entidad Grupo
"Practica1" con una relacion "pertenece_a" desde cada alumno.
Datos (promedio, aprueba): Carla Mendez 9.50 si; Gabriela Luna 9.50 si;
Ana Torres 9.00 si; Elena Castro 8.17 si; Fabian Ortiz 6.83 si;
Bruno Salas 6.17 si; Diego Ruiz 5.50 no; Hector Vega 5.33 no.
Al terminar, llama a read_graph y resume lo que guardaste.

SESION 2 (conversacion nueva)
Usa unicamente las herramientas del servidor MCP memory. Sin que yo
te de los datos de nuevo, dime que alumnos no aprobaron y quien tiene
el promedio mas alto. Indica que herramienta usaste para consultarlo.
```

### Secuencia de herramientas

**Sesión 1:**

| # | Herramienta | Qué hizo | Capturas |
|---|---|---|---|
| 1 | `create_entities` | Creó 8 entidades `Alumno` con sus observaciones y la entidad `Grupo` `Practica1` (sin observaciones) | `act_d_2` a `act_d_8` |
| 2 | `add_observations` | Volvió a enviar las mismas observaciones de los 8 alumnos | `act_d_9` a `act_d_13` |
| 3 | `create_relations` | Creó 8 relaciones `pertenece_a`, de cada alumno hacia `Practica1` | `act_d_14` a `act_d_18` |
| 4 | `read_graph` | Leyó el grafo completo y resumió su contenido | `act_d_19` a `act_d_21` |

**Sesión 2 (conversación nueva):**

| # | Herramienta | Qué hizo | Capturas |
|---|---|---|---|
| 1 | `read_graph` | Leyó el grafo y respondió desde lo almacenado | `act_d_22`, `act_d_23` |

> De estas llamadas solo se capturaron los argumentos y las solicitudes de aprobación, no la respuesta cruda de cada una. La solicitud de aprobación de `add_observations` no se capturó.

### Resultado

- **Sesión 1:** el agente resumió lo guardado: la entidad `Practica1`, los 8 alumnos con su promedio y si aprueban, y las 8 relaciones `pertenece_a`.
- **Sesión 2:** sin recibir los datos otra vez, el agente respondió usando únicamente `read_graph`:
  - No aprobaron **Diego Ruiz (5.50)** y **Hector Vega (5.33)**.
  - El promedio más alto es **9.50**, con empate entre **Carla Mendez** y **Gabriela Luna**.
- **Persistencia:** la respuesta de la Sesión 2 coincide con los datos de la Sesión 1 y con el `reporte.md` de la Actividad A, así que el grafo se conservó entre conversaciones.

### Verificación del archivo `salida/memoria.jsonl`

Revisé el archivo de memoria que generó el servidor:

- Tiene **17 líneas**: 9 entidades (8 `Alumno` y 1 `Grupo`) y 8 relaciones.
- Cada alumno tiene **solo dos observaciones** (`promedio final` y `aprueba`). Aunque el agente envió las observaciones dos veces (en `create_entities` y en `add_observations`), el servidor ignoró las repetidas y no hay duplicados.
- Los valores coinciden con los de `salida/reporte.md`.

Primeras líneas del archivo:

```json
{"type":"entity","name":"Practica1","entityType":"Grupo","observations":[]}
{"type":"entity","name":"Carla Mendez","entityType":"Alumno","observations":["promedio final: 9.50","aprueba: si"]}
{"type":"relation","from":"Carla Mendez","to":"Practica1","relationType":"pertenece_a"}
```

### Observaciones

- **Llamada redundante:** el agente guardó las observaciones dos veces (`create_entities` y luego `add_observations`) aunque la instrucción no lo pedía. No causó daño porque el servidor ignora los datos repetidos.
- **`read_graph` frente a `search_nodes`:** para consultar usó `read_graph`, que devuelve todo el grafo. Con 9 entidades es razonable; con un grafo grande convendría `search_nodes` u `open_nodes`, que traen solo lo necesario.
- **Datos de prueba:** los alumnos son ficticios, así que publicar `memoria.jsonl` no expone información personal.

### Por qué considero confiable este servidor

- **Origen:** es uno de los servidores de referencia que mantiene el proyecto del protocolo en su repositorio oficial, publicado en npm con licencia MIT, y de código abierto, por lo que se puede revisar.
- **Alcance limitado:** funciona en local, no necesita cuenta, claves ni acceso a internet para operar, y guarda todo en un archivo mío que puedo abrir, editar o borrar. No lo comprobé inspeccionando su código, pero la documentación lo describe así y el archivo `memoria.jsonl` generado es texto plano legible.
- **Control de cada llamada:** el agente me pidió aprobación antes de cada herramienta, así que vi qué se guardaba.
- **Precaución:** `npx -y` descarga y ejecuta código en mi equipo, por lo que solo conviene usarlo con paquetes de fuentes confiables, como este.

### Problema encontrado al configurarlo

Al conectar el servidor, `filesystem` apareció en rojo con el error `Cannot access directory ...\antigravity\$P`. Había quedado la variable `$P` sin sustituir en el archivo de configuración que lee la aplicación, porque el bloque se pegó a mano en lugar de ejecutarse en Git Bash. Se resolvió escribiendo la ruta real (`C:/Users/.../practica1-mcp`) en el archivo que lee Antigravity. Después, los tres servidores quedaron conectados.

### Capturas

**Los tres servidores conectados, con sus herramientas**
![Servidores conectados](evidencias/act_d_1.png)

**Sesión 1: `create_entities` (argumentos y aprobación)**
![create_entities 1](evidencias/act_d_2.png)
![create_entities 2](evidencias/act_d_3.png)
![create_entities 3](evidencias/act_d_4.png)
![create_entities 4](evidencias/act_d_5.png)
![create_entities 5](evidencias/act_d_6.png)
![create_entities 6](evidencias/act_d_7.png)
![Aprobación create_entities](evidencias/act_d_8.png)

**Sesión 1: `add_observations` (argumentos)**
![add_observations 1](evidencias/act_d_9.png)
![add_observations 2](evidencias/act_d_10.png)
![add_observations 3](evidencias/act_d_11.png)
![add_observations 4](evidencias/act_d_12.png)
![add_observations 5](evidencias/act_d_13.png)

**Sesión 1: `create_relations` (argumentos y aprobación)**
![create_relations 1](evidencias/act_d_14.png)
![create_relations 2](evidencias/act_d_15.png)
![create_relations 3](evidencias/act_d_16.png)
![create_relations 4](evidencias/act_d_17.png)
![Aprobación create_relations](evidencias/act_d_18.png)

**Sesión 1: `read_graph` (aprobación y resumen del agente)**
![Aprobación read_graph](evidencias/act_d_19.png)
![Resumen 1](evidencias/act_d_20.png)
![Resumen 2](evidencias/act_d_21.png)

**Sesión 2 (conversación nueva): `read_graph` y respuesta**
![Aprobación read_graph sesión 2](evidencias/act_d_22.png)
![Respuesta sesión 2](evidencias/act_d_23.png)

**Archivo de memoria:** [`salida/memoria.jsonl`](salida/memoria.jsonl)

## 6. Problemas encontrados

1. **Node.js y Claude Code no estaban instalados.** Al verificar las herramientas en Git Bash, `node`, `npx` y `claude` no se reconocían. Se instaló Node.js LTS desde nodejs.org, se reinició la terminal y se instaló Claude Code con npm (npm bloqueó su script de postinstalación y hubo que permitirlo con `--allow-scripts`).
2. **Claude Code requiere suscripción o crédito de API.** La pantalla de inicio de sesión solo ofrecía una cuenta con suscripción, una cuenta de Anthropic Console (facturación por uso) o una plataforma de terceros, y no contaba con ninguna. Se cambió a **Google Antigravity**, que la guía permite como alternativa equivalente con una cuenta personal de Google.
3. **Comandos de la guía en Git Bash.** Los comandos de la guía están escritos para PowerShell. En Git Bash, un argumento que empieza con `/` (como `/c`) se convierte en una ruta de Windows, y `$PWD` entrega una ruta de estilo Unix. Por eso la configuración se escribió directamente como JSON, usando `pwd -W` para obtener la ruta de Windows.
4. **Antigravity no mostraba los servidores.** Al principio la sección *MCP Servers* mostraba 0 servidores, porque la aplicación no leía el archivo `.agents/mcp_config.json` del proyecto. Se copió la configuración al archivo que usa la aplicación y se recargó, tras lo cual los servidores aparecieron conectados.
5. **`filesystem` en rojo con la ruta `$P`.** Al agregar el servidor `memory`, `filesystem` falló con el error `Cannot access directory ...\antigravity\$P`: la variable `$P` se había pegado sin sustituir en el archivo que lee la aplicación, porque el bloque se copió a mano en lugar de ejecutarse en Git Bash. Se arregló escribiendo la ruta real en ese archivo.
6. **Aviso de Chrome.** La ventana controlada por Playwright mostró «No se admite el indicador de línea de comandos (--disable-blink-features=AutomationControlled)...». Es normal con Playwright y no impidió la actividad.
7. **El agente hizo llamadas que no se le pidieron** (por ejemplo `list_allowed_directories`, listados de carpetas y lectura de las instrucciones). Todas eran de los servidores indicados y se aprobaron una por una; se documentan en cada actividad.
8. **La captura de pantalla de la Actividad B no apareció en `salida/`.** En la secuencia original, `browser_take_screenshot` recibió `salida/captura.png` como nombre de archivo y la herramienta no devolvió error visible en mis capturas, pero el archivo no estaba en la carpeta del proyecto: lo busqué en el proyecto, en la carpeta de `playwright` y en mi carpeta de usuario sin encontrarlo. No determiné la causa. Lo resolví repitiendo la captura en una consulta aparte, indicando la ruta de destino, y esta vez el archivo quedó en `salida/captura.png`. **No tomé capturas de pantalla de esa segunda consulta**, así que no queda registro de su instrucción ni de sus aprobaciones; solo existe el archivo resultante.
9. **Avisos de Git** sobre el cambio de saltos de línea (LF por CRLF) en Windows, sin efecto en la práctica. La rama inicial se llamaba `master` y se renombró a `main` al publicar.

**Qué verifiqué para estar seguro del resultado:**

- `reporte.md`: recalculé a mano cada promedio, los aprobados y el promedio del grupo (7.50).
- `entidades.csv`: comparé sus 32 filas con la tabla de Wikipedia y verifiqué tres filas a mano (Actividad B).
- Actividad C: confirmé en el mensaje de error que el rechazo vino del servidor `filesystem` y no del modelo.
- Actividad D: revisé `memoria.jsonl` y comprobé que no hay observaciones duplicadas.

## 7. Fuentes

- Wikipedia, «Anexo:Entidades federativas de México por superficie, población y densidad», origen de los datos de `salida/entidades.csv`. Se abrió con el enlace corto `https://es.wikipedia.org/?curid=5897912`.
- Guía de la práctica: *Práctica 1: Conectar servidores MCP a un agente*, Ingeniería en Sistemas Computacionales.
- Documentación del protocolo: https://modelcontextprotocol.io
- MCP en Google Antigravity: https://antigravity.google/docs/mcp
- MCP en Claude Code: https://code.claude.com/docs/en/mcp
- Servidor `filesystem` y servidor `memory`: https://github.com/modelcontextprotocol/servers
- Servidor `memory` en npm: https://www.npmjs.com/package/@modelcontextprotocol/server-memory
- Servidor `playwright`: https://github.com/microsoft/playwright-mcp
- Registro oficial de servidores MCP: https://registry.modelcontextprotocol.io
