# Decisiones de diseño — Semana 3

## 1. Punto de entrada

El proyecto mantiene un único punto de entrada:

```text
IngestaSensores.main()
```

No se crean aplicaciones independientes por semana.

`BancoDePruebas` es una clase auxiliar y no contiene `main`.

## 2. Búsqueda por timestamp

Se utilizan dos estrategias:

- Búsqueda lineal: no requiere ordenamiento.
- Búsqueda binaria: requiere que el arreglo esté ordenado por timestamp.

Los datos sintéticos de `GeneradorDatos` se generan en orden cronológico, por lo que la búsqueda binaria por timestamp cumple su precondición.

## 3. Búsqueda por PM2.5

No se asume que los datos estén ordenados por PM2.5.

Por tanto, la búsqueda binaria por PM2.5 se conserva como experimento para demostrar el efecto de una precondición incumplida.

## 4. Comparación de String

Los identificadores de estación se comparan mediante:

```java
equals()
```

y no mediante:

```java
==
```

porque se necesita comparar contenido.

## 5. Medición

La comparación principal entre algoritmos utiliza el número de comparaciones.

El tiempo en milisegundos se conserva como evidencia experimental, pero no es la única medida utilizada.

## 6. Evolución del proyecto

La Semana 3 agrega una nueva capacidad a la misma plataforma:

```text
Sensores
   ↓
Ingesta
   ↓
Repositorio
   ↓
Búsqueda
   ↓
Medición de eficiencia
```

La Semana 4 podrá extender esta misma arquitectura para estudiar ordenamiento.

# Decisiones de diseño — Semana 5

## 7. Arreglo vs. lista enlazada

Durante la Semana 5 se compararon dos implementaciones de la interfaz `Repositorio`: `RepositorioArreglo` y `RepositorioEnlazado`.

Ambas permiten realizar las mismas operaciones mediante un contrato común, pero sus costos son diferentes dependiendo de la operación realizada.

Para inserciones frecuentes al inicio se prefiere una lista enlazada, porque solo necesita crear un nuevo nodo y modificar enlaces. En cambio, un arreglo debe desplazar los elementos existentes.

Para consultas frecuentes por posición se prefiere un arreglo, ya que permite acceso directo por índice, mientras que una lista enlazada debe recorrer los nodos desde el inicio hasta llegar a la posición solicitada.

## 8. Uso del puntero `ultimo`

`RepositorioEnlazado` mantiene una referencia al último nodo de la lista.

Esto permite implementar `agregarAlFinal()` sin recorrer todos los nodos existentes. El nodo que antes era el último se conecta con el nuevo nodo y después se actualiza la referencia `ultimo`.

De esta forma, agregar al final no requiere recorrer toda la cadena.

## 9. Eliminación de nodos

Al eliminar un nodo intermedio no se debe cortar la cadena asignando `null` al enlace del nodo anterior.

El nodo anterior debe conectarse con el nodo que estaba después del elemento eliminado.

Los tests de `RepositorioEnlazado` verificaron correctamente la eliminación del primer nodo, un nodo intermedio y el último nodo, además de comprobar la consistencia entre `tamano()` y `contarRecorriendo()`.

Resultado de pruebas:

```text
RepositorioEnlazadoTest: 5 pruebas aprobadas
```

## 10. Lista doblemente enlazada

Para `HistorialNavegable` se utiliza una lista doblemente enlazada.

Cada nodo mantiene una referencia al nodo `siguiente` y otra al nodo `anterior`. Esto permite recorrer el historial en ambos sentidos sin regresar al comienzo de la lista.

El atributo `cursor` representa la lectura en la que se encuentra actualmente el operador.

Los tests comprobaron correctamente el recorrido hacia adelante, el recorrido hacia atrás y los métodos de navegación `siguiente()` y `anterior()`.

Resultado de pruebas:

```text
HistorialNavegableTest: 3 pruebas aprobadas
```

## 11. Experimento de inserción al inicio

Se comparó el costo de insertar lecturas al inicio de un arreglo y de una lista enlazada.

Resultados obtenidos:

```text
Lecturas     Arreglo: movimientos     Lista: saltos
10.000              49.995.000             0
50.000           1.249.975.000             0
100.000          4.999.950.000             0
```

En el arreglo, cada nueva inserción al inicio obliga a desplazar los elementos existentes.

En la lista enlazada no es necesario recorrer la estructura. El nuevo nodo se conecta con el antiguo primer nodo y pasa a convertirse en `primero`.

Por esta razón, para inserciones frecuentes al inicio la lista enlazada resulta más conveniente.

## 12. Acceso por posición

También se comparó el recorrido utilizando `obtener(i)` para cada posición.

Resultados obtenidos:

```text
Lecturas     Arreglo: accesos     Lista: saltos
1.000                  1.000           499.500
10.000                10.000        49.995.000
50.000                50.000     1.249.975.000
```

El arreglo permite acceder directamente a una posición mediante su índice.

En una lista enlazada, `obtener(i)` debe comenzar desde el primer nodo y avanzar hasta la posición solicitada. Repetir esta operación para todos los índices provoca una gran cantidad de recorridos.

Por lo tanto, aunque el código que utiliza ambas estructuras puede verse similar, el costo de ejecución puede ser muy diferente.

## 13. Búsqueda binaria sobre una lista enlazada

Se comparó una búsqueda lineal con una búsqueda binaria sobre una lista enlazada de 50.000 elementos.

Resultados obtenidos:

```text
Búsqueda lineal:
posición encontrada = 49999
saltos = 50.000

Búsqueda binaria:
posición encontrada = 49999
saltos = 749.990
```

La búsqueda binaria realiza pocas comparaciones, pero necesita acceder repetidamente a posiciones intermedias.

En un arreglo este acceso es directo. En una lista enlazada, llegar a cada posición media requiere recorrer nodos desde el inicio.

Por esta razón, la búsqueda binaria no resulta necesariamente eficiente sobre una lista enlazada.

## 14. Decisión final de estructura

No se considera que una estructura sea mejor en todos los casos.

La decisión depende de las operaciones que predominan:

- Si predominan inserciones al inicio, se prefiere una lista enlazada.
- Si predominan consultas por posición, se prefiere un arreglo.
- Si se necesita navegación hacia adelante y hacia atrás, se utiliza una lista doblemente enlazada.

Los experimentos muestran que la elección de la estructura de datos debe realizarse junto con el algoritmo que se utilizará sobre ella.