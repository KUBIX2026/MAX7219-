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
![Snake](../assets/Snake.png)


## Batalla naval

Se realizó un diagrama que define las acciones que realiza nuestro módulo desde la preparación hasta el fin del juego. El objetivo es que la matriz pueda mostrar el histórico de lanzamientos de cada jugador, para que no repitan jugadas erróneas.


```mermaid

flowchart TD
    A(["Inicializar juego:<br/>NAVAL BATTLE"])
    B["PREPARAR NAVES<br/>(esperar que cada<br/>jugador ubique sus<br/>barcos)"]
    C{"Jugadores<br/>listos?"}

    D["Turno P1"]
    E{"Impactó algun<br/>objetivo"}
    F["Registrar<br/>daño en p2"]
    G{"Verificar<br/>si HP2=0"}
    H["Finalizar<br/>juego,<br/>ganador P1"]

    I["Turno P2"]
    J{"Impactó algun<br/>objetivo"}
    K["Registrar<br/>daño en p1"]
    L{"Verificar<br/>si HP1=0"}
    M["Finalizar<br/>juego,<br/>ganador P2"]

    N(["JUEGO TERMINADO.<br/>MOSTRAR PUNTAJES DE LA<br/>PARTIDA"])

    A --> B
    B --> C
    C -->|No| B
    C -->|Sí| D

    D --> E
    E -->|Sí| F
    F --> G
    G -->|Sí| H
    G -->|No| D

    E -->|No| I
    I --> J
    J -->|Sí| K
    K --> L
    L -->|Sí| M
    L -->|No| I

    J -->|No| D

    H --> N
    M --> N
```
![BatallaNaval](../assets/BatallaNaval.png)


## Galaga

Se realizó un diagrama en el que la matriz muestra la cantidad de naves (vidas) de cada jugador y el stage en el que se encuentran. Se supone que el juego es infinito y solo termina cuando el jugador muere, es decir, cuando es impactado 3 veces.

```mermaid

flowchart TD
    A(["Iniciar juego:<br/>Galaga"])

    B["Seleccionar<br/>nave(s)"]

    C{"Jugador(es)<br/>listo(s)?"}

    D["Iniciar juego,<br/>3 vidas por<br/>jugador"]

    E{"¿Jugador es<br/>impactado una<br/>vez?"}

    F["Mostrar vida en la<br/>pantalla -1"]

    G{"Vida<br/>total = 0?"}

    H["GAME OVER"]

    I["Continuar<br/>juego."]

    J{"¿El jugador<br/>consiguió avanzar<br/>de nivel?"}

    K["Mostrar lvl<br/>'x' en<br/>pantalla +<br/>cantidad de<br/>vidas del<br/>jugador"]

    A --> B
    B --> C

    C -->|No| B
    C -->|Sí| D

    D --> E

    E -->|Sí| F
    F --> G

    G -->|No| E
    G -->|Sí| H

    E -->|No| I
    I --> J

    J -->|Sí| K
    K --> E
```
![Galaga](../assets/Galaga.png)


##Breakout


```mermaid


flowchart TD
    A(["Iniciar juego: Breakout"])
    B["Seleccionar modo de juego<br/>1 o 2 jugadores"]
    C["Inicializar cada jugador<br/>3 vidas, puntaje 0, nivel 1<br/>Muro independiente"]
    D["Seleccionar jugador activo"]
    E{"¿Jugador activo<br/>tiene vidas?"}
    F["Mostrar vidas en MAX7219<br/>Preparar pelota sobre paleta"]
    G{"¿Jugador lanzó<br/>la pelota?"}
    H["Esperar lanzamiento"]
    I["Pelota en movimiento"]
    J{"¿Pelota cayó?"}
    K["Restar una vida al<br/>jugador activo"]
    L["Actualizar corazones<br/>en MAX7219"]
    M{"¿Quedan vidas<br/>al jugador?"}
    N{"¿Hay otro jugador<br/>con vidas?"}
    O["Cambiar al otro jugador<br/>Restaurar su muro y puntaje"]
    P{"¿Se rompió<br/>un bloque?"}
    Q["Sumar puntos según fila<br/>Aumentar velocidad"]
    R{"¿Muro vacío?"}
    S["Mostrar LEVEL n<br/>Preparar siguiente muro"]
    T["Incrementar nivel<br/>y velocidad inicial"]
    U["Mostrar LOSE en MAX7219"]
    V["Mostrar puntajes finales"]
    W(["Fin"])
    X{"¿Queda algún<br/>jugador con vidas?"}

    A --> B --> C --> D --> E

    E -->|Sí| F
    E -->|No| X
    X -->|Sí| D
    X -->|No| U

    F --> G
    G -->|No| H
    H --> G
    G -->|Sí| I

    I --> J

    J -->|No| P
    P -->|No| I
    P -->|Sí| Q
    Q --> R
    R -->|No| I
    R -->|Sí| S
    S --> T
    T --> F

    J -->|Sí| K
    K --> L
    L --> M

    M -->|Sí| N
    M -->|No| X

    N -->|Sí| O
    N -->|No| F
    O --> E

    U --> V --> W
```
![Breakout](../assets/Breakout.png)

