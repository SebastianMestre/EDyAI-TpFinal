# Observaciones


A lo largo del trabajo, se establecieron estas normas para aportar consistencia
al codigo:

- Las funciones documentan si toman ownership sobre alguno de sus argumentos (a
falta de documentacion, se debe suponer que NO toman ownership).
- Por lo tanto, si es necesario, generar copias de valores que se pasan como
argumento, es responsabilidad de quien llama.
- Se considera que cualquier objeto se puede 'mover' copiando los bytes que lo
componen a un nuevo lugar, y no volviendo a usar el objeto original.
- Algunas estructuras pequennas se pasan por valor (en particular, si no hay una
buena razon para pasar por puntero, Span se pasa por valor, lo mismo para
OptionalContactId, etc).
- Las estructuras de datos guardan sus datos por valor. Eso es, no se guarda un
puntero a los datos, se guardan los bytes que componen el valor. (A no ser que se
considere al puntero mismo como un valor).

# Dificultades

Resulto problematico tener que referirse al mismo contacto en distintas
estructuras de datos. En particular, mantenerlos sincronizados y hacer las copias
y destrucciones necesarias se considero demasiado complejo.

Para evitar estos problemas, se tomaron estas medidas:

- Centralizar el almacenamiento de los contactos en una sola estructura, lo cual
simplifico el manejo de memoria significativamente.
- Usar handles numericos en vez de punteros para referirse a los contactos, lo
cual da mas flexibilidad con como se puede implementar el almacenamiento.
- Interpretar la modificacion de un contacto como su borrado, seguido de la
creacion de un contacto nuevo, con los cambios solicitados.

De estos puntos salen las consecuencias:

- Las estructuras auxiliares solo necesitan guardar handles, que son triviales de
copiar y faciles de destruir.
- Los contactos pueden ser inmutables una vez que se crean.
- Es posible tener una representacion unica para las operaciones (un par de
handles opcionales)

# Modulos

- io: helpers para entrada/salida por consola
- string: helpers para el uso de strings
- types: tipos utiles, por ejemplo para simplificar el uso de punteros a funcion
- contact: la definicion del tipo de dato Contact
- history: un contenedor de operaciones, se usa para implementar deshacer/rehacer
- index: una estructura de datos para la busqueda eficiente por nombre y apellido
- database: un objeto que maneja las operaciones y el deshacer/rehacer
- storage: un contenedor de contactos. Maneja la memoria de cada contacto.
- estructuras de datos: span, vector, circular\_buffer, bst, slot\_map

# Estructuras de datos

## Span

Un puntero a un bloque de memoria contigua, con conocimiento del tamanno del
bloque. Resulto muy util como pieza de construccion para otras estructuras de
datos mas complejas.

Existe el macro SPANOF(x), que evalua a un Span que apunta al espacio de memoria
que ocupa el argumento. El argumento debe ser un lvalue.

Para clarificar el uso de SPANOF se ofrece este ejemplo de codigo:

```c
int x = 100;
Span s = SPANOF(x);
assert(&x == s.begin);
```

## Vector

Un array dinamico. Esta es una estructura de datos de proposito general muy util.
Entre otras cosas, sirvio como pieza de construccion para al menos una estructura
mas compleja.

## CircularBuffer

Una (casi doble) cola circular. En el trabajo se utiliza para implementar la
funcionalidad de deshacer/rehacer.

Use una cola circular ya que se alineaba muy bien con los requisitos del trabajo:
recordar una cantidad maxima de operaciones, y olvidar las mas viejas si esta
cantidad se supera.

## Bst

Un arbol AVL. En el trabajo, se utiliza para realizar busquedas por nombre y
apellido de forma eficiente.

La principal razon de su uso (en vez de usar una tabla hash) fue desafiarme a mi
mismo. He implementado tablas hash de distintos tipos en varias ocasiones, pero
nunca habia implementado un AVL correctamente.

## SlotMap

Un slot map es un contenedor que soporta insercion, acceso aleatorio, y borrado
en tiempo constante. A diferencia de otros diccionarios, las claves son numericas
y determinadas por la estructura misma al insertar un elemento. Aparte, la clave
no depende del valor que se guarda.

Esto resulta util ya que el uso que se le da es ser un almacenamiento de datos
con posibles repetidos. Otra caracteristica que llevo a su uso es que la
implementacion es bastante sencilla.

# Algoritmos


## deshacer/rehacer

Use una cola circular, que guarda las operaciones que se realizaron. Al deshacer,
se ejecuta la operacion inversa.

Si la operacion fue de creacion, se ejecuta el borrado, y viceversa.

## guardar ordenado

Use quicksort sobre un array.

## buscar por suma de edades

Use programacion dinamica. El problema es muy parecido al conocido 'subset sum'.

# Compilacion/Invocacion

# Bibliografia

Para la implementacion del AVL, me base en la implementacion de la Catedra,
algunas observaciones personales, y [1]. Como referencia, use las manpages de
Ubuntu y [2]. Aparte, tome varias ideas de estilo de [3].

[1] https://www.geeksforgeeks.org/avl-tree-set-2-deletion/
[2] https://en.cppreference.com/
[3] https://floooh.github.io/2019/09/27/modern-c-for-cpp-peeps