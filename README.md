# Lenguaje M en el Editor Avanzado — Limpieza de datos

Práctica de limpieza de datos de ventas (TechStore) escribiendo código M a mano en el
Editor Avanzado de Power Query, sin usar la interfaz gráfica.

## Estructura del repositorio

```
lenguaje-m-editor-avanzado/
├── script_limpieza.md   → código M completo y comentado
└── README.md            → este archivo
```

## Preguntas conceptuales

### 1. ¿Qué hace exactamente el bloque `let...in`? ¿Por qué cada paso puede referenciar al anterior?

El bloque `let...in` es la estructura básica de una consulta en M. Entre `let` y `in` se
definen una serie de **pasos**, que en realidad son variables: cada uno tiene un nombre, un
signo `=` y una expresión. Después del `in` va la expresión final que la consulta devuelve,
que normalmente es el nombre del último paso.

Cada paso puede referenciar al anterior porque los pasos no son instrucciones que se ejecutan
de arriba hacia abajo como en un lenguaje imperativo, sino **definiciones de valores dentro de
un mismo ámbito (scope)**. Cuando escribo `LimpiarEspacios = Table.TransformColumns(Origen, ...)`,
estoy diciendo que el valor `LimpiarEspacios` se calcula a partir del valor `Origen`, que ya
está definido en el mismo bloque. M resuelve estas dependencias y evalúa cada paso cuando hace
falta. El resultado es un encadenamiento: `Origen → LimpiarEspacios → EstandarizarCategoria →
EliminarPruebas → TiparColumnas`, donde cada transformación toma como entrada el resultado de
la anterior. Esto hace que la consulta sea legible y fácil de depurar, porque puedo ver el
resultado intermedio de cualquier paso.

### 2. ¿Por qué M es Case Sensitive y qué consecuencia práctica tiene? Dá un ejemplo.

M es case sensitive (distingue mayúsculas de minúsculas) porque así fue diseñado el lenguaje:
los identificadores, nombres de funciones, nombres de pasos y comparaciones de texto tienen en
cuenta las mayúsculas. `Table.SelectRows` y `table.selectrows` son, para M, dos cosas distintas,
y solo la primera existe como función nativa.

La consecuencia práctica es doble:

- **En el código:** todas las funciones nativas de M empiezan con mayúscula
  (`Text.Trim`, `Table.TransformColumns`). Si las escribo en minúscula, M no las encuentra y
  tira error. Lo mismo con los nombres de pasos: si defino un paso como `LimpiarEspacios` y
  después lo referencio como `limpiarespacios`, la consulta se rompe porque para M son
  identificadores diferentes.

- **En los datos:** las comparaciones de texto también distinguen mayúsculas. Un ejemplo
  concreto de este ejercicio: si filtrara los registros de prueba **antes** de estandarizar la
  categoría, la condición `[categoria] <> "PRUEBA"` solo eliminaría las filas escritas
  exactamente como "PRUEBA", pero dejaría pasar cualquier fila con "prueba" o "Prueba". Por eso
  primero estandarizo con `Text.Proper` (que convierte todo a "Prueba") y recién después filtro
  con una única condición `<> "Prueba"`.

### 3. ¿Cuál es la diferencia entre `Text.Trim` y `Text.Clean`?

Las dos limpian texto, pero atacan problemas distintos:

- **`Text.Trim`** elimina los **espacios en blanco** al inicio y al final de un texto (y por
  defecto también espacios repetidos). Por ejemplo, `Text.Trim(" Laptop ")` devuelve `"Laptop"`.
  Es lo que usé para arreglar `nombre_producto`, donde el problema eran espacios sobrantes en
  los bordes.

- **`Text.Clean`** elimina los **caracteres no imprimibles** o de control (saltos de línea,
  tabulaciones, retornos de carro, etc.), que muchas veces vienen ocultos en exportaciones de
  sistemas legacy y no se ven a simple vista. No toca los espacios normales.

En resumen: `Text.Trim` se ocupa de los espacios visibles de los bordes; `Text.Clean` se ocupa
de la "basura" invisible que ensucia el texto. En la práctica suelen combinarse
(`Text.Clean(Text.Trim(...))`) cuando los datos vienen muy sucios.

### 4. ¿Por qué filtraste los registros "PRUEBA" después de estandarizar y no antes?

Por el punto 2: M es case sensitive en las comparaciones de texto. En los datos originales, la
categoría de prueba aparece escrita como "PRUEBA" (todo en mayúsculas), pero podría venir como
"prueba", "Prueba", etc. Si filtrara antes de estandarizar, tendría que escribir múltiples
condiciones (`<> "PRUEBA"` y `<> "prueba"` y `<> "Prueba"`...) para cubrir todas las variantes,
y aun así podría escaparse alguna.

Estandarizando primero con `Text.Proper`, todas esas variantes quedan convertidas a una única
forma: "Prueba". Después, un solo filtro `[categoria] <> "Prueba"` las elimina a todas de forma
segura. El orden de los pasos importa: primero normalizo, después filtro. Así el filtro es
simple, robusto y no depende de cómo estaba escrita originalmente cada fila.

