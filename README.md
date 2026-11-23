# 🧬 Trabajo Práctico Integrador: Juego de la Vida de Conway

**Materia:** Paradigmas de la Programación  
**Grupo:** N.º 19  
**Tecnología:** Pharo 14 (Smalltalk) + Morphic  

### 👥 Integrantes del Grupo 19
* Fedorchuk, Francisco.
* Fernandez, Santiago.
* Ludemann, Facundo.
* Viola, Yara Yani.
* Ortiz, Benito de Jesús.
* Tomsich, Walter Daniel.

---

## 📋 Tabla de Contenidos
1. [Descripción del Proyecto y Funcionalidades](#-descripción-del-proyecto-y-funcionalidades)
2. [Guía Paso a Paso: Cómo cargar las clases y ejecutarlo en Pharo 14](#-guía-paso-a-paso-cómo-cargar-las-clases-y-ejecutarlo-en-pharo-14)
3. [Arquitectura Orientada a Objetos](#-arquitectura-orientada-a-objetos)
4. [Documentación Técnica y Código Fuente por Clase](#-documentación-técnica-y-código-fuente-por-clase)
   * [Clase `Automata`](#1-clase-automata)
   * [Clase `Tablero`](#2-clase-tablero)
   * [Clase `TableroVista`](#3-clase-tablerovista)
5. [Ejecución desde el Playground](#-ejecución-desde-el-playground)

---

## 🧩 Descripción del Proyecto y Funcionalidades

Este proyecto implementa el **Juego de la Vida de John Conway** en **Pharo 14**, utilizando Programación Orientada a Objetos pura y el framework gráfico **Morphic**.

### Reglas de Transición
Cada celda de la grilla bidimensional es un autómata finito con dos estados posibles: `#vivo` o `#muerto`. En cada generación ($t \rightarrow t+1$), todos los autómatas actualizan su estado de manera **simultánea** evaluando sus 8 vecinos circundantes (**vecindad de Moore**):
* **Supervivencia:** Una célula `#vivo` con **2 o 3 vecinos vivos** permanece `#vivo` en la siguiente generación.
* **Nacimiento:** Una célula `#muerto` con exactamente **3 vecinos vivos** pasa a `#vivo` en la siguiente generación.
* **Muerte (soledad o sobrepoblación):** En cualquier otro caso (`< 2` o `> 3` vecinos vivos), la célula pasa a `#muerto` (o permanece muerta).

### Características y Extensiones Implementadas
* **Interfaz gráfica interactiva (`Morphic`):** Permite hacer clic sobre cualquier celda para alternar su estado (`#vivo` / `#muerto`).
* **Simulación paso a paso y continua:** Botón **`Siguiente Gen`** para avanzar de a una generación y **`Play / Pausa`** para ejecutar la evolución automática cada `300 ms`.
* **Generación aleatoria (`Aleatorio`):** Puebla el tablero con una distribución aleatoria de autómatas vivos y muertos (densidad del 35%) lista para simular.
* **Borde Toroidal activable (`Borde Toroidal`):** Permite alternar dinámicamente entre un tablero finito (con bordes) y un tablero toroidal (donde el borde derecho conecta con el izquierdo y el superior con el inferior).
* **Dimensiones parametrizables (`Cambiar Tamano`) y limpieza (`Limpiar`):** Permite redimensionar filas y columnas en tiempo de ejecución o reiniciar la grilla.
* **Barra de estado en tiempo real:** Muestra el número de generación actual, la población viva y si el borde toroidal está `Activado` o `Desactivado`.

---

## 🚀 Guía Paso a Paso: Cómo cargar las clases y ejecutarlo en Pharo 14

Sigue estos pasos dentro de una imagen de **Pharo 14** (abierta desde *Pharo Launcher*):

### Paso 1: Abrir el Navegador de Clases (System Browser)
1. Haz clic izquierdo en cualquier parte libre del fondo de Pharo para abrir el **World Menu**.
2. Selecciona **Browse** (o presiona el atajo `Ctrl + O` seguido de `B`). Se abrirá la ventana **System Browser (Calypso)**.

### Paso 2: Crear el Paquete `'JuegoDeLaVidaDeConway'`
1. En el panel izquierdo del *System Browser* (lista de paquetes), haz clic derecho y selecciona **New package** (o presiona `Ctrl + N` sobre ese panel).
2. Escribe exactamente el nombre:
   ```text
   JuegoDeLaVidaDeConway
   ```
3. Presiona **Enter / OK**.

### Paso 3: Crear las 3 Clases del Proyecto
Con el paquete `JuegoDeLaVidaDeConway` seleccionado, en el panel inferior de edición de código reemplaza la plantilla por la definición de cada clase y guárdala presionando **`Ctrl + S`** (o clic derecho $\rightarrow$ **Accept**). Repite esto para las tres clases:

1. **Crear `Automata`:**
   ```smalltalk
   Object << #Automata
   	slots: { #estado };
   	package: 'JuegoDeLaVidaDeConway'
   ```
   *(Presiona `Ctrl + S`)*

2. **Crear `Tablero`:**
   ```smalltalk
   Object << #Tablero
   	slots: { #filas . #columnas . #casillas . #toroidal };
   	package: 'JuegoDeLaVidaDeConway'
   ```
   *(Presiona `Ctrl + S`)*

3. **Crear `TableroVista`:**
   ```smalltalk
   Morph << #TableroVista
   	slots: { #tablero . #tamanoCelda . #corriendo . #generacion . #etiquetaInfo };
   	package: 'JuegoDeLaVidaDeConway'
   ```
   *(Presiona `Ctrl + S`)*

### Paso 4: Agregar los Métodos a Cada Clase
> **Importante:** Todos los métodos de este proyecto son **métodos de instancia** (asegúrate de que en el *System Browser* esté seleccionada la pestaña **Instance side** y **no** *Class side*).

Para cada clase (`Automata`, `Tablero` y `TableroVista`):
1. Selecciona la clase en el segundo panel del *System Browser*.
2. Haz clic en el panel de métodos (o selecciona un protocolo como `all` / `as yet unclassified`).
3. En el editor inferior, borra la plantilla de método, pega el código de uno de los métodos listados en la [sección de código fuente](#-documentación-técnica-y-código-fuente-por-clase) y presiona **`Ctrl + S`** (**Accept**) para compilarlo.
4. Repite el proceso hasta cargar todos los métodos de las 3 clases. *(Sugerencia: carga primero los métodos de `Automata`, luego los de `Tablero` y por último los de `TableroVista`)*.

### Paso 5: Abrir el Playground y Ejecutar la Aplicación
1. Abre el **World Menu** (clic izquierdo en el fondo) y selecciona **Playground** (o presiona `Ctrl + O` seguido de `W`).
2. Escribe la siguiente expresión:
   ```smalltalk
   TableroVista new abrirVentana.
   ```
3. Selecciona la línea y presiona **`Ctrl + D`** (o clic derecho $\rightarrow$ **Do it**).
4. Se abrirá la ventana interactiva **`Juego de la Vida de Conway`**. Ya puedes dibujar células con clic o presionar **`Aleatorio`**, activar **`Borde Toroidal`** y pulsar **`Play / Pausa`**.

*(Opcional)* Si deseas guardar tu trabajo dentro de tu imagen de Pharo antes de cerrar, abre el **World Menu** y selecciona **Save**.

---

## 🏗️ Arquitectura Orientada a Objetos

El diseño divide las responsabilidades en **Modelo** y **Vista**:

```text
+--------------------------------------------------+
|               CAPA DE VISTA (Morphic)            |
|  TableroVista (hereda de Morph)                  |
|  - Dibuja la grilla en Canvas (drawOn:)          |
|  - Gestiona botones, clics y ciclo step          |
+--------------------------------------------------+
                         |
                         | delega lógica y estado
                         v
+--------------------------------------------------+
|              CAPA DE MODELO (Dominio)            |
|  Tablero (hereda de Object)                      |
|  - Matriz 2D de casillas (Array de Arrays)       |
|  - Vecindad de Moore (finita o toroidal)         |
|  - Evolución simultánea y población              |
+--------------------------------------------------+
                         |
                         | contiene (filas x columnas)
                         v
+--------------------------------------------------+
|  Automata (hereda de Object)                     |
|  - Estado individual (#vivo / #muerto)           |
|  - Reglas de transición de Conway                |
+--------------------------------------------------+
```

---

## 💻 Documentación Técnica y Código Fuente por Clase

### 1. Clase `Automata`

La clase `Automata` representa una célula individual dentro del tablero del Juego de la Vida de Conway, hereda de `Object` y pertenece al paquete `JuegoDeLaVidaDeConway`. Tiene la variable de instancia `estado` (almacenada como slot) para registrar si la célula está viva (`#vivo`) o muerta (`#muerto`), y se encarga de informar su estado, cambiarlo o alternarlo y calcular cuál será su estado en la siguiente generación según la cantidad de vecinos vivos.

#### Definición de la clase
```smalltalk
Object << #Automata
	slots: { #estado };
	package: 'JuegoDeLaVidaDeConway'
```

#### Métodos de instancia

* **`estaVivo`**: No recibe parámetros y devuelve un booleano indicando si el autómata está vivo actualmente. Evalúa si la variable de instancia `estado` coincide exactamente con el símbolo `#vivo` (`estado == #vivo`), retornando `true` cuando lo está y `false` en caso contrario.
```smalltalk
estaVivo
	^ estado == #vivo
```

* **`aplicarEstado:`**: Recibe un parámetro llamado `nuevoEstado` (esperado como `#vivo` o `#muerto`) y asigna ese valor directamente a la variable de instancia `estado`. Se utiliza tanto para establecer el estado inicial de una celda como para aplicar los estados calculados al cerrar una generación.
```smalltalk
aplicarEstado: nuevoEstado
	estado := nuevoEstado
```

* **`alternarEstado`**: No recibe parámetros e invierte el estado actual del autómata. Consulta si la célula está viva mediante `self estaVivo`: si es verdadero le aplica el estado `#muerto`, y si es falso le aplica el estado `#vivo`, permitiendo prender o apagar celdas de forma directa, por ejemplo al hacer clic en la interfaz.
```smalltalk
alternarEstado
	self estaVivo
		ifTrue: [ self aplicarEstado: #muerto ]
		ifFalse: [ self aplicarEstado: #vivo ]
```

* **`calcularProximoEstadoConVecinos:`**: Recibe la cantidad de vecinos vivos (`cantidadVecinos`) y devuelve el símbolo (`#vivo` o `#muerto`) que corresponderá en la próxima generación sin modificar el estado actual del autómata. Si la célula está viva, devuelve `#vivo` solo cuando tiene 2 o 3 vecinos vivos y `#muerto` en cualquier otro caso; si está muerta, devuelve `#vivo` únicamente cuando tiene exactamente 3 vecinos vivos y `#muerto` en caso contrario.
```smalltalk
calcularProximoEstadoConVecinos: cantidadVecinos
	(self estaVivo)
		ifTrue: [ ^ (cantidadVecinos = 2 or: [ cantidadVecinos = 3 ])
				ifTrue: [ #vivo ]
				ifFalse: [ #muerto ] ]
		ifFalse: [ ^ (cantidadVecinos = 3)
				ifTrue: [ #vivo ]
				ifFalse: [ #muerto ] ]
```

---

### 2. Clase `Tablero`

La clase `Tablero` representa la grilla bidimensional donde se alojan y evolucionan los autómatas del Juego de la Vida de Conway, hereda de `Object` y pertenece al paquete `JuegoDeLaVidaDeConway`. Tiene las variables de instancia `filas` (largo), `columnas` (ancho), `casillas` (matriz de arrays anidados con instancias de `Automata`) y el atributo booleano `toroidal` (que indica si los bordes de la grilla están conectados entre extremos opuestos o si son finitos). Se encarga de crear o redimensionar el tablero, poblarlo aleatoriamente, acceder a sus celdas, contar vecinos vivos (en modo plano o toroidal), calcular y mostrar la población y avanzar las generaciones de forma simultánea.

#### Definición de la clase
```smalltalk
Object << #Tablero
	slots: { #filas . #columnas . #casillas . #toroidal };
	package: 'JuegoDeLaVidaDeConway'
```

#### Métodos de instancia

* **`aleatorizar`**: No recibe parámetros y puebla el tablero con una distribución aleatoria de autómatas vivos y muertos utilizando una densidad por defecto del 35% (`0.35`). Delega la operación en `self aleatorizarDensidad: 0.35`.
```smalltalk
aleatorizar
	self aleatorizarDensidad: 0.35
```

* **`aleatorizarDensidad:`**: Recibe un número decimal llamado `probabilidadVivo` (entre `0.0` y `1.0`) y recorre todas las filas y columnas del tablero asignando a cada autómata el estado `#vivo` si un número aleatorio generado con `Random new next` es menor que `probabilidadVivo`, o `#muerto` en caso contrario, mediante `aplicarEstado:`.
```smalltalk
aleatorizarDensidad: probabilidadVivo
	| random |
	random := Random new.
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			| estadoElegido |
			estadoElegido := (random next < probabilidadVivo)
				ifTrue: [ #vivo ]
				ifFalse: [ #muerto ].
			(self casillaEnFila: i columna: j) aplicarEstado: estadoElegido ] ]
```

* **`alternarEnFila:columna:`**: Recibe dos parámetros numéricos (`fila` y `columna`). Obtiene el autómata ubicado en esa posición mediante `self casillaEnFila: fila columna: columna` y le envía el mensaje `alternarEstado` para cambiarlo entre vivo y muerto.
```smalltalk
alternarEnFila: fila columna: columna
	(self casillaEnFila: fila columna: columna) alternarEstado
```

* **`alternarToroidal`**: No recibe parámetros e invierte el estado booleano de la variable de instancia `toroidal` (si estaba en `false` pasa a `true` y viceversa), devolviendo el nuevo valor resultante para activar o desactivar el comportamiento de bordes conectados.
```smalltalk
alternarToroidal
	toroidal := toroidal not.
	^ toroidal
```

* **`avanzarGeneracion`**: No recibe parámetros y hace evolucionar todo el tablero a la siguiente generación respetando la evaluación simultánea de las reglas de Conway. Primero recorre todas las filas y columnas para construir una estructura temporal (`proximosEstados`) donde guarda el próximo estado de cada autómata según su cantidad de vecinos vivos (`contarVecinosFila:columna:` y `calcularProximoEstadoConVecinos:`), y luego recorre nuevamente la grilla para aplicar todos esos estados resultantes sobre cada casilla mediante `aplicarEstado:`.
```smalltalk
avanzarGeneracion
	| proximosEstados |
	proximosEstados := (1 to: filas) collect: [ :i |
		(1 to: columnas) collect: [ :j |
			| automata vecinos |
			automata := self casillaEnFila: i columna: j.
			vecinos := self contarVecinosFila: i columna: j.
			automata calcularProximoEstadoConVecinos: vecinos ] ].

	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			(self casillaEnFila: i columna: j)
				aplicarEstado: ((proximosEstados at: i) at: j) ] ]
```

* **`calcularPoblacion`**: No recibe parámetros y devuelve un número entero con la cantidad total de autómatas vivos en el tablero. Inicializa un contador (`total`) en `0`, recorre todas las filas de `1` a `filas` y todas las columnas de `1` a `columnas`, y suma `1` por cada casilla cuyo método `estaVivo` devuelva verdadero.
```smalltalk
calcularPoblacion
	| total |
	total := 0.
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			((self casillaEnFila: i columna: j) estaVivo)
				ifTrue: [ total := total + 1 ] ] ].
	^ total
```

* **`cambiarAncho:`**: Recibe un parámetro llamado `nuevoAncho` y modifica el ancho del tablero manteniendo el largo actual. Internamente invoca a `self configurarFilas: filas columnas: nuevoAncho` para regenerar la matriz de casillas.
```smalltalk
cambiarAncho: nuevoAncho
	self configurarFilas: filas columnas: nuevoAncho
```

* **`cambiarLargo:`**: Recibe un parámetro llamado `nuevoLargo` y modifica el largo del tablero manteniendo el ancho actual. Internamente invoca a `self configurarFilas: nuevoLargo columnas: columnas` para regenerar la matriz de casillas.
```smalltalk
cambiarLargo: nuevoLargo
	self configurarFilas: nuevoLargo columnas: columnas
```

* **`casillaEnFila:columna:`**: Recibe dos parámetros (`fila` y `columna`) y devuelve el objeto `Automata` almacenado en esa coordenada del tablero. Accede primero a la fila correspondiente en `casillas` (`(casillas at: fila)`) y luego a la posición de la columna (`at: columna`).
```smalltalk
casillaEnFila: fila columna: columna
	^ (casillas at: fila) at: columna
```

* **`columnas`**: Devuelve el valor almacenado en la variable de instancia `columnas`, indicando la cantidad actual de columnas (ancho) del tablero.
```smalltalk
columnas
	^ columnas
```

* **`configurarFilas:columnas:`**: Recibe dos parámetros (`cantidadFilas` y `cantidadColumnas`) y construye o redimensiona la grilla del tablero. Asigna ambos valores a `filas` y `columnas`, y crea en `casillas` una colección de filas de `1` a `filas` donde cada fila contiene una colección de `1` a `columnas` inicializada con nuevas instancias de `Automata`.
```smalltalk
configurarFilas: cantidadFilas columnas: cantidadColumnas
	filas := cantidadFilas.
	columnas := cantidadColumnas.
	casillas := (1 to: filas) collect: [ :i |
		(1 to: columnas) collect: [ :j | Automata new ] ]
```

* **`contarVecinosFila:columna:`**: Recibe dos coordenadas (`fila` y `columna`) y devuelve un entero con la cantidad de celdas vecinas vivas alrededor de esa posición. Recorre los desplazamientos relativos de filas y columnas entre `-1` y `1` omitiendo la propia celda (`df = 0` y `dc = 0`). Si `toroidal` es verdadero, envuelve las coordenadas usando aritmética modular en base 1 (`((fila - 1 + df) \\ filas) + 1` y `((columna - 1 + dc) \\ columnas) + 1`) para conectar bordes opuestos (izquierda con derecha y arriba con abajo); si `toroidal` es falso, calcula la posición directa y verifica los límites con `estaDentroFila:columna:`. Finalmente suma `1` a `cantidad` por cada vecino válido cuyo autómata responda verdadero a `estaVivo`.
```smalltalk
contarVecinosFila: fila columna: columna
	| cantidad |
	cantidad := 0.
	(-1 to: 1) do: [ :df |
		(-1 to: 1) do: [ :dc |
			| nf nc esValida |
			(df = 0 and: [ dc = 0 ]) ifFalse: [
				toroidal
					ifTrue: [
						nf := ((fila - 1 + df) \\ filas) + 1.
						nc := ((columna - 1 + dc) \\ columnas) + 1.
						esValida := true ]
					ifFalse: [
						nf := fila + df.
						nc := columna + dc.
						esValida := self estaDentroFila: nf columna: nc ].
				(esValida and: [ (self casillaEnFila: nf columna: nc) estaVivo ])
					ifTrue: [ cantidad := cantidad + 1 ] ] ] ].
	^ cantidad
```

* **`estaDentroFila:columna:`**: Recibe dos coordenadas (`f` y `c`) y devuelve un booleano indicando si la posición cae dentro de los límites válidos de la grilla (`1..filas` y `1..columnas`).
```smalltalk
estaDentroFila: f columna: c
	^ (f between: 1 and: filas) and: [ c between: 1 and: columnas ]
```

* **`filas`**: Devuelve el valor almacenado en la variable de instancia `filas`, indicando la cantidad actual de filas (largo) del tablero.
```smalltalk
filas
	^ filas
```

* **`initialize`**: Establece el estado inicial por defecto de cada nueva instancia de `Tablero`. Invoca a `super initialize`, inicializa `toroidal` en `false` (borde finito por defecto) y llama a `self configurarFilas: 10 columnas: 10` para crear una grilla inicial de 10 por 10 autómatas.
```smalltalk
initialize
	super initialize.
	toroidal := false.
	self configurarFilas: 10 columnas: 10
```

* **`mostrarPoblacion`**: Escribe en el `Transcript` la cantidad total de autómatas vivos junto con una representación visual de la grilla en texto (`'O '` si está vivo o `'. '` si está muerto).
```smalltalk
mostrarPoblacion
	Transcript
		show: 'Poblacion: ', self calcularPoblacion printString;
		cr.
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			| simbolo |
			simbolo := ((self casillaEnFila: i columna: j) estaVivo)
				ifTrue: [ 'O ' ]
				ifFalse: [ '. ' ].
			Transcript show: simbolo ].
		Transcript cr ]
```

* **`toroidal`**: Devuelve el valor booleano almacenado en la variable de instancia `toroidal`, permitiendo consultar sin modificar si el modo de borde toroidal está activado (`true`) o desactivado (`false`).
```smalltalk
toroidal
	^ toroidal
```

---

### 3. Clase `TableroVista`

La clase `TableroVista` provee una interfaz gráfica interactiva en Morphic para visualizar y controlar un tablero del Juego de la Vida de Conway, hereda de `Morph` y pertenece al paquete `JuegoDeLaVidaDeConway`. Declara las variables de instancia `tablero`, `tamanoCelda`, `corriendo`, `generacion` y `etiquetaInfo`, y se encarga de dibujar la grilla, responder a clics del usuario, avanzar generaciones de forma manual o automática, generar configuraciones aleatorias, activar o desactivar el borde toroidal, redimensionar el tablero y mostrar en tiempo real la generación, población y estado del borde toroidal en la barra superior.

#### Definición de la clase
```smalltalk
Morph << #TableroVista
	slots: { #tablero . #tamanoCelda . #corriendo . #generacion . #etiquetaInfo };
	package: 'JuegoDeLaVidaDeConway'
```

#### Métodos de instancia

* **`abrirVentana`**: Construye y abre en el mundo de Morphic una ventana interactiva (`SystemWindow` titulada `'Juego de la Vida de Conway'`) que combina la vista del tablero con una barra superior de controles (`Siguiente Gen`, `Play / Pausa`, `Aleatorio`, `Borde Toroidal`, `Cambiar Tamano`, `Limpiar` y `etiquetaInfo`).
```smalltalk
abrirVentana
	| ventana barra btnPaso btnPlay btnAleatorio btnToroidal btnTamano btnLimpiar |
	ventana := SystemWindow labelled: 'Juego de la Vida de Conway'.
	barra := Morph new
		layoutPolicy: RowLayout new;
		color: Color darkGray;
		yourself.

	btnPaso := SimpleButtonMorph new
		label: 'Siguiente Gen';
		target: self;
		actionSelector: #paso;
		yourself.

	btnPlay := SimpleButtonMorph new
		label: 'Play / Pausa';
		target: [ corriendo := corriendo not ];
		actionSelector: #value;
		yourself.

	btnAleatorio := SimpleButtonMorph new
		label: 'Aleatorio';
		target: self;
		actionSelector: #generarAleatorio;
		yourself.

	btnToroidal := SimpleButtonMorph new
		label: 'Borde Toroidal';
		target: self;
		actionSelector: #alternarModoToroidal;
		yourself.

	btnTamano := SimpleButtonMorph new
		label: 'Cambiar Tamano';
		target: self;
		actionSelector: #pedirNuevoTamano;
		yourself.

	btnLimpiar := SimpleButtonMorph new
		label: 'Limpiar';
		target: [
			tablero configurarFilas: tablero filas columnas: tablero columnas.
			generacion := 0.
			self actualizarInfo.
			self changed ];
		actionSelector: #value;
		yourself.

	self actualizarInfo.
	barra
		addMorphBack: btnPaso;
		addMorphBack: btnPlay;
		addMorphBack: btnAleatorio;
		addMorphBack: btnToroidal;
		addMorphBack: btnTamano;
		addMorphBack: btnLimpiar;
		addMorphBack: etiquetaInfo.

	ventana addMorph: barra frame: (0@0 corner: 1@0.12).
	ventana addMorph: self frame: (0@0.12 corner: 1@1).
	ventana openInWorld
```

* **`actualizarDimensionVisual`**: Ajusta el tamaño gráfico del `Morph` de acuerdo con las dimensiones del tablero (`tablero columnas * tamanoCelda @ (tablero filas * tamanoCelda)`) y solicita el redibujado con `self changed`.
```smalltalk
actualizarDimensionVisual
	self extent: (tablero columnas * tamanoCelda) @ (tablero filas * tamanoCelda).
	self changed
```

* **`actualizarInfo`**: Refresca el texto de `etiquetaInfo` mostrando la generación actual, la población viva y el estado del borde toroidal (`'Activado'` o `'Desactivado'`) consultando `tablero toroidal` sin modificarlo.
```smalltalk
actualizarInfo
	| estadoBorde |
	etiquetaInfo ifNotNil: [
		estadoBorde := tablero toroidal
			ifTrue: [ 'Activado' ]
			ifFalse: [ 'Desactivado' ].
		etiquetaInfo contents:
			'  Generacion: ', generacion printString,
			'  |  Poblacion: ', tablero calcularPoblacion printString,
			'  |  Borde toroidal: ', estadoBorde ]
```

* **`alternarModoToroidal`**: Responde al botón `Borde Toroidal` de la barra. Invoca `tablero alternarToroidal` para invertir el modo de borde en el modelo y luego llama a `self actualizarInfo` y `self changed` para reflejar el cambio en la barra.
```smalltalk
alternarModoToroidal
	tablero alternarToroidal.
	self actualizarInfo.
	self changed
```

* **`drawOn:`**: Dibuja la grilla completa sobre el lienzo `aCanvas`, pintando cada celda en verde oscuro (`Color green muchDarker`) si su autómata está vivo o en blanco (`Color white`) si está muerto, junto con un borde de 1 píxel en gris claro (`Color lightGray`).
```smalltalk
drawOn: aCanvas
	super drawOn: aCanvas.
	1 to: tablero filas do: [ :f |
		1 to: tablero columnas do: [ :c |
			| rect colorCelda |
			rect := ((self left + ((c - 1) * tamanoCelda)) @ (self top + ((f - 1) * tamanoCelda)))
				extent: (tamanoCelda @ tamanoCelda).
			colorCelda := ((tablero casillaEnFila: f columna: c) estaVivo)
				ifTrue: [ Color green muchDarker ]
				ifFalse: [ Color white ].
			aCanvas fillRectangle: rect color: colorCelda.
			aCanvas frameRectangle: rect width: 1 color: Color lightGray ] ]
```

* **`generarAleatorio`**: Responde al botón `Aleatorio` de la barra. Invoca `tablero aleatorizar`, reinicia `generacion` en `0`, actualiza la etiqueta informativa y solicita el redibujado de la grilla.
```smalltalk
generarAleatorio
	tablero aleatorizar.
	generacion := 0.
	self actualizarInfo.
	self changed
```

* **`handlesMouseDown:`**: Devuelve `true` para indicar que este `Morph` captura los eventos de clic del mouse sobre su área.
```smalltalk
handlesMouseDown: anEvent
	^ true
```

* **`initialize`**: Configura los valores iniciales de la vista al crearse: instancia `Tablero new`, fija `tamanoCelda := 24`, `corriendo := false`, `generacion := 0`, crea `etiquetaInfo`, actualiza la información y ajusta las dimensiones visuales.
```smalltalk
initialize
	super initialize.
	tablero := Tablero new.
	tamanoCelda := 24.
	corriendo := false.
	generacion := 0.
	etiquetaInfo := StringMorph new
		color: Color white;
		yourself.
	self actualizarInfo.
	self actualizarDimensionVisual
```

* **`mouseDown:`**: Convierte la posición del clic en índices de fila y columna y, si cae dentro de los límites del tablero, alterna el estado del autómata en esa casilla, actualiza la barra de información y redibuja.
```smalltalk
mouseDown: anEvent
	| posRelativa fila columna |
	posRelativa := anEvent position - self position.
	columna := (posRelativa x // tamanoCelda) + 1.
	fila := (posRelativa y // tamanoCelda) + 1.
	(tablero estaDentroFila: fila columna: columna) ifTrue: [
		tablero alternarEnFila: fila columna: columna.
		self actualizarInfo.
		self changed ]
```

* **`paso`**: Avanza la simulación una generación en el tablero (`tablero avanzarGeneracion`), incrementa `generacion` en `1`, actualiza `etiquetaInfo` y solicita el redibujado con `self changed`.
```smalltalk
paso
	tablero avanzarGeneracion.
	generacion := generacion + 1.
	self actualizarInfo.
	self changed
```

* **`pedirNuevoTamano`**: Solicita mediante cuadros de diálogo (`UIManager default request:initialAnswer:`) la nueva cantidad de filas y de columnas, valida que sean mayores que `0`, reconfigura el tablero, reinicia `generacion` en `0` y ajusta el tamaño de la vista.
```smalltalk
pedirNuevoTamano
	| fTexto cTexto f c |
	fTexto := UIManager default request: 'Cantidad de filas (largo):' initialAnswer: tablero filas printString.
	fTexto ifNil: [ ^ self ].
	cTexto := UIManager default request: 'Cantidad de columnas (ancho):' initialAnswer: tablero columnas printString.
	cTexto ifNil: [ ^ self ].
	f := fTexto asInteger ifNil: [ tablero filas ].
	c := cTexto asInteger ifNil: [ tablero columnas ].
	(f > 0 and: [ c > 0 ]) ifTrue: [
		tablero configurarFilas: f columnas: c.
		generacion := 0.
		self actualizarInfo.
		self actualizarDimensionVisual ]
```

* **`step`**: Ejecuta automáticamente `self paso` en cada ciclo de animación de Morphic mientras `corriendo` sea `true`.
```smalltalk
step
	corriendo ifTrue: [ self paso ]
```

* **`stepTime`**: Define el intervalo de tiempo en milisegundos (`300 ms`) entre cada ejecución automática de `step`.
```smalltalk
stepTime
	^ 300 "milisegundos entre generaciones"
```

---

## ▶️ Ejecución desde el Playground

Para iniciar el juego en **Pharo 14**, ejecuta en el **Playground** con **Do it** (`Ctrl + D` / `Cmd + D`):

```smalltalk
TableroVista new abrirVentana.
```

Esta sentencia instancia `TableroVista` (ejecutando `initialize`, que prepara un `Tablero` de `10 x 10` celdas con `toroidal` desactivado por defecto, celdas de `24 px`, `corriendo` en `false` y `generacion` en `0`) y abre la ventana interactiva `SystemWindow` en el entorno gráfico de Pharo.
