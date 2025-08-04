## 3.1 Suma mediante reducción

**Escribe en C++ utilizando `std::thread` el código para realizar la suma de un vector de elementos sin emplear `std::atomic` ni `std::mutex`.**

¿Será correcto el resultado del código anterior? Explica el motivo.
No, ya que si hacemos la suma de las porciones en una variable compartida dentro de cada thread, la escritura se puede solapar entre dos o más threads.

**Dado el código `ej1-alumono.cpp` Analízalo y arréglalo para que sea correcto utilizando funciones de biblioteca. Prueba su correción. Ten en cuenta las opciones de compilación: `-lpthread` y `-mcmodel=large`**

Se han añadido los mutex para un acceso concurrente a las variables *indice_trabajo* y *sumatorio_concurrente* y se ha probado que funcione en central.cps.unizar.es

## 3.2 Suma mediante reducción con cuatro hilos y operaciones atómicas

**Utilizad el código ensamblador escrito para corregir el ejercicio de la sección 3.1 del guión. Probad la corrección de la implementación. ¿Es correcta la suma entre varios hilos con es nuevo código?**

Si lo haces con registros xN si, pero si intentas hacerlo con los de wN (de 32 bits) el resultado será erroneo

**¿Cuál sería la diferencia utilizando las instrucciones `ldxr/stxr`?**

Mientras que ldaxr/stlxr garantizan un orden correcto en la visibilidad de las operaciones de memoria ademas de atomicidad, ldxr/stxr solo garantizan atomicidad sin garantizar orden en la visibilidad de las operaciones

## 4.1 Mutex con spin-lock

**Implementad vuestra versión de mutex en ensamblador y utilizarla en el programa de la sección 3.1. Probad la corrección del programa.**

Hecho

**¿Qué sucede cuando un thread intenta entrar en la sección crítica, pero ésta está ocupada? Identifica la secuencia de instrucciones que se ejecuta en ese caso.**

Lo que sucede es que se queda en el siguiente bucle dando vueltas hasta que en algun momento el valor de la variable x0 cambie de 1 a 0 y asi detectar que esta libre

```
loop:
    ldaxr       w1, [x0]
    cmp         w1, #0
    b.ne        loop
```

## 4.2 Spin-lock energéticamente eficiente (Optativo)

**Identifica dónde entra en modo de ahorro de energía el procesador.**

Es la instruccion `wfe`, o wait for event, la cual pone al procesador en bajo consumo hasta que le llegue un evento y este se despierte.
La primera vez que entra en el bucle este `wfe` recibira un evento de `sevl` para que simpre se ejecute una primera iteracion del bucle de `wfe`

**¿Se entera el Sistema Operativo de que el procesador está en modo bajo consumo?**

Como es un cambio transparente al sistema operativo, el procesador no se deberia de enterar, ya que no se produce ni una interrupcion ni un cambio de contexto, solamente accedemos a un modo de menor consumo en el que no tenemos aun trabajo que realizar (hasta que llegue un evento claro)

## Códigos de los programas usados

### ej1-alumno.cpp

```
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

using namespace std;

const long long n_elementos = 1000000000;
const unsigned int n_threads = 40;
const unsigned int trozos = 100; // de cuanto son los trozo de trabajo que cojo. trozo -> #elementos de v_elems

std::vector<std::thread> v_threads; //vector donde salvaguardo los threads que voy creando
std::mutex mutex_biblioteca1; // mutex para actualizar coger trozo de trabajo con garantias
std::mutex mutex_biblioteca2; // mutex para actualizar sumatorio_concurrente con garantias
long long indice_trabajo = 0; //me indica el punto de procesamiento -> ultimo elemento asignado a thread

int v_elems[n_elementos]; //vector que almacena los elementos que deben ser sumados

long long sumatorio_concurrente = 0; //variable compartida con el sumatorio de los valores de v_elems

void funcion(int th){
        long long mi_suma = 0;
	while (true){

		long long mi_indice_trabajo_ini;
		long long mi_indice_trabajo_fin;		

		// Leemos la variable indice_trabajo en exclusion mutua
		{
			std::lock_guard<std::mutex> lock(mutex_biblioteca1);
			if (indice_trabajo >= n_elementos) break;
			mi_indice_trabajo_ini = indice_trabajo;
			indice_trabajo += trozos; 
			mi_indice_trabajo_fin = std::min(indice_trabajo, n_elementos);
		} // Al salir del scope se produce un mutex.unlock() de manera automatica 
		
		for (long long i = mi_indice_trabajo_ini; i < mi_indice_trabajo_fin; i++){

			mi_suma = mi_suma + v_elems[i];
		}

		//cout << "Soy el thread: " << th << " acabo de realizar el trozo: " << mi_indice_trabajo_ini ;
		//cout << " - " << mi_indice_trabajo_fin << endl; 
	}

	// Lo mismo que antes pero con sumatorio_concurrente
	{
		std::lock_guard<std::mutex> lock(mutex_biblioteca2);
		sumatorio_concurrente += mi_suma;
	}

}

int main(){

	long long sumatorio = 0;

	//inicialización del vector
	for (int i = 0; i < n_elementos; i ++){
		v_elems[i] = i;
	}

	//calculo del sumatorio secuencial
	for (int i = 0; i < n_elementos; i ++){
		sumatorio = sumatorio + v_elems[i];
	}
	

	for (int i = 0; i < n_threads; i++){
		std::thread th(funcion, i);
		v_threads.push_back(std::move(th));
	}

	for (std::thread & th : v_threads){

		if (th.joinable())
			th.join();
	}

	cout << "El valor de la componente 200000 es: " << v_elems[200000] << endl;
	cout << "El resultado de la operación sumatorio es : " << to_string(sumatorio) << endl;
	cout << "El resu de la oper sumatorio concurrente es : " << to_string(sumatorio_concurrente) << endl;
	
	return 0;

}
```

## ej2-alumno.cpp

```
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

using namespace std;

// Declaración de la función en ensamablador en la que implementaremos el fetch and add
extern "C" long long mi_fetch_and_add(long long *x, long long add);

const long long n_elementos = 100000000;
const unsigned int n_threads = 40;
const unsigned int trozos = 100; // de cuanto son los trozo de trabajo que cojo. trozo -> #elementos de v_elems

std::vector<std::thread> v_threads; //vector donde salvaguardo los threads que voy creando
std::mutex mutex_biblioteca1; // mutex para actualizar coger trozo de trabajo con garantias
std::mutex mutex_biblioteca2; // mutex para actualizar sumatorio_concurrente con garantias
long long indice_trabajo = 0; //me indica el punto de procesamiento -> ultimo elemento asignado a thread

//int v_elems[n_elementos]; //vector que almacena los elementos que deben ser sumados
std::vector<int> v_elems(n_elementos);

long long sumatorio_concurrente = 0; //variable compartida con el sumatorio de los valores de v_elems

void funcion(int th){
        long long mi_suma = 0;
	while (true){

		long long mi_indice_trabajo_ini;
		long long mi_indice_trabajo_fin;

		// Leemos y sumamos la variable para despues usarla 
		mi_indice_trabajo_ini = mi_fetch_and_add(&indice_trabajo, trozos);
		
		if (mi_indice_trabajo_ini >= n_elementos) break;

		mi_indice_trabajo_fin = std::min((long long)(mi_indice_trabajo_ini + trozos), n_elementos);

		for (unsigned int i = mi_indice_trabajo_ini; i < mi_indice_trabajo_fin; i++){

			mi_suma = mi_suma + v_elems[i];
		}

		//cout << "Soy el thread: " << th << " acabo de realizar el trozo: " << mi_indice_trabajo_ini ;
		//cout << " - " << mi_indice_trabajo_fin << endl; 
	}

	mi_fetch_and_add(&sumatorio_concurrente, mi_suma);
}

int main(){

	long long sumatorio = 0;

	//inicialización del vector
	for (int i = 0; i < n_elementos; i ++){
		v_elems[i] = i;
	}

	//calculo del sumatorio
	for (int i = 0; i < n_elementos; i ++){
		sumatorio = sumatorio + v_elems[i];
	}
	
	cout << "El valor de la componente 200000 es: " << v_elems[200000] << endl;
	cout << "El resultado de la operación sumatorio es : " << to_string(sumatorio) << endl;


	for (int i = 0; i < n_threads; i++){
		std::thread th(funcion, i);
		v_threads.push_back(std::move(th));
	}

	for (std::thread & th : v_threads){

		if (th.joinable())
			th.join();
	}

	cout << "El valor de la componente 200000 es: " << v_elems[200000] << endl;
	cout << "El resu de la oper sumatorio concurrente es : " << to_string(sumatorio_concurrente) << endl;
	
	return 0;

}
```

### mi_fa-alumno.s

```

    .arch armv8-a
    .text
    .global mi_fetch_and_add
    .type mi_fetch_and_add, %function

mi_fetch_and_add:
   // x0 -> direccion en memoria
   // x1 -> valor a sumar

_loop:
   ldaxr    x2, [x0]
   add      x3, x2, x1     // Sumamos el valor de la direccion de memoria y el valor a sumar
   stlxr    w4, x3, [x0]   // Escribimos valor en x0
   cbnz     w4, _loop       // Intentar de nuevo si falla
   
   mov      x0, x2
   ret

```

## ej3-alumno.cpp

```
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

using namespace std;

extern "C" void mi_mutex_lock(int* mutex);
extern "C" void mi_mutex_unlock(int* mutex);

const long long n_elementos = 10000000;
const unsigned int n_threads = 40;
const unsigned int trozos = 100; // de cuanto son los trozo de trabajo que cojo. trozo -> #elementos de v_elems

std::vector<std::thread> v_threads; //vector donde salvaguardo los threads que voy creando
// 0 -> libre, 1 -> ocupado
int mutex1 = 0;	// mutex para actualizar coger trozo de trabajo con garantias
int mutex2 = 0; // mutex para actualizar sumatorio_concurrente con garantias
long long indice_trabajo = 0; //me indica el punto de procesamiento -> ultimo elemento asignado a thread

int v_elems[n_elementos]; //vector que almacena los elementos que deben ser sumados

long long sumatorio_concurrente = 0; //variable compartida con el sumatorio de los valores de v_elems

void funcion(int th){
        long long mi_suma = 0;
	while (true){

		long long mi_indice_trabajo_ini;
		long long mi_indice_trabajo_fin;		

		mi_mutex_lock(&mutex1);
		if (indice_trabajo >= n_elementos) {
			mi_mutex_unlock(&mutex1);
			break; 
		}
		mi_indice_trabajo_ini = indice_trabajo;
		indice_trabajo += trozos; 
		mi_indice_trabajo_fin = std::min(indice_trabajo, n_elementos);
		mi_mutex_unlock(&mutex1);
		
		for (long long i = mi_indice_trabajo_ini; i < mi_indice_trabajo_fin; i++){

			mi_suma = mi_suma + v_elems[i];
		}

		//cout << "Soy el thread: " << th << " acabo de realizar el trozo: " << mi_indice_trabajo_ini ;
		//cout << " - " << mi_indice_trabajo_fin << endl; 
	}

	mi_mutex_lock(&mutex2);
	sumatorio_concurrente += mi_suma;
	mi_mutex_unlock(&mutex2);

}

int main(){

	long long sumatorio = 0;

	//inicialización del vector
	for (int i = 0; i < n_elementos; i ++){
		v_elems[i] = i;
	}

	//calculo del sumatorio secuencial
	for (int i = 0; i < n_elementos; i ++){
		sumatorio = sumatorio + v_elems[i];
	}
	

	for (int i = 0; i < n_threads; i++){
		std::thread th(funcion, i);
		v_threads.push_back(std::move(th));
	}

	for (std::thread & th : v_threads){

		if (th.joinable())
			th.join();
	}

	cout << "El valor de la componente 200000 es: " << v_elems[200000] << endl;
	cout << "El resultado de la operación sumatorio es : " << to_string(sumatorio) << endl;
	cout << "El resu de la oper sumatorio concurrente es : " << to_string(sumatorio_concurrente) << endl;
	
	return 0;

}
```

### mi_mutex-alumno.s

```

    .global mi_mutex_lock
    .align 2
    .type mi_mutex_lock,%function

mi_mutex_lock:
    .cfi_startproc
    
loop:
    ldaxr       w1, [x0]
    cmp         w1, #0
    b.ne        loop           // si no esta disponible el mutex seguimos intentando hasta w1 = 0

    mov         w1, #1
    stlxr       w2, w1, [x0]
    cbnz        w2, loop

    dmb         ish            // aseguramos orden de operaciones de memoria tras aquirir el mutex

	ret 
    .cfi_endproc

    .global mi_mutex_unlock
    .align 2
    .type mi_mutex_unlock,%function
mi_mutex_unlock:
    .cfi_startproc

    dmb         ish
    str         wzr, [x0]

    ret
    .cfi_endproc
```
