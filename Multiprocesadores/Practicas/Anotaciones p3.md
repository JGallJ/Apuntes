Trabajaremos en lab1.02

### Parte 1. Conversión de formato RGB a escala de grises

1. 
Funcion estandarizada, ponderacion donde le estamos dando un valor a cada color
gray = roundf(0.299*R + 0.587*G + 0.114*B);

2. 
Hacemos la compilacion: make test_rgb2gray
Ejecutamos la funcion: ./test_rgb2gray -c0 -r
para mirar el hexa: hexdump images/2013-10-02_Campo_Base_Annapurna_gray.jpg

3. ¿Ha vectorizado el bucle interno en rgb2gray_roundf0()?
Analiza el fichero que contiene el ensamblador y busca las instrucciones vectoriales (si existen) correspondientes al bucle en rgb2gray_roundf0()
Podemos apreciar en ensamblador que no esta vectorizado ya que sus funciones utilizan la "s" de scalar


4. ¿Ha vectorizado el bucle interno en rgb2gray_roundf1()?
El bucle de la funcion rgb2gray_roundf1 llega desde 350 (o por ahi) hasta el 626 donde .L18

Vamos a ver el rendimiento del codigo

Para la funcion rgb2gray_roundf0
                     Time
        función      (ms)    ns/px    Gpixels/s
  rgb2gray_roundf0   20.1     2.00       0.50 
          read_PGM   89.0     8.87       0.11 
           cmpGray   13.1     1.3       7.66      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

Para la funcion rgb2gray_roundf1
                     Time
        función      (ms)    ns/px    Gpixels/s
  rgb2gray_roundf1    9.0     0.90       1.11 
          read_PGM   88.9     8.86       0.11 
           cmpGray   12.6     1.3       7.97      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)
Practicamente el doble de rapido a pesar del codigo tan pesado. Para optimizarlo vamos a tratar de quitar las conversiones de datos en el codigo (funcion rgb2gray_cast0())

5. ¿Ha vectorizado el bucle interno en rgb2gray_cast0()?

Codigo practicamente igual

                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_cast0    8.7     0.87       1.15 
          read_PGM   89.0     8.87       0.11 
           cmpGray   12.5     1.2       8.01      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

6. 
Para rgb2gray_cast1() se ha puesto f en lugar de double, cargandonos bastantes conversiones de tipos

Funcion empieza en linea 2885 y termina en 3835 o por ahi
   - Bucle principal 3053-3707 aprox

                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_cast1    4.9     0.49       2.03 
          read_PGM   89.0     8.87       0.11 
           cmpGray   12.6     1.3       7.95      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

Añadiendo esta f hemos trabajado en vez de 64 bits con 32 bits consiguiendo la velocidad otra vez por 2

7. 

En esta variante se ha eliminado la suma del valor 0.5, produciendose diferencias (no mucho a priori) con un error de 1
                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_cast2    4.9     0.49       2.05 
          read_PGM   88.8     8.85       0.11 
           cmpGray   27.0     2.7       3.72     48.2%     1 (      0)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)


10. 


### Parte 2. Transformación en la disposición de datos

1. Modifica la funcion rgb2gray_SOA0()

Se ha añadido lo siguiente
for (int i = 0; i < height*width; i++)
    {
        R_in[i] = pixels_in[3*i + 0];
      	G_in[i] = pixels_in[3*i + 1]; 
        B_in[i] = pixels_in[3*i + 2]; 
    }

for (int i = 0; i < height*width; i++)
        {
             pixels_out[i] = (unsigned char)(0.5f + 0.299*R_in[i], 0.587*G_in[i], 0.114*B_in[i]);
        }

2. Comprobar que compila y ejecuta y medir rendimiento

Deberia salir x4 veces mas que la version 5 escalar
                     Time
        función      (ms)    ns/px    Gpixels/s
     rgb2gray_SOA0    3.0     0.30       3.30 
          read_PGM   88.4     8.80       0.11 
           cmpGray   12.0     1.2       8.39      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

3. 


4. Implementa la función rgb2gray_SOA1() como una variante de rgb2gray_SOA0() en la que se cuenta el tiempo de la transformacion de datos

Poner lo mismo de antes pero con un start_t antes de hacer la reasignacion en vectores

                     Time
        función      (ms)    ns/px    Gpixels/s
     rgb2gray_SOA1    6.5     0.65       1.55 
          read_PGM   88.1     8.77       0.11 
           cmpGray   11.6     1.2       8.67      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)


5. Escribe una función rgb2gray_block() que entrelace la transformación de los datos con los cálculos a realizar. De esta forma, las variables auxiliares almacenarán en formato SoA parte de los valores RGB (en concreto, BLOCK píxeles) en lugar de todos los valores RGB de la imagen.


pixels_in = &image_in->pixels[3*i];
pixels_out = &image_out->pixels[i];

R_in[j] = pixels_in[3*j + 0];
G_in[j] = pixels_in[3*j + 1]; 
B_in[j] = pixels_in[3*j + 2]; 

pixels_out[j] = (unsigned char)(0.5f + 0.299f*R_in[j] + 0.587f*$



6. Medir rendimiento

                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_block    5.1     0.50       1.98 
          read_PGM   88.1     8.78       0.11 
           cmpGray   12.4     1.2       8.09      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

- con BLOCK 128
                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_block    4.9     0.49       2.04 
          read_PGM   87.9     8.76       0.11 
           cmpGray   10.5     1.0       9.53      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)

- con BLOCK 256
                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_block    5.5     0.55       1.83 
          read_PGM   87.6     8.73       0.11 
           cmpGray   11.4     1.1       8.83      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)
- con BLOCK 512
                     Time
        función      (ms)    ns/px    Gpixels/s
    rgb2gray_block    6.2     0.61       1.63 
          read_PGM   87.7     8.74       0.11 
           cmpGray   10.5     1.0       9.56      0.0%     0 (     -1)
        función      Time    ns/px    Gpixels/s  %diff  max_dif (max_idx)
                     (ms)




