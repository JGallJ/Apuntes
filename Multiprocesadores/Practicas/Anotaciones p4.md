# Parte 1, primera aproximacion

### Parallel
Compilo el programa con 
```
g++ -O3 -fopenmp parallel.cpp -o parallelomp.bin
```
o bien
```
./compomp.sh parallel
```

Cambio el numero de threads disponibles. En mi caso he usado el primero
```
$ export OMP_NUM_THREADS=2 # bash/ksh syntax
% setenv OMP_NUM_THREADS 2 # csh syntax
```

Pregunta, **¿Cuál es el número máximo de threads que soporta?**
Probamos metiendo 2, funciona
Probamos metiendo 128, funciona
Probamos metiendo 256, no funciona
Reducimos a 200, tampoco funciona
Disminuimos a 150, funciona
Aumentamos a 175, sigue funcionando
En 187 sigue funcionando
195 también corre
196 también corre, pero si aumentamos más ya no nos deja

Numero maximo de threads es de ***196***

### Nested

Modifico el numero de threads a 5
Añado lo siguiente al fichero
```
omp_set_dynamic(5);
```

Salida :
	Primera regi�n paralela: thread 0 de 2
	Regi�n anidada paralela (Equipo 0): thread 0 de 1
	Primera regi�n paralela: thread 1 de 2
	Regi�n anidada paralela (Equipo 1): thread 0 de 1

aplico cambios al numero maximo de niveles. En este caso lo pongo a 5
```
export OMP_MAX_ACTIVE_LEVELS=valor_natural # valor > 0
export OMP_NESTED=true|false
```

Salida :
	Primera regi�n paralela: thread 0 de 2
	Primera regi�n paralela: thread 1 de 2
	Regi�n anidada paralela (Equipo 0): thread 0 de 4
	Regi�n anidada paralela (Equipo 0): thread 1 de 4
	Regi�n anidada paralela (Equipo 0): thread 2 de 4
	Regi�n anidada paralela (Equipo 0): thread 3 de 4
	Regi�n anidada paralela (Equipo 1): thread 0 de 4
	Regi�n anidada paralela (Equipo 1): thread 2 de 4
	Regi�n anidada paralela (Equipo 1): thread 1 de 4
	Regi�n anidada paralela (Equipo 1): thread 3 de 4

Pregunta, **¿Cuál es el número máximo de threads que llegas a ver?**
Llego a ver el mismo número de antes, **196**, pero esta vez puedo ajustar el numero de threads entre los threads de la primera región y los threads de la región anidada
La suma total de estos no puede superar los 196. Se ha probado por ejemplo a ejectuar con 3 en región principal y 65 en anidada (3\*65 + 1 = 196) , siendo con 66 cuando ya deja de funcionar, y también con 2 en región principal y 97 en anidada (2\*97 + 1 = 195)

### Reduccion
Probamos con diferente numero de threads y vemos que sucede

Con 2:
Thread 0 I = 0 J = 0 K = 0
Thread 1 I = 1 J = 1 K = 1
Thread 0 I = 1 J = 0 K = 1

Con 4:
Thread 0 I = 0 J = 0 K = 0
Thread 2 I = 2 J = 2 K = 2
Thread 1 I = 1 J = 1 K = 1
Thread 3 I = 3 J = 3 K = 3
Thread 0 I = 6 J = 0 K = 3

Con 8:
Thread 0 I = 0 J = 0 K = 0
Thread 3 I = 3 J = 3 K = 3
Thread 1 I = 1 J = 1 K = 1Thread 5 I = 5 J = 5 K = 5Thread 6 I = 6 J = 6 K = 6
Thread 4 I = 4 J = 4 K = 4
Thread 2 I = 2 J = 2 K = 2
Thread 7 I = 7 J = 7 K = 7


Thread 0 I = 28 J = 0 K = 7

### Fordir

Ejecucione realizadas:
Threads = 2, DIM = 16:
	Thread-0 de 2 tiene N=2
	Thread-0 de 2 tiene N=3
	Thread-0 de 2 tiene N=4
	Thread-0 de 2 tiene N=5
	Thread-0 de 2 tiene N=10
	Thread-0 de 2 tiene N=11
	Thread-0 de 2 tiene N=12
	Thread-0 de 2 tiene N=13
	Thread-1 de 2 tiene N=6
	Thread-1 de 2 tiene N=7
	Thread-1 de 2 tiene N=8
	Thread-1 de 2 tiene N=9
	Thread-1 de 2 tiene N=14
	Thread-1 de 2 tiene N=15
	M�ximo gradiente: 55.947006 posicion: 0

Threads = 16, DIM = 16, static = 1:
	Thread-0 de 16 tiene N=2
	Thread-3 de 16 tiene N=5
	Thread-5 de 16 tiene N=7
	Thread-13 de 16 tiene N=15
	Thread-12 de 16 tiene N=14
	Thread-2 de 16 tiene N=4
	Thread-6 de 16 tiene N=8Thread-7 de 16 tiene N=9
	Thread-8 de 16 tiene N=10Thread-1 de 16 tiene N=3
	Thread-11 de 16 tiene N=13
	Thread-10 de 16 tiene N=12
	Thread-9 de 16 tiene N=11
	
	Thread-4 de 16 tiene N=6
	
	M�ximo gradiente: 85.404464 posicion: 0

Threads = 16, DIM = 128, static = 4:
	Thread-0 de 32 tiene N=2
	Thread-0 de 32 tiene N=3
	Thread-0 de 32 tiene N=4
	Thread-0 de 32 tiene N=5
	Thread-26 de 32 tiene N=106Thread-24 de 32 tiene N=98
	Thread-24 de 32 tiene N=99
	Thread-24 de 32 tiene N=100
	Thread-24 de 32 tiene N=101
	
	Thread-26 de 32 tiene N=107
	Thread-30 de 32 tiene N=122Thread-25 de 32 tiene N=102
	Thread-27 de 32 tiene N=110
	Thread-27 de 32 tiene N=111
	Thread-2 de 32 tiene N=10Thread-27 de 32 tiene N=112
	Thread-27 de 32 tiene N=113
	Thread-26 de 32 tiene N=108
	Thread-26 de 32 tiene N=109
	Thread-29 de 32 tiene N=118
	Thread-29 de 32 tiene N=119Thread-25 de 32 tiene N=103Thread-3 de 32 tiene N=14
	
	
	Thread-2 de 32 tiene N=11
	Thread-2 de 32 tiene N=12
	Thread-2 de 32 tiene N=13
	
	Thread-29 de 32 tiene N=120
	Thread-29 de 32 tiene N=121
	Thread-3 de 32 tiene N=15Thread-15 de 32 tiene N=62
	Thread-15 de 32 tiene N=63
	Thread-15 de 32 tiene N=64
	Thread-15 de 32 tiene N=65
	Thread-25 de 32 tiene N=104
	Thread-25 de 32 tiene N=105
	
	Thread-30 de 32 tiene N=123Thread-28 de 32 tiene N=114
	
	Thread-3 de 32 tiene N=16
	Thread-3 de 32 tiene N=17
	Thread-1 de 32 tiene N=6
	Thread-1 de 32 tiene N=7
	Thread-1 de 32 tiene N=8
	Thread-1 de 32 tiene N=9
	
	Thread-28 de 32 tiene N=115Thread-30 de 32 tiene N=124
	Thread-30 de 32 tiene N=125
	Thread-7 de 32 tiene N=30
	Thread-8 de 32 tiene N=34Thread-7 de 32 tiene N=31
	Thread-5 de 32 tiene N=22Thread-4 de 32 tiene N=18
	Thread-4 de 32 tiene N=19Thread-14 de 32 tiene N=58
	
	Thread-10 de 32 tiene N=42
	Thread-6 de 32 tiene N=26Thread-10 de 32 tiene N=43
	Thread-6 de 32 tiene N=27Thread-4 de 32 tiene N=20
	Thread-4 de 32 tiene N=21
	
	Thread-10 de 32 tiene N=44
	Thread-10 de 32 tiene N=45
	
	Thread-12 de 32 tiene N=50
	Thread-12 de 32 tiene N=51
	Thread-22 de 32 tiene N=90Thread-12 de 32 tiene N=52
	Thread-12 de 32 tiene N=53
	Thread-6 de 32 tiene N=28
	Thread-6 de 32 tiene N=29
	
	Thread-22 de 32 tiene N=91
	Thread-22 de 32 tiene N=92
	Thread-22 de 32 tiene N=93
	
	Thread-14 de 32 tiene N=59
	Thread-14 de 32 tiene N=60
	Thread-14 de 32 tiene N=61
	
	Thread-5 de 32 tiene N=23
	Thread-28 de 32 tiene N=116
	Thread-28 de 32 tiene N=117
	Thread-19 de 32 tiene N=78
	Thread-19 de 32 tiene N=79
	Thread-19 de 32 tiene N=80
	Thread-19 de 32 tiene N=81
	Thread-17 de 32 tiene N=70
	Thread-21 de 32 tiene N=86
	Thread-21 de 32 tiene N=87
	Thread-21 de 32 tiene N=88
	Thread-31 de 32 tiene N=126
	Thread-31 de 32 tiene N=127
	
	Thread-5 de 32 tiene N=24
	Thread-5 de 32 tiene N=25
	Thread-17 de 32 tiene N=71
	Thread-17 de 32 tiene N=72
	Thread-17 de 32 tiene N=73
	Thread-18 de 32 tiene N=74
	Thread-18 de 32 tiene N=75
	Thread-18 de 32 tiene N=76
	Thread-18 de 32 tiene N=77
	Thread-16 de 32 tiene N=66
	Thread-16 de 32 tiene N=67
	Thread-16 de 32 tiene N=68
	Thread-16 de 32 tiene N=69
	Thread-23 de 32 tiene N=94
	Thread-23 de 32 tiene N=95
	Thread-23 de 32 tiene N=96
	Thread-23 de 32 tiene N=97
	Thread-7 de 32 tiene N=32
	Thread-7 de 32 tiene N=33Thread-13 de 32 tiene N=54
	Thread-11 de 32 tiene N=46Thread-20 de 32 tiene N=82Thread-21 de 32 tiene N=89
	
	Thread-11 de 32 tiene N=47
	Thread-11 de 32 tiene N=48
	Thread-11 de 32 tiene N=49
	
	Thread-13 de 32 tiene N=55
	Thread-13 de 32 tiene N=56
	Thread-13 de 32 tiene N=57
	Thread-9 de 32 tiene N=38
	Thread-9 de 32 tiene N=39
	Thread-9 de 32 tiene N=40
	Thread-9 de 32 tiene N=41
	
	Thread-20 de 32 tiene N=83
	Thread-20 de 32 tiene N=84
	Thread-20 de 32 tiene N=85
	Thread-8 de 32 tiene N=35
	Thread-8 de 32 tiene N=36
	Thread-8 de 32 tiene N=37
	M�ximo gradiente: 66.527105 posicion: 0

### Parallel for

utilizar el comando time para medir tiempos de la siguiente manera
```
time <comando a monitorizar>
```

**Con 2 threads:**
Entorno de Ejecuci�n:
 - M�ximo n� de threads disponibles (omp_get_max_threads): 2
 - N� de procesadores disponibles (omp_get_num_procs): 96
 - Threads utilizados (omp_get_num_threads): 2
Chrono::high_resolution_clock, tiempo transcurrido: 22.5814
Time, tiempo transcurrido: 44.5867

real	0m24.269s
user	0m44.425s
sys	0m1.849s

**Con 4 threads:**
Entorno de Ejecuci�n:
 - M�ximo n� de threads disponibles (omp_get_max_threads): 4
 - N� de procesadores disponibles (omp_get_num_procs): 96
 - Threads utilizados (omp_get_num_threads): 4
Chrono::high_resolution_clock, tiempo transcurrido: 11.4731
Time, tiempo transcurrido: 45.6361

real	0m13.508s
user	0m44.925s
sys	0m2.751s

**Con 8 threads:**
Entorno de Ejecuci�n:
 - M�ximo n� de threads disponibles (omp_get_max_threads): 8
 - N� de procesadores disponibles (omp_get_num_procs): 96
 - Threads utilizados (omp_get_num_threads): 8
Chrono::high_resolution_clock, tiempo transcurrido: 6.77136
Time, tiempo transcurrido: 52.0249

real	0m8.710s
user	0m44.829s
sys	0m9.138s

**Con 16 threads:**
Entorno de Ejecuci�n:
 - M�ximo n� de threads disponibles (omp_get_max_threads): 16
 - N� de procesadores disponibles (omp_get_num_procs): 96
 - Threads utilizados (omp_get_num_threads): 16
Chrono::high_resolution_clock, tiempo transcurrido: 5.08818
Time, tiempo transcurrido: 78.4761

real	0m6.646s
user	0m45.676s
sys	0m34.370s

### Sections

**Con 2 threads:**
Esta es la secci�n 1 ejecutada por el thread 0
Esta es la secci�n 2 ejecutada por el thread 1
Esta es la secci�n 4 ejecutada por el thread 1
Esta es la secci�n 3 ejecutada por el thread 0

**Con 4 threads:**
Esta es la secci�n 4 ejecutada por el thread 3
Esta es la secci�n 1 ejecutada por el thread 0
Esta es la secci�n 3 ejecutada por el thread 2
Esta es la secci�n 2 ejecutada por el thread 1

**Con 8 threads:**
Esta es la secci�n 4 ejecutada por el thread 2
Esta es la secci�n 1 ejecutada por el thread 0
Esta es la secci�n 3 ejecutada por el thread 3
Esta es la secci�n 2 ejecutada por el thread 5

### Single

**Con 4 threads:**
Resultado de la suma es: 3.6e+07
D[0]=2.000000
D[1]=2.000000
D[2]=2.000000
D[3]=2.000000
D[4]=2.000000
D[5]=2.000000
D[6]=2.000000
D[7]=2.000000
D[8]=2.000000
D[9]=2.000000
D[10]=2.000000
D[11]=2.000000

### Master
Solo se ejecuta el código por el thread master de un equpo

### Barrier

**Con 4 threads:**
Escribe un valor:
10
Mi numero de thread es: 0
Numero de threads: 4
Valor de L es: 10
Mi numero de thread es: 3
Numero de threads: 4
Valor de L es: 10
Mi numero de thread es: 2
Numero de threads: 4
Valor de L es: 10
Mi numero de thread es: 1
Numero de threads: 4
Valor de L es: 10

### Flush
Marca un punto de sincronización donde se requiere una visión consistente de memoria

### Atomic

Con atomic:
0 - 0
11065 - 11065
11145 - 11145
11131 - 11131
11070 - 11070
11080 - 11080
11093 - 11093
11098 - 11098
11186 - 11186
11132 - 11132

Sin atomic:
0 - 0
3703 - 11162 ERROR
3727 - 11168 ERROR
3807 - 11190 ERROR
3547 - 11128 ERROR
3702 - 11087 ERROR
3758 - 11171 ERROR
3786 - 10941 ERROR
3649 - 11146 ERROR
3657 - 11007 ERROR

### Ordered
Solo puede aparecer de mano de una directiva for o parallel for

# Parte 2, ejercicios

Ejercicio 2a

```

```

Ejercicio 3b
En este caso, la variable t es un puntero, por lo que al intentar provatizar esta variable nos porduce un error de segmentation core dumped
La mejor forma que se me ocurre es hacer esta una variable local temporal como en el ejemplo anterior

Ejercicio 4a
El bucle no se puede paralelizar ya que presenta una dependencia productor consumidor a distancia 1

Ejercicio 5
El bucle principal no puede paralelizarse debido a una dependencia productor consumidor a distancia 1
Con poner ``` #pragma omp parallel for reduction(+:q)``` esto ya se soluciona

# Para practica 5

\#pragma omp task
mirar en documentacion


Experimentación con escalabilidad del codigo a comentar en la memoria