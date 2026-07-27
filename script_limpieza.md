# Script de limpieza — Lenguaje M (Editor Avanzado)

Este script limpia el archivo de ventas exportado desde el sistema legacy de TechStore,
directamente en código M, sin usar la interfaz gráfica de Power Query.

## Código M

```m
let
    // Paso 1: Fuente de datos original escrita a mano con Table.FromRecords.
    // Todos los valores se cargan como texto para controlar el tipado manualmente después.
    Origen = Table.FromRecords({
        [id_venta = "1", nombre_producto = " Laptop Pro 15 ", categoria = "Computación", precio = "1200.00", fecha_venta = "2024-01-05"],
        [id_venta = "2", nombre_producto = "Mouse Inalámbrico", categoria = "accesorios", precio = "28.00", fecha_venta = "2024-01-08"],
        [id_venta = "3", nombre_producto = " Teclado Mecánico", categoria = "PRUEBA", precio = "95.00", fecha_venta = "2024-01-12"],
        [id_venta = "4", nombre_producto = "Monitor 4K ", categoria = "computación", precio = "450.00", fecha_venta = "2024-02-03"],
        [id_venta = "5", nombre_producto = " Auriculares BT", categoria = "Audio", precio = "120.00", fecha_venta = "2024-02-10"],
        [id_venta = "6", nombre_producto = "SSD Externo 1TB ", categoria = "PRUEBA", precio = "130.00", fecha_venta = "2024-03-05"],
        [id_venta = "7", nombre_producto = "Webcam HD", categoria = "Accesorios", precio = "85.00", fecha_venta = "2024-03-12"]
    }),
    // Paso 2: Eliminar espacios en blanco al inicio y al final de nombre_producto usando Text.Trim.
    LimpiarEspacios = Table.TransformColumns(Origen, {{"nombre_producto", Text.Trim, type text}}),
    // Paso 3: Estandarizar la columna categoria a Title Case con Text.Proper
    // para unificar "computación", "COMPUTACIÓN" y "Computación" en una sola forma.
    EstandarizarCategoria = Table.TransformColumns(LimpiarEspacios, {{"categoria", Text.Proper, type text}}),
    // Paso 4: Filtrar y eliminar los registros de prueba.
    // Se filtra DESPUÉS de estandarizar, así "PRUEBA", "prueba" o "Prueba" ya quedaron
    // convertidos a "Prueba" y una única condición los atrapa a todos.
    EliminarPruebas = Table.SelectRows(EstandarizarCategoria, each [categoria] <> "Prueba"),
    // Paso 5: Definir los tipos de datos correctos de cada columna.
    // Se pasa "en-US" como tercer argumento para que el precio use el punto como
    // separador decimal (1200.00 y no 120000) y la fecha se interprete como año-mes-día.
    TiparColumnas = Table.TransformColumnTypes(EliminarPruebas, {
        {"id_venta", Int64.Type},
        {"nombre_producto", type text},
        {"categoria", type text},
        {"precio", type number},
        {"fecha_venta", type date}
    }, "en-US")
in
    TiparColumnas
```

## Resultado esperado

La tabla final tiene **5 filas** (se eliminaron los registros 3 y 6, de categoría "PRUEBA"):

| id_venta | nombre_producto  | categoria   | precio | fecha_venta |
|----------|------------------|-------------|--------|-------------|
| 1        | Laptop Pro 15    | Computación | 1200   | 2024-01-05  |
| 2        | Mouse Inalámbrico| Accesorios  | 28     | 2024-01-08  |
| 4        | Monitor 4K       | Computación | 450    | 2024-02-03  |
| 5        | Auriculares BT   | Audio       | 120    | 2024-02-10  |
| 7        | Webcam HD        | Accesorios  | 85     | 2024-03-12  |

- `nombre_producto` sin espacios al inicio ni al final.
- `categoria` estandarizada en Title Case.
- Tipos de datos correctos: `id_venta` entero, `precio` decimal, `fecha_venta` fecha, el resto texto.
