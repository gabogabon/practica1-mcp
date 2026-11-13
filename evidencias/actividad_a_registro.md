## Actividad A: lectura y escritura de archivos

**Servidor usado:** `filesystem` (todas las llamadas fueron a herramientas de este servidor).
**Agente:** Google Antigravity App, modelo Gemini 3.8 Flash. Aprobación de herramientas: cada llamada MCP pidió confirmación ("Allow using this MCP tool?") y se aprobó con la opción 1, *Yes, allow this time*.

### Instrucción dada al agente

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

### Secuencia de herramientas

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

### Observaciones

- **Llamadas adicionales.** El agente hizo por su cuenta llamadas que la instrucción no pedía: `list_allowed_directories` al inicio (comprobar qué carpetas puede ver el servidor), y listados de la raíz del proyecto, `instrucciones` y `salida`, además de leer `instrucciones/actividad_a.txt`. Todas son de `filesystem` y de solo lectura, salvo la escritura final, que fue la pedida.
- **Aprobaciones.** La aplicación se detuvo con *Waiting for user input* antes de cada llamada. Se eligió siempre *Yes, allow this time*, para ver y aprobar cada una por separado.
- **Duración.** El resumen del agente indica "Worked for 40m"; ese tiempo incluye las esperas de aprobación.
- **Resumen del agente.** Dijo haber listado `datos`, leído el CSV con `read_text_file`, calculado los promedios (media de los tres parciales, dos decimales, aprueba con >= 6.0, promedio del grupo 7.50) y escrito el reporte con `write_file`.

### Resultado: `salida/reporte.md`

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

### Capturas

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
