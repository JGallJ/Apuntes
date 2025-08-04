Para evitar los percances con los alineamientos no deseados utlizaremos lab000 para la practica

##### Parte 1. Efecto del alineamiento de los vectores en memoria

2. Diferencias entre los ficheros axpy_align_v1() y axpy_align_v2()
    vmovups donde en el v1 pone vmovaps, lo que quiere decir que accede a direcciones no alineadas de memoria (la a significa alienamiento)
    
    alligned = a 
    unalingned = u


4. Observa en el siguiente enlace el ensamblador del bucle en axpy_align_v2() generado por las versiones
7.2 y 13.2 de gcc

el compilador hace la estrategia de peeling, donde hace un prologo de N elementos y un epílogo de M-N elementos siendo M el número de elementos totales de una linea de cache y N siendo donde comienza el alienamiento

0 *1 2 3 4 5 6 7 8 PROLOGO
9* 10 11 12 13 14 15
-16- EPILOGO


5. Comenta brevemente los tiempos de ejecución obtenidos de cada una de las versiones de axpy_align

                      Time	TPE
         Loop          ns      ps/el	  Checksum
     axpy_align_v1   118.8     116.0     8053064192.000000
     axpy_align_v2   241.7     236.1     8053064192.000000
axpy_align_v1_intr   119.9     117.1     8053064192.000000
axpy_align_v2_intr   239.1     233.5     8053064192.000000

El desalineamiento consume hasta el doble de tiempo, no recomendable
Intentar trabajar siempre con datos alineados


6. ¿Qué ocurre? ¿Cuál crees que es la causa?
Al intentar descomentar la linea axpy_align_v1_intru() en el programa principal nos da segmentation fault por intentar acceder a direcciones no alineadas

Se puede probar con gdb para conocer que instrucción a producido el error


##### Parte 2. Efecto del solapamiento de las variables en memoria


2. Busca en el actual estándar de C el significado de la palabra clave restrict y explica su efecto en esta
función.
Restrict -> los parámetros nos tienen solape, por lo que la versión v2 no presenta solapamiento mientras que la v1 si


3. Busca en la documentación de gcc el significado del citado pragma y explica su efecto en esta función.
#pragma GCC ivdep -> forma de indicar al compilador que es posible la vectorización
    si te equivocas puedes obetener resultados incorrectos y se produzca solapamiento


4. Busca en la documentación de gcc el significado de __builtin_assume_aligned() y explica su efecto en
esta función.

real *xx = __builtin_assume_aligned(vx, ARRAY_ALIGNMENT);
real *yy = __builtin_assume_aligned(vy, ARRAY_ALIGNMENT);
real *zz = __builtin_assume_aligned(vz, ARRAY_ALIGNMENT);

Forma de decir al compilar que no solapan y encima estan alineados


5. Comenta brevemente los tiempos de ejecución obtenidos. Relaciona los resultados con las características
de cada cada código ejecutado.

                     Time	TPE
             Loop     ns       ps/el     Checksum
     axpy_alias_v1  4508.5    4402.8     683.113220
     axpy_alias_v1   132.4     129.3     682.446289
     axpy_alias_v1   106.4     103.9     682.668701
     axpy_alias_v1   106.1     103.6     1024.000000
     axpy_alias_v2   210.1     205.2     1024.000000
     axpy_alias_v2   106.8     104.3     1024.000000
     axpy_alias_v3   209.9     205.0     1024.000000
     axpy_alias_v3   107.0     104.4     1024.000000
     axpy_alias_v4   106.9     104.4     1024.000000
              axpy   105.4     102.9     8053064192.000000

variables globales mas sencillas y mejores en ciertas situaciones para aumentar prestaciones


##### Parte 3. Efecto de los accesos no secuenciales (stride) a memoria

Vamos a acceder a 1 de cada 2 elementos
En la teoria deberia de dar los mismos resultados con stride1 y con stride2

//no se puede ejecutar en lab000 por que no tiene avx2 (pelotudo)
    //necesario cambiar la variable -mavx2 a -mavx
    
los resultados temporales son mejores los de escalar que los de vectoriales
    aceleracion negativa segun chuso

el codigo vectorial con stride intenta fundir en un mismo vector todos los elementos necesarios, haciendo operaciones de blend que le llevan a costar más tiempo que hacíendolo de manera escalar, aunque luego se usen operaciones vectoriales


##### Parte 4. Efecto de las sentencias condicionales en el cuerpo del bucle

Codigo condicional que deberia detener la vectorizacion o al menos ralentizarla, pero realmente se pueden vectorizar

1. ¿Ha vectorizado el bucle en cond_vec()?

Si se ha vectorizado

2. Analiza el fichero que contiene el ensamblador y echa un vistazo a las instrucciones correspondientes al bucle vectorizado. ¿Cuántas instrucciones vectoriales corresponden al cuerpo del bucle interno?
Detalla las operaciones realizadas por las instrucciones vectoriales del bucle.

Se compara vectorialmente y luego se realiza una opearicion de blend de 4 operandos donde se compacta en base a los resultados de la comparación vectorial




