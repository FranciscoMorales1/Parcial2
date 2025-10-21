

---

# **Punto 1 y 2 — Implementación y Pruebas del Analizador CRUD**

---

## **1. Descripción general**

Se partió de la **gramática `gramatica.g4`**, la cual define las operaciones básicas de un lenguaje CRUD inspirado en SQL (`CREATE`, `INSERT`, `SELECT`, `UPDATE`, `DELETE`).

A partir de esta gramática se generaron las clases auxiliares de ANTLR que implementan el **analizador léxico y sintáctico**.
Finalmente, se desarrolló un archivo **`Main.java`** que permite ejecutar el analizador de forma interactiva, mostrando el árbol sintáctico tanto en texto como en forma gráfica.

---

## **2. Ejecución del analizador**

**Windows:**

```bash
java -cp ".;antlr-4.13.2-complete.jar" Main
```

**Linux / macOS:**

```bash
java -cp ".:antlr-4.13.2-complete.jar" Main
```

El programa inicia un entorno interactivo que permite escribir comandos SQL uno por uno.
Cada sentencia se analiza y se muestra su árbol sintáctico tanto en la consola como en una ventana gráfica.

Para finalizar la ejecución, se debe escribir:

```
EXIT
```

o

```
SALIR
```

---

## **3. Casos de prueba sugeridos**

Ingresar los siguientes comandos en la consola del programa (sin incluir `SQL>`):

```sql
CREATE TABLE users (id, name, age);
INSERT INTO users VALUES ('Ana', 22);
SELECT * FROM users;
UPDATE users SET age = 25 WHERE name = 'Ana';
DELETE FROM users WHERE id = 1;
```

Cada instrucción produce un árbol sintáctico que demuestra que la gramática reconoce correctamente las estructuras CRUD.

---

## **4. Archivos incluidos**

* `gramatica.g4`: definición del lenguaje
* `Main.java`: implementación del analizador interactivo
* `Ejemplos.txt`: casos de prueba
* `antlr-4.13.2-complete.jar`: biblioteca requerida por ANTLR
* Archivos `.java` y `.class` generados automáticamente por ANTLR

---

## **5. Resultado final**

El proyecto permite ejecutar sentencias CRUD y visualizar su análisis sintáctico en formato textual y gráfico, cumpliendo con los requerimientos del Punto 2 del parcial.

---

# **Punto 3 — Analizador Sintáctico Ascendente (Python)**

---

## **1. Gramática original**

```
E → E + T | T
T → T * F | F
F → ( E ) | id
```

**Símbolo inicial:** `E`
**Terminales:** `id`, `+`, `*`, `(`, `)`
**No terminales:** `E`, `T`, `F`

Esta gramática representa expresiones aritméticas simples con suma, multiplicación y paréntesis.
Las operaciones tienen la precedencia habitual (`*` sobre `+`) y ambas son asociativas a la izquierda.

---

## **2. Transformación a forma LL(1)**

Para eliminar la recursión izquierda inmediata, se aplica el patrón:

```
A → A α | β    ⇒
A  → β A'
A' → α A' | ε
```

### **Aplicación paso a paso**

**Para E:**

```
E → E + T | T
E  → T E'
E' → + T E' | ε
```

**Para T:**

```
T → T * F | F
T  → F T'
T' → * F T' | ε
```

**Para F:**

```
F → ( E ) | id
```

### **Gramática LL(1) equivalente**

```
E  → T E'
E' → + T E' | ε
T  → F T'
T' → * F T' | ε
F  → ( E ) | id
```

---

## **3. Conjuntos FIRST, FOLLOW y de Predicción**

**FIRST**

```
FIRST(E)  = { (, id }
FIRST(E') = { +, ε }
FIRST(T)  = { (, id }
FIRST(T') = { *, ε }
FIRST(F)  = { (, id }
```

**FOLLOW**

```
FOLLOW(E)  = { ), $ }
FOLLOW(E') = { ), $ }
FOLLOW(T)  = { +, ), $ }
FOLLOW(T') = { +, ), $ }
FOLLOW(F)  = { *, +, ), $ }
```

**Conjuntos de Predicción**

```
E  → T E'         ⇒ { (, id }
E' → + T E'       ⇒ { + }
E' → ε            ⇒ { ), $ }
T  → F T'         ⇒ { (, id }
T' → * F T'       ⇒ { * }
T' → ε            ⇒ { +, ), $ }
F  → ( E )        ⇒ { ( }
F  → id           ⇒ { id }
```

---

## **4. Diseño del Analizador Ascendente (Shift–Reduce)**

### **Objetivo**

Implementar un analizador sintáctico ascendente que trabaje con una pila y reglas de reducción basadas en la gramática original.

### **Estructura**

* **Entrada:** cadena de tokens (`id`, `+`, `*`, `(`, `)`, `$`)
* **Pila:** contiene símbolos terminales y no terminales
* **Operaciones:**

  * **SHIFT:** mueve el siguiente token de la entrada a la pila
  * **REDUCE:** sustituye una secuencia de símbolos de la pila por un no terminal según las producciones

### **Reglas de reducción**

```
(E)     → F
id      → F
T * F   → T
E + T   → E
F       → T
T       → E
```

### **Estrategia**

El analizador examina la pila y el token siguiente (lookahead).
Aplica reducciones solo cuando no haya un operador de mayor precedencia pendiente.
Acepta si al final la pila contiene únicamente `E`.

---

## **5. Implementación en Python**

El código del analizador se encuentra en `parser_asc.py`.
Implementa el proceso de tokenización, análisis con pila y reducción controlada por precedencia.

```python
# parser_asc.py (fragmento resumido)
TOK_ID, TOK_PLUS, TOK_MUL, TOK_LP, TOK_RP, TOK_EOF = 'id', '+', '*', '(', ')', '$'

def tokenize(src): ...
def try_reduce(stack, lookahead): ...
def parse_expression(expr, debug=False): ...
```

El programa lee expresiones desde consola y devuelve `ACEPTA` o `RECHAZA`.

---

## **6. Pruebas de funcionamiento**

Puedes ingresar todas las pruebas a la vez creando un archivo `tests.txt` con el siguiente contenido:

```
id
id+id
id*id
id+id*id
(id)
(id+id)*id
id*(id+id)*id
(id+id+id)*(id*id)
+id
id+
id*
(id+)
(id+id
id)
((id+id)
```

Ejecutar:

```bash
python parser_asc.py < tests.txt
```

---

### **Casos válidos**

```
expr> id
ACEPTA
expr> id+id
ACEPTA
expr> id*id
ACEPTA
expr> id+id*id
ACEPTA
expr> (id)
ACEPTA
expr> (id+id)*id
ACEPTA
expr> id*(id+id)*id
ACEPTA
expr> (id+id+id)*(id*id)
ACEPTA
```

### **Casos inválidos**

```
expr> +id
RECHAZA
expr> id+
RECHAZA
expr> id*
RECHAZA
expr> (id+)
RECHAZA
expr> (id+id
RECHAZA
expr> id)
RECHAZA
expr> ((id+id)
RECHAZA
```

---

## **7. Conclusión**

El analizador ascendente implementado:

* Reconoce correctamente las expresiones definidas por la gramática original
* Controla la precedencia entre `+` y `*` y el uso de paréntesis
* Rechaza expresiones sintácticamente incorrectas
* Demuestra la aplicación completa del proceso: transformación LL(1), conjuntos FIRST/FOLLOW y diseño de parser ascendente con pila

---

# **Punto 4 — Comparación de Algoritmos de Análisis Sintáctico: CYK vs Predictivo**

---

## **1. Introducción**

Este proyecto compara dos algoritmos de análisis sintáctico:

* **Parser CYK:** Algoritmo ascendente que usa programación dinámica. Complejidad O(n³).
* **Parser Predictivo:** Algoritmo descendente recursivo. Complejidad O(n).

**Diferencias principales:**

* CYK es más general (maneja cualquier gramática libre de contexto).
* Predictivo es más rápido pero solo funciona con gramáticas LL(1).
* CYK usa una tabla n×n, mientras que el Predictivo usa recursión directa.

---

## **2. Ejecución**

**Requisitos:** Python 3.6+

**Pasos:**

1. Colocar los tres archivos en la misma carpeta:

   * `cyk_parser.py`
   * `predictive_parser.py`
   * `comparison.py`
2. Ejecutar:

   ```bash
   python comparison.py
   ```

El programa mostrará automáticamente las comparaciones de corrección y rendimiento.

---

## **3. Resultados de la comparación**

```
==================================================
Verificación de corrección:
id                   | CYK:   0    | Predictivo:   1
id+id                | CYK:   0    | Predictivo:   1
id*id                | CYK:   0    | Predictivo:   1
id+id*id             | CYK:   0    | Predictivo:   1
(id+id)*id           | CYK:   0    | Predictivo:   1
id*(id+id)*id        | CYK:   0    | Predictivo:   1
id+id+id*id+id       | CYK:   0    | Predictivo:   1

==================================================
Benchmarking (1000 iteraciones por caso):
Caso de prueba       | CYK (ms)   | Predictivo (ms) | Ratio
-----------------------------------------------------------------
id                   |   0.0007 |      0.0004 |   1.84x
id+id                |   0.0019 |      0.0007 |   2.56x
id*id                |   0.0019 |      0.0006 |   3.23x
id+id*id             |   0.0049 |      0.0008 |   5.79x
(id+id)*id           |   0.0077 |      0.0018 |   4.24x
id*(id+id)*id        |   0.0128 |      0.0014 |   8.99x
id+id+id*id+id       |   0.0138 |      0.0017 |   8.26x

==================================================
ANÁLISIS ESTADÍSTICO:
Tiempo promedio CYK: 0.0062 ms
Tiempo promedio Predictivo: 0.0011 ms
Speedup promedio: 5.85x
```

---

## **4. Conclusión**

El parser predictivo es aproximadamente seis veces más rápido en promedio que CYK para esta gramática, demostrando la ventaja de los algoritmos descendentes especializados sobre los métodos generales ascendentes.

---

# **Punto 5 — Algoritmo de Emparejamiento para Parser Descendente Recursivo**

---

## **1. Descripción**

Implementación de un parser predictivo descendente recursivo que utiliza un **algoritmo de emparejamiento** para verificar tokens en el análisis sintáctico.

---

## **2. Características**

* Parser predictivo descendente recursivo
* Algoritmo de emparejamiento integrado
* Entrada interactiva desde consola
* Gramática de expresiones aritméticas

---

## **3. Gramática implementada**

```
E  → T E'
E' → + T E' | ε
T  → F T' 
T' → * F T' | ε
F  → ( E ) | id
```

---

## **4. Uso del algoritmo de emparejamiento**

El algoritmo `match()` es fundamental y se usa en:

* `parse_E_prime()` → Empareja el token `+`
* `parse_T_prime()` → Empareja el token `*`
* `parse_F()` → Empareja tokens `(`, `)` e `id`

---

## **5. Ejecución**

```bash
python parser.py
```

---

## **6. Ejemplo de uso**

```
=== Parser Predictivo con Algoritmo de Emparejamiento ===
Gramática: E → E + T | T, T → T * F | F, F → (E) | id
Solo se acepta el literal 'id' como operando.
ENTER vacío para salir.
--------------------------------------------
expr> id
ACEPTA
expr> id+id
ACEPTA
expr> id*id
ACEPTA
expr> id+id*id
ACEPTA
expr> (id+id)*id
ACEPTA
expr> id+
RECHAZA
```

---

## **7. Archivos incluidos**

* `parser.py`: implementación del parser predictivo con algoritmo de emparejamiento

---


