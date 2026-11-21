# 🧬 Juego de la Vida de Conway (Conway's Game of Life) en Pharo 14

Implementación del autómata celular **Juego de la Vida de John Conway** desarrollado en **Pharo 14 (Smalltalk)** bajo el paradigma de Programación Orientada a Objetos (POO) y con interfaz gráfica interactiva construida en **Morphic**.

---

## 📋 Tabla de Contenidos
1. [Descripción del Problema y Reglas](#-descripción-del-problema-y-reglas)
2. [Arquitectura Orientada a Objetos](#-arquitectura-orientada-a-objetos)
3. [Estructura de Clases y Explicación Técnica](#-estructura-de-clases-y-explicación-técnica)
4. [Código Fuente Completo (Pharo 14)](#-código-fuente-completo-pharo-14)
5. [Cómo Ejecutarlo en Pharo 14](#-cómo-ejecutarlo-en-pharo-14)
6. [Variación Implementada: Grilla Toroidal](#-variación-implementada-grilla-toroidal)

---

## 🧩 Descripción del Problema y Reglas

El Juego de la Vida es un autómata celular bidimensional compuesto por una grilla de autómatas finitos (células) con dos estados posibles: `0` (**muerta**) o `1` (**viva**). La evolución de la grilla de un instante $t$ al siguiente $t+1$ ocurre de manera **simultánea** evaluando los 8 vecinos circundantes (**vecindad de Moore**) bajo las siguientes reglas de transición:

1. **Supervivencia / Estabilidad:** Si una célula tiene exactamente **2 vecinos vivos**, mantiene su estado actual en $t+1$.
2. **Nacimiento / Supervivencia:** Si una célula tiene exactamente **3 vecinos vivos**, su estado en $t+1$ será **1 (viva)** (nace si estaba muerta, o sobrevive si estaba viva).
3. **Muerte (soledad o sobrepoblación):** Si una célula tiene **menos de 2** o **más de 3 vecinos vivos**, su estado en $t+1$ será **0 (muerta)**.

---

## 🏗️ Arquitectura Orientada a Objetos

El diseño separa el sistema en **dos capas independientes** para garantizar bajo acoplamiento, encapsulamiento y extensibilidad:

```text
+-----------------------------------+       +-----------------------------------+
|         CAPA DE MODELO            |       |       CAPA DE VISTA (Morphic)     |
|       (Lógica del Dominio)        |       |       (Interfaz Gráfica)          |
+-----------------------------------+       +-----------------------------------+
|                                   |       |                                   |
|  GrillaVida                       |<------|  JuegoDeLaVidaMorph               |
|   ├── filas, columnas, generacion |       |   ├── etiquetaInfo (StringMorph)  |
|   └── celdas (Array de Arrays)    |       |   ├── botones (SimpleButtonMorph) |
|         └── Celula (estado: 0|1)  |<------|   └── celdasMorph (CelulaMorph)   |
|                                   |       |                                   |
|  GrillaVidaToroidal (Subclase)    |       |                                   |
+-----------------------------------+       +-----------------------------------+
```

* **Modelo (`Celula`, `GrillaVida`, `GrillaVidaToroidal`):** Contiene únicamente la representación matemática de la grilla, el conteo de vecinos, la población y las reglas de evolución. No depende de ningún componente visual.
* **Vista (`CelulaMorph`, `JuegoDeLaVidaMorph`):** Dibuja la grilla en pantalla mediante Morphic, captura los clics del mouse para configurar patrones iniciales y actualiza la interfaz en cada paso de tiempo.

---

## 🔍 Estructura de Clases y Explicación Técnica

### 1. `Celula` (Modelo)
* **Superclase:** `Object` | **Slots:** `estado`
* **Responsabilidad:** Representa un autómata finito individual y encapsula las reglas de transición de Conway.
* **Métodos clave:**
  * `initialize`: Inicializa `estado := 0` (muerta por defecto).
  * `estaViva`, `nacer`, `morir`, `alternarEstado`: Consultan y modifican su propio estado sin exponer su variable interna directamente a otros objetos.
  * `proximoEstadoConVecinos:`: Calcula el estado futuro ($t+1$) sin alterar todavía su estado actual ($t$).

### 2. `GrillaVida` (Modelo)
* **Superclase:** `Object` | **Slots:** `filas`, `columnas`, `celdas`, `generacion`
* **Responsabilidad:** Administra la matriz 2D parametrizable (`Array` de `Array`s), calcula vecinos en frontera finita, mide la población en $t$ y ejecuta la transición simultánea.
* **Métodos clave:**
  * `filas:columnas:` *(Class side)*: Constructor parametrizado que instancia e inicializa la grilla.
  * `contarVecinosVivosEnFila:columna:`: Explora los desplazamientos `df` y `dc` en `[-1, 1]` descartando `(0, 0)` y verificando límites con `between:and:`.
  * `avanzarGeneracion`: Implementado en **dos fases** (primero calcula todos los estados futuros en `futurosEstados` y luego los aplica simultáneamente) para evitar que el cambio de una célula contamine el cálculo de sus vecinas en la misma generación.
  * `poblacion`: Cuenta cuántas células tienen estado `1` en el tiempo actual $t$.

### 3. `GrillaVidaToroidal` (Modelo - Variación)
* **Superclase:** `GrillaVida` | **Slots:** `{}`
* **Responsabilidad:** Implementa una topología toroidal (mundo sin bordes, donde el borde superior conecta con el inferior y el izquierdo con el derecho).
* **Polimorfismo:** Sobrescribe únicamente `contarVecinosVivosEnFila:columna:` usando aritmética modular (`\\`) en base 1, reutilizando toda la lógica de evolución y la interfaz gráfica sin modificarlas.

### 4. `CelulaMorph` (Vista)
* **Superclase:** `BorderedMorph` | **Slots:** `celula`, `accionClick`
* **Responsabilidad:** Representa visualmente una `Celula` como un cuadrado de `20 @ 20` píxeles (negro = viva, blanco = muerta) y captura clics mediante `handlesMouseDown:` y `mouseDown:`.

### 5. `JuegoDeLaVidaMorph` (Vista)
* **Superclase:** `BorderedMorph` | **Slots:** `grilla`, `celdasMorph`, `etiquetaInfo`
* **Responsabilidad:** Contenedor visual principal que muestra el tiempo $t$, la población actual, los botones de simulación (`Paso (t+1)` y `+10 Gen`) y la grilla interactiva.

---

## 💻 Código Fuente Completo (Pharo 14)

Todos los elementos pertenecen al paquete `'JuegoDeLaVida'`.

### Clase `Celula`

```smalltalk
Object << #Celula
	slots: { #estado };
	package: 'JuegoDeLaVida'
```

**Métodos de instancia (`Celula`):**
```smalltalk
initialize
	super initialize.
	estado := 0.

estado
	^ estado

estaViva
	^ estado = 1

nacer
	estado := 1

morir
	estado := 0

alternarEstado
	self estaViva
		ifTrue: [ self morir ]
		ifFalse: [ self nacer ]

proximoEstadoConVecinos: cantidadVecinosVivos
	"Regla 1: con 2 vecinos vivos, mantiene su estado actual.
	 Regla 2: si está muerta y tiene 3 vecinos vivos, nace (1).
	 Regla 3: si está viva y tiene < 2 o > 3 vecinos, muere (0)."
	cantidadVecinosVivos = 2 ifTrue: [ ^ estado ].
	cantidadVecinosVivos = 3 ifTrue: [ ^ 1 ].
	^ 0
```

---

### Clase `GrillaVida`

```smalltalk
Object << #GrillaVida
	slots: { #filas . #columnas . #celdas . #generacion };
	package: 'JuegoDeLaVida'
```

**Método de clase — Class side (`GrillaVida class`):**
```smalltalk
filas: f columnas: c
	^ self new
		inicializarConFilas: f columnas: c;
		yourself
```

**Métodos de instancia (`GrillaVida`):**
```smalltalk
inicializarConFilas: f columnas: c
	filas := f.
	columnas := c.
	generacion := 0.
	celdas := Array new: filas.
	1 to: filas do: [ :i |
		| filaArray |
		filaArray := Array new: columnas.
		1 to: columnas do: [ :j |
			filaArray at: j put: Celula new ].
		celdas at: i put: filaArray ]

filas
	^ filas

columnas
	^ columnas

generacion
	^ generacion

celulaEnFila: i columna: j
	^ (celdas at: i) at: j

poblacion
	"Cuenta cuántas células están en estado 1 en el tiempo t"
	| total |
	total := 0.
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			(self celulaEnFila: i columna: j) estaViva
				ifTrue: [ total := total + 1 ] ] ].
	^ total

contarVecinosVivosEnFila: fila columna: col
	"Recorre las 8 posiciones alrededor (vecindad de Moore) dentro de los límites de la grilla"
	| vivos |
	vivos := 0.
	-1 to: 1 do: [ :df |
		-1 to: 1 do: [ :dc |
			| nf nc |
			(df = 0 and: [ dc = 0 ]) ifFalse: [
				nf := fila + df.
				nc := col + dc.
				((nf between: 1 and: filas) and: [ nc between: 1 and: columnas ])
					ifTrue: [
						(self celulaEnFila: nf columna: nc) estaViva
							ifTrue: [ vivos := vivos + 1 ] ] ] ] ].
	^ vivos

avanzarGeneracion
	"Fase 1: Calcular todos los estados futuros (t+1) sin alterar los actuales (t)"
	| futurosEstados |
	futurosEstados := (1 to: filas) collect: [ :i | Array new: columnas ].
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			| vecinos celulaActual |
			vecinos := self contarVecinosVivosEnFila: i columna: j.
			celulaActual := (celdas at: i) at: j.
			(futurosEstados at: i) at: j put: (celulaActual proximoEstadoConVecinos: vecinos) ] ].

	"Fase 2: Aplicar simultáneamente los nuevos estados"
	1 to: filas do: [ :i |
		1 to: columnas do: [ :j |
			((futurosEstados at: i) at: j) = 1
				ifTrue: [ ((celdas at: i) at: j) nacer ]
				ifFalse: [ ((celdas at: i) at: j) morir ] ] ].

	generacion := generacion + 1

evolucionarGeneraciones: n
	n timesRepeat: [ self avanzarGeneracion ]
```

---

### Clase `GrillaVidaToroidal`

```smalltalk
GrillaVida << #GrillaVidaToroidal
	slots: {};
	package: 'JuegoDeLaVida'
```

**Métodos de instancia (`GrillaVidaToroidal`):**
```smalltalk
contarVecinosVivosEnFila: fila columna: col
	"Vecindad de Moore con envolvente toroidal usando aritmética modular"
	| vivos |
	vivos := 0.
	-1 to: 1 do: [ :df |
		-1 to: 1 do: [ :dc |
			| nf nc |
			(df = 0 and: [ dc = 0 ]) ifFalse: [
				nf := ((fila - 1 + df) \\ filas) + 1.
				nc := ((col - 1 + dc) \\ columnas) + 1.
				((celdas at: nf) at: nc) estaViva
					ifTrue: [ vivos := vivos + 1 ] ] ] ].
	^ vivos
```

---

### Clase `CelulaMorph`

```smalltalk
BorderedMorph << #CelulaMorph
	slots: { #celula . #accionClick };
	package: 'JuegoDeLaVida'
```

**Métodos de instancia (`CelulaMorph`):**
```smalltalk
initialize
	super initialize.
	self extent: 20 @ 20.
	self borderWidth: 1.
	self borderColor: Color gray.
	self color: Color white.

celula: unaCelula
	celula := unaCelula.
	self actualizarColor.

accionClick: unBloque
	accionClick := unBloque

actualizarColor
	celula estaViva
		ifTrue: [ self color: Color black ]
		ifFalse: [ self color: Color white ].

handlesMouseDown: anEvent
	^ true

mouseDown: anEvent
	celula alternarEstado.
	self actualizarColor.
	accionClick ifNotNil: [ accionClick value ]
```

---

### Clase `JuegoDeLaVidaMorph`

```smalltalk
BorderedMorph << #JuegoDeLaVidaMorph
	slots: { #grilla . #celdasMorph . #etiquetaInfo };
	package: 'JuegoDeLaVida'
```

**Método de clase — Class side (`JuegoDeLaVidaMorph class`):**
```smalltalk
filas: f columnas: c
	^ self new
		construirConFilas: f columnas: c;
		yourself
```

**Métodos de instancia (`JuegoDeLaVidaMorph`):**
```smalltalk
construirConFilas: f columnas: c
	| tamCelda anchoGrilla altoGrilla btnPaso btnSimular |
	grilla := GrillaVida filas: f columnas: c.
	tamCelda := 20.
	anchoGrilla := c * tamCelda.
	altoGrilla := f * tamCelda.

	self color: Color lightGray.
	self bounds: (50 @ 50 extent: (anchoGrilla max: 320) @ (altoGrilla + 70)).

	"Etiqueta superior con Tiempo t y Población"
	etiquetaInfo := StringMorph contents: ''.
	etiquetaInfo position: self position + (10 @ 10).
	self addMorph: etiquetaInfo.

	"Botón: Avanzar 1 generación"
	btnPaso := SimpleButtonMorph new.
	btnPaso label: 'Paso (t+1)'.
	btnPaso position: self position + (10 @ 32).
	btnPaso target: self; actionSelector: #pasoSiguiente.
	self addMorph: btnPaso.

	"Botón: Evolucionar 10 generaciones"
	btnSimular := SimpleButtonMorph new.
	btnSimular label: '+10 Gen'.
	btnSimular position: self position + (100 @ 32).
	btnSimular target: self; actionSelector: #simularDiezGeneraciones.
	self addMorph: btnSimular.

	"Construir grilla visual de CelulaMorph usando Array de Arrays"
	celdasMorph := (1 to: f) collect: [ :i |
		(1 to: c) collect: [ :j |
			| cm |
			cm := CelulaMorph new.
			cm celula: (grilla celulaEnFila: i columna: j).
			cm position: self position + (((j - 1) * tamCelda) @ (((i - 1) * tamCelda) + 65)).
			cm accionClick: [ self actualizarVista ].
			self addMorph: cm.
			cm ] ].

	self actualizarVista

actualizarVista
	1 to: grilla filas do: [ :i |
		1 to: grilla columnas do: [ :j |
			((celdasMorph at: i) at: j) actualizarColor ] ].
	etiquetaInfo contents: 'Tiempo (t): ', grilla generacion asString, '  |  Poblacion: ', grilla poblacion asString

pasoSiguiente
	grilla avanzarGeneracion.
	self actualizarVista

simularDiezGeneraciones
	10 timesRepeat: [
		grilla avanzarGeneracion.
		self actualizarVista.
		World doOneCycle.
		(Delay forMilliseconds: 150) wait ]
```

---

## ▶️ Cómo Ejecutarlo en Pharo 14

1. Abre una imagen de **Pharo 14** desde Pharo Launcher.
2. Abre el **System Browser (Calypso)** y crea las clases anteriores dentro del paquete `JuegoDeLaVida` (recordando ubicar los métodos `filas:columnas:` en el **Class side**).
3. Abre el **Playground** (`Ctrl + O` + `W` o desde el *World Menu*), escribe la siguiente expresión y ejecútala con **Do it** (`Ctrl + D` / `Cmd + D`):

```smalltalk
(JuegoDeLaVidaMorph filas: 20 columnas: 25) openInWorld.
```

4. Haz clic sobre las celdas de la grilla para activar células (por ejemplo, formando un oscilador *Blinker* o un *Glider*) y utiliza los botones **`Paso (t+1)`** o **`+10 Gen`** para observar la evolución de las generaciones y la población.

---

## 🔄 Variación Implementada: Grilla Toroidal

Para probar la variación sin bordes (`GrillaVidaToroidal`) desde el Playground:

```smalltalk
| grillaToroidal |
grillaToroidal := GrillaVidaToroidal filas: 10 columnas: 10.
(grillaToroidal celulaEnFila: 1 columna: 2) nacer.
(grillaToroidal celulaEnFila: 1 columna: 3) nacer.
(grillaToroidal celulaEnFila: 1 columna: 4) nacer.
grillaToroidal avanzarGeneracion.
grillaToroidal poblacion.
```