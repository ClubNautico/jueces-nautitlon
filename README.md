Proyecto desactivado el 17/11/2026. Este repo solo guarda las instrucciones para reconstruirlo; el código y su historial se eliminaron a propósito.

# HANDOFF: Página "Jueces Nautitlón" (desactivada el 17/11/2026)

Documento para reconstruir la página desde cero en el futuro. Solo instrucciones y descripción: no hay código fuente, ni contraseñas, ni historial de cambios. Todo lo que aquí dice "la página" se refiere a la versión usada en la V Nautitlón 2026, Jornada 1 (jueves 5 de noviembre de 2026).

---

## 1. Qué era y para qué

Una página web pensada para celulares, usada junto a la piscina por los jueces/cronometristas durante una competencia. Cada juez abría la página, ponía su nombre y la contraseña de la competencia, y veía el programa de pruebas y series con una tarjeta por carril (nombre, edad, tiempos). Podía:

- escribir los tiempos de cada pase de 50 m y el tiempo final,
- o usar un cronómetro propio dentro de la página, con botón de pase y de parar,
- marcar NS (no se presentó) o DQ (descalificado) y dejar una nota.

Todo se guardaba en una hoja de Google Sheets (vía un Apps Script) para que varios celulares vieran lo mismo y para tener los tiempos juntos al final. Objetivo final del usuario (no logrado): pasar todos los tiempos de una sola vez a Team Manager 8 (TM8).

Fue la plataforma modelo de la que luego se copió la mecánica de la página de Tests de agua (mismo patrón: página estática en GitHub Pages + Apps Script + hoja + modo sin conexión).

## 2. Estado al desactivarla y qué se eliminó

Estado antes de apagarla:
- Se usó en la Jornada 1 (5/11). El 12/11 el usuario la dejó "en pausa" sin corregir los hallazgos de la auditoría (`_CLAUDE\AUDITORIA PAGINAS 2026-11-12`, sección 3, hallazgos J1 a J10). Los más importantes: repositorio público con nombre y edad de unos 60 menores en el código (J1), contraseña débil y sin límite de intentos (J2), el servidor aceptaba cualquier marca de tiempo (J3), una fila entera se reemplaza si dos jueces escriben el mismo carril (J6).
- Cambios del último día (17/11): (a) el logo de la barra superior lleva a la primera serie pendiente (ver sección 3), (b) la contraseña de la competencia pasó a guardarse solo en sessionStorage (ver sección 3 y 9). OJO: esos cambios existían solo en el `index.html` publicado/`sitio/`; la `plantilla.html` fuente NO los tenía (ver trampas).
- Dirección publicada: https://clubnautico.github.io/jueces-nautitlon/ (repo de GitHub ClubNautico/jueces-nautitlon, cuenta del club).

### Qué se eliminó el 17/11/2026 (lista A resumida)
Carpeta `COMPETENCIAS\NAUTITLON\Jueces web\` completa, con:
- `plantilla.html` (la fuente de la página), `plantilla_Codigo.gs` (fuente del Apps Script), `construir.py` (genera la página y el script), `sembrado.json` (programa de pruebas/series con nadadores), `Codigo.gs` (generado, llevaba la contraseña y el programa), `pin.txt` (contraseña; no se copia), `url_script.txt`, `prueba_api.py` (prueba manual del backend), `.claude\launch.json`.
- `sitio\` (el repositorio git publicado): `index.html`, `config.js`, `sw.js`, `robots.txt`, `README.md`, `.git`.
- `_respaldos_logo_2026-11-17\` (copia previa al cambio del logo).
- `RESPALDO hoja Tiempos V Nautitlon (12-11-2026).xlsx` (copia de la hoja con los tiempos reales: pestañas Tiempos, Historial, una prueba anterior y su historial): NO se borró; se movió a `COMPETENCIAS\NAUTITLON\RESPALDO hoja Tiempos V Nautitlon (12-11-2026).xlsx` para conservar los tiempos (p. ej. para TM8).
- Respaldos de seguridad `_respaldos_seguridad_2026-11-17\jueces_index.html` y `jueces_plantilla.html` en la raíz del club.
- Acceso directo `OneDrive\Desktop\Jueces Nautitlón (en pausa).lnk` (apuntaba a la carpeta Jueces web).
- Referencias editadas en el hub (tarjeta "Jueces Nautitlón" en `PAGINA INICIO\sitio\js\config.js` y en su `index.html` estático) y la memoria del proyecto.

Lo que NO se elimina con eso y sigue existiendo salvo que el usuario lo borre a mano:
- La hoja de Google Sheets en el Drive de la cuenta del club (cuenta y id: ver la copia privada del handoff en la carpeta `_CLAUDE` de REPORTES NADADORES), pestañas Tiempos e Historial (más "Prueba anterior" e "Historial anterior" de un formato viejo de 25 m).
- El proyecto de Apps Script asociado y su implementación como aplicación web (URL de la implementación: ver la copia privada del handoff en la carpeta `_CLAUDE` de REPORTES NADADORES). Mientras exista, quien tenga la contraseña puede leer y reescribir la hoja. Para apagarlo de verdad: en Apps Script, Implementar, Administrar implementaciones, archivar; y/o borrar el proyecto.
- El repositorio de GitHub ClubNautico/jueces-nautitlon sigue existiendo, pero el 17/11 se desactivó GitHub Pages y se reemplazó todo el contenido e historial por un único commit con solo un README (copia saneada de este handoff). Solo falta borrarlo del todo si el usuario quiere (Settings, Danger zone).
- Nota: las URL del Sheet y del Apps Script no son secretas; la contraseña sí lo era y no está en este documento.

Lo que NO es de la página y no se tocó: resultados, series, inscripciones, PDF, Excel y carga a TM8 del Nautitlón (siguen en `COMPETENCIAS\NAUTITLON\` y en `08_TEAM_MANAGER.md`).

## 3. Cómo funcionaba (pantallas y flujo de un juez)

Es una sola página (un solo archivo HTML con estilos y lógica incluidos, más un archivo de configuración y un service worker). Tiene tres pantallas:

1. Entrada. Título "V Nautitlón 2026", fecha y dos campos: "Tu nombre" y "Contraseña de la competencia" (con botón Mostrar/Ocultar contraseña). Al enviar, la página consulta al servidor con la contraseña para comprobarla ANTES de dejar entrar; si no hay red o la contraseña es mala, muestra el error y no entra. Es obligatoria: se decidió que sin contraseña válida no se entra.
2. Pantalla de trabajo. Barra superior fija con: logo, botón de programa (☰), título de la prueba (#número, sexo, categoría), subtítulo (distancia, estilo, grupo, "N pases de 50 m"), estado de sincronización, serie actual ("Serie 2 de 4"), hora de la serie y contador "x de y listos". Debajo, un filtro de carril (Todos, 1, 2, 3, 4), un aviso breve de cómo escribir, y las tarjetas de carril. Abajo, barra fija con "Anterior" y "Siguiente serie" (en la última serie dice "Última serie").
3. Programa. Panel que cubre la pantalla con todas las pruebas y, por cada una, botones de series con su hora. Colores: verde = serie completa, amarillo = faltan pases o hay que revisar, azul = serie en curso (la que se está viendo). Desde ahí se salta a cualquier serie. Al final, botón "Cambiar juez (nombre)" que cierra la sesión del juez.

Tarjeta de carril: número de carril, nombre completo, edad ("Menor de 9 años" cuando el dato es "< 9"), una etiqueta de estado (Listo, Revisar, NS, DQ), el cronómetro, una casilla por cada pase de 50 m, la casilla de "Tiempo final", el texto "Suma de pases", mensajes de aviso, nota opcional y botones NS, DQ y Nota. Al pie, firma: nombre del juez y hora de la última modificación. Los carriles sin nadador se muestran como "Carril N vacío".

Reglas de escritura de tiempos:
- Solo se escriben dígitos; la página da formato sola: 3245 es 32.45 y 10512 es 1:05.12.
- Cada casilla es el tiempo de ESE tramo de 50 m, no el acumulado. Al lado de cada pase se muestra el acumulado calculado (a partir del segundo pase).
- El tiempo final se llena solo con la suma de los pases cuando están todos; si el juez lo escribe a mano, queda como "final manual" y deja de recalcularse. Si faltan pases, basta con escribir solo el final.
- Si el final escrito no coincide con la suma de pases, la tarjeta pasa a "Revisar" y avisa la diferencia.
- Un tiempo a medias (menos de 3 dígitos, segundos mayores de 59, etc.) queda como "borrador" marcado en rojo, no se guarda ni se envía, y al cambiar de serie se avisa en qué carril quedó sin completar.
- Estilos de pase para pruebas combinadas: en 100 combinado, el primer tramo es Mariposa-Espalda y el segundo Pecho-Libre; en 200 combinado, los cuatro estilos en orden (Mariposa, Espalda, Pecho, Libre).
- Enter pasa a la siguiente casilla.

Estados de una serie (para colores del programa): una tarjeta está "lista" si tiene final y no hay diferencias ni pases faltantes, o si está marcada NS o DQ; "revisar" si hay final pero hay discrepancia o faltan pases; "parcial" si hay algún dato; la serie es verde si todas las tarjetas con nadador están listas, amarilla si el resto está listo o por revisar, azul/gris en proceso.

Cronómetro por carril (agregado después de la primera versión):
- Botón Iniciar; mientras corre aparece "Pase N m" (un toque por cada pase intermedio) y "Parar". Tras parar: "Continuar" y "Reiniciar".
- Si el carril ya tiene tiempos escritos, el primer toque en Iniciar pide un segundo toque de confirmación (avisa que borra los tiempos) y expira a los 3 segundos.
- Se ignoran toques demasiado seguidos (menos de 3 s entre pases, menos de 1 s para parar) con un aviso.
- La hora que vale es la del toque del dedo, no la del clic posterior. Se usan dos relojes (de alta precisión y de sistema) y, si discrepan más de medio segundo (el celular se durmió), manda el del sistema.
- Conversión a centésimas: se trunca el acumulado de cada pase y cada pase es la resta de acumulados, así los pases siempre suman exactamente el final.
- El estado del cronómetro se guarda localmente para sobrevivir recargas.

Navegación y arranque:
- Al entrar (o si la serie guardada ya no existe) la página abre en la PRIMERA SERIE PENDIENTE: la primera, en orden de programa, cuya serie no esté completa (verde); si todas están completas, la primera.
- Cambio del 17/11: el logo de la barra superior es un botón ("Ir al inicio de Jueces Nautitlón") que cierra el panel de programa si está abierto y lleva a la primera serie pendiente, sin borrar tiempos ni borradores ni cerrar la sesión del juez. Es un ícono SVG propio de 28 px en un botón de 44 px de área táctil, a la izquierda del botón de programa.
- Mantiene la pantalla encendida (Wake Lock) y la renueva al volver a la pestaña.
- Salir: "Cambiar juez" borra la contraseña y el nombre de la sesión y vuelve a la entrada. Los tiempos pendientes de enviar quedan guardados en el celular.

Seguridad en la sesión (cambio del 17/11): la contraseña de la competencia vive SOLO en sessionStorage (se pierde al cerrar el navegador). En localStorage se guarda la configuración (nombre del juez, carril, prueba y serie actual) sin la contraseña. Hay una migración: si una versión vieja la dejó en localStorage, se mueve a sessionStorage y se borra de la otra. Resultado práctico: al cerrar el navegador hay que volver a escribir la contraseña (los datos pendientes no se pierden).

## 4. Modelo de datos

### 4.1 En el celular (localStorage y sessionStorage)
- `jn_cfg_v1` (localStorage): configuración del juez: nombre, filtro de carril, índice de prueba y de serie. Sin contraseña.
- Clave de contraseña (sessionStorage, `jn_pin_v1`): solo mientras el navegador está abierto.
- `jn_dat_v2` (localStorage): los datos. Tiene dos partes: "filas" (mapa clave de carril a registro) y "pend" (lista de claves pendientes de enviar).
- `jn_cro_v1` (localStorage): estado de los cronómetros por clave de carril (inicio en dos relojes, marcas de pase, fin).
- Los borradores de texto a medio escribir no se guardan (solo en memoria).
- Se usa try/catch en toda lectura/escritura por si el navegador bloquea el almacenamiento.

Clave de carril: "prueba-serie-carril", por ejemplo 28-1-1 es la prueba 28, serie 1, carril 1.

Registro de un carril (en cliente y servidor): estado (vacío, NS o DQ), tiempos (por distancia 50, 100, 150, 200 y "final"), nota (máx. 120 caracteres), nombre del juez (máx. 40), marca de tiempo del cliente (ts, milisegundos) y bandera "auto" (el final proviene de la suma de pases o fue escrito a mano).

### 4.2 Programa sembrado (embebido en la página)
Lista de pruebas; cada prueba tiene: número, sexo (DAMAS/VARONES), categoría (texto como "≤10", "11-23", "24-70"), distancia (50, 100 o 200), estilo (PECHO, MARIPOSA, COMBINADO), grupo (CLASIFICADOS o MÁSTERS) y series; cada serie tiene número, hora (texto "18:30") y carriles (número, nombre completo y edad; nombre vacío = carril vacío; la edad "< 9" significa menor de 9). La versión de la Jornada 1 tenía 14 pruebas (números 18 a 31), 39 series y 145 nadadores-carril, de 18:30 a 19:55 aprox.; carriles 1 a 4.

### 4.3 Google Sheet (documento en el Drive de la cuenta del club; id: ver la copia privada del handoff en la carpeta `_CLAUDE` de REPORTES NADADORES)
Pestaña "Tiempos", una fila por carril, formato de texto en todas las celdas (para que Sheets no convierta tiempos en horas/fechas), primera fila y primera columna congeladas, columna final oculta. Columnas, en orden:
Clave, Prueba, Evento (texto armado: sexo, categoría, distancia, estilo, grupo), Serie, Hora, Carril, Nombre, Edad, Estado, Final, Suma pases, Origen final (vacío, "Suma de pases" o "Manual"), 50 m, 100 m, 150 m, 200 m, Nota, Juez, Actualizado (hora HH:mm:ss, zona America/Guayaquil), ts (marca de tiempo del cliente, oculta).
Una fila nace con los datos del programa (hasta Edad) y se llena al recibir tiempos. Una fila cuenta como "con datos" solo si su ts es mayor que cero.

Pestaña "Historial": bitácora de solo agregar, columnas Hora, Clave, Juez, Estado, Tiempos (texto JSON), Nota; una línea por cada fila guardada. El respaldo del 12/11 mostraba que llegó al tope de 1000 filas de lectura (el archivo de respaldo así lo muestra).

Si al arrancar encuentra una hoja "Tiempos" de formato anterior (sin la columna "Suma pases"), le cambia el nombre a "Prueba anterior HH-mm" (y el Historial igual) sin borrarlas y crea una nueva. Así quedó una prueba inicial con pases cada 25 m.

## 5. Backend (Apps Script): acciones y reglas

Es un script "ligado" a la cuenta del club, publicado como Aplicación web: se ejecuta como el dueño y con acceso "cualquier persona" (la protección es la contraseña, no la identidad de Google).

Contenido que lleva incrustado (lo inserta `construir.py` al generar el archivo, por eso el .gs generado nunca se publica en GitHub): la contraseña de la competencia, el id de la hoja, las filas del programa y la distancia de cada prueba.

Endpoints:
- GET: responde un JSON de salud ("ok" y nombre de servicio). Sirve para comprobar que está activo.
- POST: el cuerpo es un JSON enviado como texto plano (Content-Type text/plain; esto evita la petición previa CORS, que Apps Script no maneja). Campos: `accion`, `pin`, y `items` (si guarda).

Reglas del POST:
1. Si el JSON no se puede leer, responde error "formato".
2. Si la contraseña no coincide, responde error "pin" (se revisa antes de tomar el candado).
3. Toma un candado (LockService) de hasta 20 s para serializar escrituras; si no lo consigue, error "ocupado".
4. Acción "guardar": procesa hasta 200 items por llamada. Por cada uno: busca la fila por clave (si no existe la ignora); compara `ts` y si el del cliente es MENOR que el guardado, lo ignora ("gana el más reciente"); acepta estado solo NS o DQ; valida cada tiempo con una expresión estricta (minutos opcionales, segundos, centésimas) y descarta los que no coincidan; solo guarda pases cuya distancia no supere la de la prueba; recalcula "Suma pases" sumando centésimas; fija "Origen final" (vacío si no hay final; "Manual" si el cliente dijo auto falso; "Suma de pases" en otro caso); limpia Nota (120) y Juez (40) quitando caracteres iniciales de fórmula (=, +, -, @) y espacios, para evitar inyección de fórmulas; escribe toda la fila de una vez; agrega una línea al Historial con hora, clave, juez, estado, tiempos y nota.
5. Cualquier otra acción (en la práctica "leer"): no modifica nada.
6. En ambos casos responde `ok` verdadero y la lista de TODAS las filas con ts mayor que cero (clave, estado, tiempos por distancia y final, nota, juez, ts y bandera auto). Es decir, guardar también devuelve el estado completo, y así se sincronizan los celulares.
7. Una función de preparación (se corre una vez desde el editor) crea la hoja y autoriza permisos.

Cómo se probaba: un script de prueba hacía "leer", luego "guardar" una fila de ejemplo con valores manuales y luego la "limpiaba" con otra de ts mayor. También se usó un servidor falso local (mock) para probar la página sin tocar la hoja real.

Cómo se leía desde el cliente: cada 20 s (si la pestaña está visible y no hay envío en curso) y cuando la pestaña vuelve a primer plano; si hay pendientes, primero envía.

## 6. Sin conexión

- Todo lo escrito se guarda de inmediato en localStorage y la clave se agrega a la lista de pendientes; el envío se programa 0,7 s después de dejar de escribir (debounce).
- Envío: manda solo los pendientes; si el servidor responde bien, quita de pendientes las claves cuyo ts no cambió mientras iba el envío; luego integra lo que devuelve el servidor. Si falla la red o el servidor, queda "N sin enviar" y reintenta cada 8 s; si fue bien y aún quedan, reintenta a los 0,3 s. Tiempo máximo por llamada: 25 s.
- Al integrar el estado del servidor, no pisa: una fila con envío pendiente, una fila local con ts igual o más nuevo, ni una tarjeta donde el juez está escribiendo (foco) o hay borradores.
- Indicador de estado arriba: "Guardado" (verde), "Enviando…" (azul), "N sin enviar" / "Sin señal" / "Sin conectar" (ámbar), "PIN incorrecto" (rojo).
- Si el servidor responde "pin" (cambió la contraseña), se cierra la sesión y se pide de nuevo, sin perder lo pendiente.
- El evento "online" del navegador dispara el envío inmediato.
- Service worker (sw.js): estrategia "red primero, copia local si falla" solo para GET del mismo origen; guarda cada respuesta en caché; permite recargar la página sin señal. Solo se registra en https. Los pedidos al Apps Script no pasan por él.
- Para que un cambio de `config.js` no se quede en caché, se sube el número `?v=` del tag que lo carga en la plantilla (estaba en 3).
- Limitación conocida: sin señal se puede seguir trabajando, pero para ENTRAR la primera vez se necesita conexión (se comprueba la contraseña contra el servidor).

## 7. Diseño

- Estilo "mesa de control junto a la piscina": una tarjeta por carril, cifras grandes, alto contraste, mucha área táctil (botones de 36 a 44 px o más), sin adornos.
- Colores del club: azul 22409A como color de marca, azul oscuro 1C3480 para la barra superior (hero), ámbar FFC93C para acentos (hora, filtro activo, foco), fondo F4F6FB, tarjetas blancas, texto 1E2430. Verde 14703F / fondo DDF3E5 para OK, ámbar oscuro 8A5A00 / fondo FFF1CC para revisar, rojo B3261E / fondo FCE3E1 para error.
- Modo oscuro automático (prefers-color-scheme): fondo 0E1424, tarjeta 172038, texto E8ECF5, y versiones claras de los colores semánticos.
- Tipografías: Josefin Slab 700 para títulos y Open Sans (400, 600, 700, 800) para texto, desde Google Fonts. Cifras con números tabulares.
- Ancho máximo 860 px centrado; respeta las zonas seguras del iPhone (notch y barra inferior); barra superior y barra de navegación fijas; foco visible con contorno ámbar; etiquetas ARIA en estado y grupos de botones.
- No indexable: meta robots noindex y un robots.txt (que en realidad no sirve porque los buscadores solo leen el de la raíz del dominio).
- Todo texto de datos se escapa antes de insertarlo en el HTML.
- Pensado para el wifi/datos móviles de la piscina; los jueces usaban sus propios celulares.

## 8. Pasos para reconstruirla desde cero

Antes: definir si la competencia nueva necesita pases cada 50 m y cuáles son sus pruebas; revisar los hallazgos de la sección 9 para no repetirlos.

1. Preparar el programa: sacar de la hoja de series oficial (en la V Nautitlón fue el PDF de series de la jornada, `Series_V_Nautitlon_JORNADA 1.pdf`, que sigue en `COMPETENCIAS\NAUTITLON\`) la lista de pruebas con número, sexo, categoría, distancia, estilo, grupo, series con hora y carriles con nombre y edad. Guardarlo como un archivo de datos con la estructura de la sección 4.2. Corregir a mano los nombres que el PDF corta (en la V Nautitlón se cortaban a 32 o 34 caracteres y se arreglaban con una tabla de sustituciones). Vaciar nombres de carriles sin nadador. Verificar contra TM8/inscripciones y excluir a quienes se retiraron.
2. Decidir privacidad ANTES de publicar: no meter nombre completo ni edad exacta de menores en una página pública. Opciones: repositorio privado (Pages privado requiere plan de pago) o, mejor, que el programa llegue del servidor DESPUÉS de validar la contraseña (como hace la página de Tests de agua) y sin edad (o solo categoría).
3. Crear una hoja nueva de Google Sheets en el Drive del club y anotar su id.
4. Crear un proyecto de Apps Script y pegar el script generado (ver sección 5). Elegir una contraseña aleatoria de 8 o más caracteres (letras y números), distinta por jornada; no reusar la anterior. Ejecutar una vez la función de preparación y autorizar permisos.
5. Implementar como Aplicación web (ejecutar como el dueño, acceso cualquier persona). Copiar la URL de implementación (termina en /exec). Cada cambio de código exige crear una nueva versión de la implementación (no basta guardar).
6. Probar con el script de prueba (leer, guardar con valores de prueba, limpiar) y con un servidor falso local antes de usar la hoja real.
7. Construir la página: en una carpeta de trabajo (mejor FUERA de Dropbox, ver trampas) tener una plantilla de la página con un marcador para el programa y una plantilla del script con marcadores para contraseña, filas del programa y distancias por prueba. Un script de construcción (construir.py) lee el programa, genera las filas planas (clave, prueba, nombre del evento, serie, hora, carril, nombre, edad), y escribe el HTML final y el script final. Si no se le da contraseña, genera una aleatoria y la guarda aparte (en un archivo que nunca se publica).
8. Escribir config.js con la URL del Apps Script (un solo campo). La plantilla debe tolerar su ausencia (sin URL, mostrar "Sin conectar", pero NO borrar los pendientes; ese fue un hallazgo de la auditoría).
9. Implementar las pantallas y reglas de las secciones 3 a 7 (entrada con comprobación de contraseña, trabajo, programa; formateo de dígitos; suma de pases; estados; cronómetro; cola sin conexión; service worker).
10. Publicar: crear un repositorio nuevo en la cuenta del club (ClubNautico), subir solo la carpeta `sitio` (index.html, config.js, sw.js, robots.txt, README); NUNCA subir el script generado ni el archivo de contraseña. Activar GitHub Pages desde la rama principal. La dirección queda https://clubnautico.github.io/NOMBRE-DEL-REPO/. Para actualizar: editar plantillas, correr construir, copiar a `sitio` y hacer push; si cambió config.js, subir su `?v=`.
11. Reponerla en el hub: en `PAGINA INICIO\sitio\js\config.js`, en la sección "Para profesores", agregar una tarjeta con título "Jueces Nautitlón" (o el nombre nuevo), descripción "Tiempos y pases de la competencia.", ícono "bandera" y la URL publicada. Además, el hub tiene una copia estática de las tarjetas en `PAGINA INICIO\sitio\index.html` (lista de tarjetas con el mismo texto, enlace y descripción): agregar el mismo ítem ahí, para que aparezca aunque no cargue el JS. Luego publicar el hub (según `PAGINA INICIO\COMO USAR.md`).
12. Prueba de campo: abrir con 2 celulares, entrar con nombres distintos, escribir en el mismo carril y en distintos, desconectar uno, reconectar, y confirmar que la hoja y los celulares coinciden. Probar iPhone (Safari) y Android (Chrome).
13. Después de la competencia: bajar un respaldo de la hoja (Excel), cambiar la contraseña o archivar la implementación, y vaciar o despublicar la página.

Reparto de trabajo que funcionó: subagentes Sonnet para escribir/editar el código y subagentes Haiku para pruebas y revisión de resultados.

## 9. Lecciones y trampas

- Editar JavaScript desde PowerShell se come los `$` (los interpreta como variables de PowerShell). Para editar código usar la herramienta de edición de archivos o scripts Python; si hay que usar PowerShell, comillas simples y here-strings literales.
- No guardar la contraseña de la competencia en localStorage (queda en claro en celulares prestados y no caduca). Usar sessionStorage y pedirla de nuevo. Ya está hecho así en la última versión; migrar cualquier copia vieja.
- La fuente se desincronizó: los dos últimos cambios (logo y contraseña) se hicieron en la página publicada, no en la plantilla fuente. Al reconstruir, edita SIEMPRE la plantilla y regenera; nunca edites el archivo generado. Después de cada cambio, comparar la plantilla con lo publicado.
- La contraseña estaba en el archivo del script generado y en un archivo de texto; no estaban en git, bien, pero si el repo se publica el programa (nombres y edades de menores) sí queda en todo el historial. Un repo público con datos de menores es el mayor riesgo; evitarlo desde el diseño.
- Servidor: limitar la marca de tiempo del cliente (por ejemplo a hora actual + 2 minutos); si no, un ts enorme bloquea una fila para siempre. Poner try/catch en el POST (si lanza una excepción, Apps Script devuelve HTML y el cliente reintenta cada 8 s indefinidamente). Poner freno de intentos de contraseña.
- "Gana el último" se decide por fila completa: dos jueces en el mismo carril se pisan (por ejemplo, si uno solo agrega la nota se pierden los pases del otro). Mezclar por campo o asignar un juez por carril.
- El cliente borraba los pendientes si config.js no cargaba; no debe hacerlo.
- En iOS los botones no reciben foco al tocarse, y la protección "no pisar a quien escribe" se basa en el foco; también habría que saltar tarjetas con el cronómetro corriendo. `performance.timeOrigin` no existe en iOS 14 o anteriores. Un único "último toque" se cruza si se tocan dos cronómetros a la vez con el filtro "Todos".
- Apps Script no maneja la petición CORS previa: enviar como text/plain y seguir redirecciones.
- Hoja de Sheets: forzar formato texto en todas las celdas o los tiempos se convierten en horas. La lectura en bloque topa con el límite de filas del Historial (1000 filas leídas en el respaldo); si crece, paginar.
- Cambiar el código del script exige crear una nueva versión de la implementación (no basta guardar).
- El robots.txt de subcarpetas de GitHub Pages no sirve; para no ser indexada, la noindex del HTML ayuda, pero lo único seguro es no publicar datos personales.
- Puertos locales: las 3 páginas (récords, entrenamientos, jueces) usaban el mismo 8765; usar un puerto distinto por página (el hub usa el 8760, asistencias 8772, etc.).
- El lanzador de pruebas (`launch.json`) apuntaba a un servidor falso guardado en la carpeta temporal de otra sesión; si se necesita, guardar el mock dentro de la carpeta del proyecto.
- Los repos .git dentro de Dropbox pueden dañarse por "copias en conflicto"; clonarlos fuera de Dropbox.
- Los datos de la competencia cambian: retiros y NS (tres nadadores salieron del meet el 6/11 y la página aún los tenía). Reconciliar el programa con TM8 antes de cada jornada.
- Comprobar la contraseña en el servidor antes de entrar evitó gente "dentro" sin permiso; mantenerlo.

## 10. Pendientes que quedaron

1. Exportador a TM8 (el objetivo final del usuario): un script local (nunca en la nube) que lea la hoja, cruce cada nadador con la base de TM8 y genere un archivo para File, Import, Meet Results de TM8. Debe correr solo en una COPIA de la base; nunca escribir directo en el .mdb, que además queda bloqueado con TM8 abierto. El cruce por nombre es local porque las fechas de nacimiento y los ID no deben ir a la página ni a la hoja. Ver `08_TEAM_MANAGER.md` y `nautitlon-2026-tm8` (memoria) antes de importar. Nunca se hizo.
2. Hallazgos de la auditoría 12/11 sin corregir (J1 a J10, sección 3 de `AUDITORIA PAGINAS 2026-11-12\1 Seguridad y funcion.md`). Los cuatro más graves están resumidos arriba.
3. Resultados de la Jornada 1 en los tiempos de la hoja: se trabajó con tiempos de TM8 (PDF del 5/11), no con esta hoja. El respaldo de la hoja del 12/11 (conservado en `COMPETENCIAS\NAUTITLON\RESPALDO hoja Tiempos V Nautitlon (12-11-2026).xlsx`) es la única copia de lo que los jueces anotaron en la página, además de la hoja de Google si no se borra.
4. Para una versión futura: programa servido desde el servidor tras la contraseña, sin edades, contraseña por jornada, acción "cerrar jornada", mezcla por campo, y un modo "solo lectura" para público/delegados.
5. Pendiente del usuario: decidir si borra la hoja de Google, el proyecto de Apps Script y el repositorio de GitHub (siguen existiendo hasta que él lo haga).
