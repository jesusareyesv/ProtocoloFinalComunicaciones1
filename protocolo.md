# COMUNICACIONES I – PROYECTO PARCIAL II

## REGLAS


1. Formas de iniciar el juego en la primera ronda (por jerarquía):
    * Tener el doble 6.
    * Tener el doble más alto.
    * Tener la ficha con mayor pinta.
    * Al azar.
2. El juego es individual.
3. El ‘sentido’ del juego será en el orden de conexión de los jugadores.
4. Las rondas serán de 2 a 4 jugadores.
5. Son 7 fichas por jugador.
7. Si el juego inicia con menos de 4 jugadores, el resto de fichas se descarta.
8. Si el jugador que tiene la mano pasa (en algún momento de la ronda), ahora la mano la tendría el siguiente jugador, siguiendo el sentido del juego.
9. Si no tiene fichas apropiadas para jugar, debe pasar.
10. En caso de una jugada errónea, el jugador es sacado de la partida y su mano se descarta.
11. No hay pozo.
12. Si un jugador se desconecta, se retira del juego.
13. No se permite reconexión.
14. La mano del jugador que se desconecta se descarta.
15. Una partida se gana cuando alguno de los jugadores llega a 100 puntos (independientemente el número de rondas).
16. Formas de ganar una ronda:
    * Dominó la ronda (se le acaben las fichas).
    * Que el juego se tranque y tenga la mejor puntuación, esto es, el menor número de pintas.
    * Ser el último jugador en la mesa (por desconexión del resto de jugadores).
17. Formas de desempate con el juego trancado (por jerarquía):
    * Gana el jugador que trancó.
    * Gana el jugador que tenga la mano (o el más cercano a él, siguiendo el sentido del juego).
    * El puntaje del ganador en una ronda será la suma de las pintas de los demás jugadores.

## PROTOCOLO
### Contemplaciones:
- El servidor es quien genera las fichas.
- El servidor y el cliente llevan los puntajes generales internamente y solo se intercambia lo establecido en el protocolo.
- Las conexiones unicast serán TCP.

Durante las comunicaciones del juego se enviarán exclusivamente objetos de tipo JSON.

Cada objeto de tendrá en su body una clave identificador con el valor DOMINOCOMUNICACIONESI. Ejemplo:
```json
{	
    "identificador": "DOMINOCOMUNICACIONESI"
}
```

El servidor inicia una mesa escuchando por el puerto 3001.

El cliente se inicia y envía un broadcast para conocer las mesas disponibles. Ejemplo:
```json
{ 	
    "identificador": "DOMINOCOMUNICACIONESI"
}
```

El servidor responde (si y solo si, hay por lo menos un espacio disponible) al broadcast del cliente con un mensaje unicast UDP para darle a saber que hay una mesa disponible. Ejemplo:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
    	"nombre_mesa": "Nombre de la mesa definido por el servidor"
}
```

El cliente, luego de conocer las mesas disponibles, selecciona la mesa a la que se quiere conectar enviándole un mensaje unicast TCP al servidor. Ejemplo:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
    	"nombre_jugador": "Nombre del jugador definido por el cliente"
}
```






El servidor responde al unicast del cliente con un mensaje para confirmar la conexión. Ejemplo:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"multicast_ip": "IP multicast definida por el servidor",
	"disponible": "Verdadero o falso. Esto es un booleano"
}
```

*Nota:* El servidor espera 30 segundos, a partir del ingreso del segundo jugador, y reiniciando dicho conteo cada vez que entra alguien. Y el juego se inicia cuando el conteo es excedido o cuando la mesa se llena

Al iniciar la partida, el servidor envía las fichas a cada jugador mediante unicast. Ejemplo:
```json
	{
	"identificador": "DOMINOCOMUNICACIONESI",
    "jugadores": [
        "identificador_jugador_1",
        "identificador_jugador_2",
        "identificador_jugador_3",
        "identificador_jugador_4"
    ],
    "identificador" : "Identificador que la mesa genera para el jugador al que le envia estas fichas.",
    "fichas": [
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 1
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 2
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 3
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 4
        }
    ]
}
```

Además de las fichas, también le envía a cada jugador su identificador asociado a través del campo ```identificador``` y los identificadores de todos los jugadores conectados a la mesa usando un array de strings almacenados en el campo ```jugadores```.

*Nota:* el array de fichas siempre debe contener 7 fichas para que cada jugador pueda comenzar la partida. Además, se envía un token único con la finalidad de que las jugadas de los clientes sean mediante el envío de dicho token y no con los valores. El servidor será quien valide estas condiciones


El servidor envía un mensaje multicast justo antes de comenzar un turno. Los mensajes del servidor se podrán identificar por el campo tipo

0: mensaje normal de juego
1: mensaje de fin de ronda
2: mensaje de fin de partida
3: mensaje de desconexión
 
… También, cuando sea un mensaje normal de juego, habrá un campo que se llamará evento_pasado, dentro del mismo estará la información de la jugada anterior, dicha jugada se podrá identificar con su tipo

0: jugada normal
1: jugada errónea
2: pasó

Ejemplo:

Mensaje de juego:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que debe iniciar o que tiene el turno",
	"tipo": 0,
	"punta_uno": "Número inicial de la lista o -1",
    "punta_dos": "Número final de la lista o -1",
    "evento_pasado": {
            "tipo": 0,
            "jugador": "Identificador de quien jugó",
    	    "ficha": { 
		        "entero_uno": 6, 
    		    "entero_dos": 6
            },
    		"punta": "true //Booleano indicando el lugar de juego. True: punta uno. False: punta dos "
    	} //Este campo no existirá en el tipo de evento 2
    } 
}
```

*Nota:* si es el inicio de la ronda punta_uno y punta_dos serán -1

Mensaje de fin de ronda:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que ganó la ronda",
	"tipo": 1,
	"puntuacion": "Puntuación del ganador de la ronda",
	"razon": "Descripción del fin de la ronda. Máximo 45 caracteres"
}
```

Mensaje de fin de partida:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que ganó la partida",
	"tipo": 2,
	//Puntuación de los jugadores
	"puntuacion_general": [
        {
            "jugador": "Identificador del jugador",
            "puntuacion": "Puntuación del jugador"
        }
    ],
    "razon": "Descripción del fin de la partida. Máximo 45 caracteres"
}
```

Mensaje de desconexión:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que se desconectó",
	"tipo": 3
}
```


Cada jugador justo después del mensaje de juego enviará un mensaje unicast con la información de su jugada, esto si está en turno, si no, un mensaje para confirmar que sigue en línea. Ejemplo:

Jugador en turno y jugando:

```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"ficha": { 
        "token": "estoesuntoken"  
        },
	"punta": "Booleano indicando el lugar de juego. True: punta uno. False: punta dos"
}
```

*Nota:* Si el token es -1, significa que el jugador quiere pasar.

Jugador que no está en turno (mensaje unicast):

```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador"
}
```

*Nota:* el servidor espera este mensaje durante 10 segundos, si no hay respuesta asume que el cliente se desconectó. También, este mensaje será igual para cada turno.
