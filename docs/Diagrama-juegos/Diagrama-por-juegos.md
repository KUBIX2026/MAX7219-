# Diagramas del uso de la matriz MAX7219 en los juegos

Se realizaron diagramas de flujo con ideas para usar la matriz de LEDs MAX7219 (4 módulos de 8×8 en cascada, 32×8 píxeles) como pantalla auxiliar durante los juegos. Los puntajes no se muestran en esta matriz porque ya los cubre otra pantalla.

## Pong

La matriz muestra una flecha que apunta al jugador que debe golpear la pelota. Se redibuja solo cuando cambia el turno: en cada saque y cada vez que una paleta devuelve la pelota.

```mermaid
flowchart TD
    A["Inicializar juego Pong"]
    B["El juego empieza con la pelota al centro"]
    C{"¿Turno?"}
    D["bitmap = flecha derecha"]
    E["bitmap = flecha izquierda"]
    F[La pelota esta en juego, la flecha queda fija]
    G{¿Que ocurre?}
    H[Mostrar Gol]
    I[Invertir turno, IZQ pasa a DER y viceversa]
    J[Fin de partida Apagar matriz   Registros 0x01 a 0x08 = 0x00 en los 4 modulos]
    K[Fin]

    A --> B
    B --> C
    C -->|Jugador 1| D
    C -->|Jugador 2| E
    D-->F
    E-->F
    F-->G
    G-->|La pelota pasa por un lado|H
    G-->|La paleta golpea la pelota|I
    I-->F
    H-->J-->K
```
![Pong](../assets/Pong.png)

## Snake

La matriz se divide en cuatro módulos, de izquierda a derecha: manzana fija, manzanas recogidas, serpiente fija y longitud de la serpiente.

```mermaid

flowchart TD
    A(["Inicializar juego: Snake"])

    B["Inicializar partida
    manzanas = 0
    longitud = 3
    Serpiente y primera manzana en el tablero"]

    C["Cargar iconos fijos
    Módulo 1 = bitmap MANZANA
    Módulo 3 = bitmap SERPIENTE"]

    D["Separar dígitos
    manzanas: decenas y unidades
    longitud: decenas y unidades"]

    E["Armar bitmap de cada contador
    Fuente 3x5 por dígito
    Módulo 2 = manzanas, Módulo 4 = longitud"]

    F["Serpiente en movimiento
    Leer control NES, teclado o mouse
    Avanzar la cabeza una casilla"]

    G{"¿La serpiente avanzó?"}

    H["longitud = longitud + 1
    Si llegó a una manzana: manzanas = manzanas + 1"]

    I["Actualizar solo módulos 2 y 4
    Enviar no-op 0x0000 a módulos 1 y 3"]

    J["Fin de partida
    Apagar matriz
    Registros 0x01 a 0x08 = 0x00 en los 4 módulos"]

    K(["Fin"])

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G

    G -->|Nada| F
    G -->|Sí| H
    H --> I
    I --> F

    G -->|Choque con pared o con el cuerpo| J
    J --> K

```


## Batalla naval

Se realizó un diagrama que define las acciones que realiza nuestro módulo desde la preparación hasta el fin del juego. El objetivo es que la matriz pueda mostrar el histórico de lanzamientos de cada jugador, para que no repitan jugadas erróneas.


![Diagrama_batalla_naval](Diagrama_naval.jpeg)


## Galaga

Se realizó un diagrama en el que la matriz muestra la cantidad de naves (vidas) de cada jugador y el stage en el que se encuentran. Se supone que el juego es infinito y solo termina cuando el jugador muere, es decir, cuando es impactado 3 veces.

![Diagrama_galaga](Diagrama_galaga.jpg)
