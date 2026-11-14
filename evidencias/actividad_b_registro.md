## Actividad B: de una página web a un CSV

**Servidores usados:** `playwright` (navegador) y `filesystem` (guardar el archivo).
**Agente:** Google Antigravity App, modelo Gemini 3.8 Flash. Todas las llamadas pidieron aprobación ("Allow using this MCP tool?") y se aprobaron con *Yes, allow this time*.

### Instrucción dada al agente

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

### Secuencia de herramientas

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
| 13 | playwright | `browser_take_screenshot` | Guardó `salida/captura.png` (`fullPage: false`, `scale: css`) | `act_b_21` |
| 14 | playwright | `browser_close` | Cerró el navegador | `act_b_22` |
| 15 | filesystem | `read_file` | Leyó `salida/entidades.csv` | `act_b_23` |

### En qué paso cambió de servidor

- Empezó con una llamada breve a `filesystem` (paso 1) para comprobar su carpeta permitida.
- El cambio principal fue del **paso 10 al 11**: tras terminar de leer y limpiar la tabla con `playwright`, pasó a `filesystem` (`create_directory`, y después `write_file`) para guardar el CSV. `playwright` solo ve la página, y `filesystem` es el único que puede escribir en la carpeta de la práctica.
- Después volvió a `playwright` para la captura de pantalla y para cerrar el navegador (pasos 13 y 14), y terminó con `filesystem` para leer el CSV (paso 15).

### Cómo obtuvo los datos de la tabla

**Ejecutando JavaScript** con `browser_evaluate`. No usó `browser_snapshot`. El agente fue avanzando por tanteo:

1. Buscó las tablas con clase `wikitable` y luego listó todas las tablas de la página.
2. Eligió la primera tabla (`table[0]`) y examinó sus filas (texto, HTML, `colspan` y `rowspan`), tanto las primeras como las últimas.
3. Con esa estructura, tomó por posición las columnas de **entidad** (3), **superficie** (4), **población** (6) y **densidad** (7), y saltó la 5, que es el porcentaje de superficie, y las tres columnas de posición.
4. Limpió los números dentro del mismo script: quitó espacios (normales y no separables) y cambió la coma decimal por punto.

### Resultado: `salida/entidades.csv`

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

### Verificación manual contra la página

Comparé el CSV con la tabla de Wikipedia (capturas `act_b_25` y `act_b_26`). Revisé tres filas, elegidas por ser las que más fácil fallan (números largos, cifras de cuatro dígitos sin separador y una fila con una capital «N/A»):

| Entidad | En el CSV (superficie, población, densidad) | En la página | ¿Coincide? |
|---|---|---|---|
| Estado de México | 22351, 16992418, 724 | 22 351, 16 992 418, 724 | Sí |
| Colima | 5627, 731391, 126 | 5627, 731 391, 126 | Sí |
| Ciudad de México | 1495, 9209944, 5966 | 1495, 9 209 944, 5966 | Sí |

### Observaciones

- **Aviso de Chrome:** al abrirse la ventana controlada por el agente apareció la barra «No se admite el indicador de línea de comandos (--disable-blink-features=AutomationControlled)...». Es normal cuando Playwright controla Chrome y no impide el funcionamiento.
- **Captura de pantalla:** el servidor solo capturó la parte visible de la página (`fullPage: false`), por lo que no incluye toda la tabla. Está en `salida/captura.png`.
- **Llamadas extra:** el agente hizo exploraciones y comprobaciones que la instrucción no pedía (`list_allowed_directories`, lectura de notas al pie, `create_directory` y `read_file` del CSV). Todas eran de `playwright` o `filesystem`.

### Capturas

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

**Captura tomada por `playwright`**
![Captura de playwright](salida/captura.png)
