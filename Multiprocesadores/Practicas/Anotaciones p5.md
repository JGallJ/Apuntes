## pi_serie

**1. Comparar el codigo generado por el compilador en -O0 y -O3 para le bucle principal**

**Binario de -O0**
```
for (i = 0; i < nsubintervals; i++){
  400d2c:	f9004bff 	str	xzr, [sp, #144]
  400d30:	14000014 	b	400d80 <main+0x1cc>
    double x = (i-0.5)*subinterval;  // S1
  400d34:	fd404be0 	ldr	d0, [sp, #144]
  400d38:	5e61d801 	scvtf	d1, d0
  400d3c:	1e6c1000 	fmov	d0, #5.000000000000000000e-01
  400d40:	1e603820 	fsub	d0, d1, d0
  400d44:	fd403fe1 	ldr	d1, [sp, #120]
  400d48:	1e600820 	fmul	d0, d1, d0
  400d4c:	fd0033e0 	str	d0, [sp, #96]
    area = area + 4.0/(1.0 + x*x);   // S2
  400d50:	fd4033e0 	ldr	d0, [sp, #96]
  400d54:	1e600801 	fmul	d1, d0, d0
  400d58:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400d5c:	1e602820 	fadd	d0, d1, d0
  400d60:	1e621001 	fmov	d1, #4.000000000000000000e+00
  400d64:	1e601820 	fdiv	d0, d1, d0
  400d68:	fd404fe1 	ldr	d1, [sp, #152]
  400d6c:	1e602820 	fadd	d0, d1, d0
  400d70:	fd004fe0 	str	d0, [sp, #152]
  for (i = 0; i < nsubintervals; i++){
  400d74:	f9404be0 	ldr	x0, [sp, #144]
  400d78:	91000400 	add	x0, x0, #0x1
  400d7c:	f9004be0 	str	x0, [sp, #144]
  400d80:	f9404be1 	ldr	x1, [sp, #144]
  400d84:	f94047e0 	ldr	x0, [sp, #136]
  400d88:	eb00003f 	cmp	x1, x0
  400d8c:	54fffd4b 	b.lt	400d34 <main+0x180>  // b.tstop
  }
 ```

Nº instrucciones coma flotante es de 10, contando la conversion scvtf

**Binario de -O3**
```
for (i = 0; i < nsubintervals; i++){
  400be4:	f100029f 	cmp	x20, #0x0
  400be8:	5400184d 	b.le	400ef0 <main+0x440>
  400bec:	d1000680 	sub	x0, x20, #0x1
  400bf0:	f100181f 	cmp	x0, #0x6
  400bf4:	54001829 	b.ls	400ef8 <main+0x448>  // b.plast
  400bf8:	b0000000 	adrp	x0, 401000 <register_tm_clones+0x20>
  area = 0.0;
  400bfc:	2f00e403 	movi	d3, #0x0
  400c00:	4e080409 	dup	v9.2d, v0.d[0]
  400c04:	d341fe82 	lsr	x2, x20, #1
  for (i = 0; i < nsubintervals; i++){
  400c08:	3dc0bc04 	ldr	q4, [x0, #752]
  400c0c:	b0000000 	adrp	x0, 401000 <register_tm_clones+0x20>
    double x = (i-0.5)*subinterval;  // S1
  400c10:	6f07f407 	fmov	v7.2d, #-5.000000000000000000e-01
  for (i = 0; i < nsubintervals; i++){
  400c14:	d2800001 	mov	x1, #0x0                   	// #0
  400c18:	3dc0c008 	ldr	q8, [x0, #768]
    area = area + 4.0/(1.0 + x*x);   // S2
  400c1c:	6f03f606 	fmov	v6.2d, #1.000000000000000000e+00
  400c20:	6f00f605 	fmov	v5.2d, #4.000000000000000000e+00
  400c24:	d503201f 	nop
  400c28:	4ea41c81 	mov	v1.16b, v4.16b
  400c2c:	91000421 	add	x1, x1, #0x1
  400c30:	4ea61cc2 	mov	v2.16b, v6.16b
  400c34:	4ee88484 	add	v4.2d, v4.2d, v8.2d
    double x = (i-0.5)*subinterval;  // S1
  400c38:	4e61d821 	scvtf	v1.2d, v1.2d
  400c3c:	4e67d421 	fadd	v1.2d, v1.2d, v7.2d
  400c40:	6e69dc21 	fmul	v1.2d, v1.2d, v9.2d
    area = area + 4.0/(1.0 + x*x);   // S2
  400c44:	4e61cc22 	fmla	v2.2d, v1.2d, v1.2d
  400c48:	6e62fca1 	fdiv	v1.2d, v5.2d, v2.2d
  400c4c:	1e604022 	fmov	d2, d1
  400c50:	5e180421 	mov	d1, v1.d[1]
  400c54:	1e622862 	fadd	d2, d3, d2
  400c58:	1e622823 	fadd	d3, d1, d2
  for (i = 0; i < nsubintervals; i++){
  400c5c:	eb01005f 	cmp	x2, x1
  400c60:	54fffe41 	b.ne	400c28 <main+0x178>  // b.any
  400c64:	927ffa81 	and	x1, x20, #0xfffffffffffffffe
  400c68:	36000814 	tbz	w20, #0, 400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400c6c:	9e620021 	scvtf	d1, x1
  400c70:	1e6c1005 	fmov	d5, #5.000000000000000000e-01
    area = area + 4.0/(1.0 + x*x);   // S2
  400c74:	1e6e1004 	fmov	d4, #1.000000000000000000e+00
  400c78:	1e621002 	fmov	d2, #4.000000000000000000e+00
  for (i = 0; i < nsubintervals; i++){
  400c7c:	91000422 	add	x2, x1, #0x1
    double x = (i-0.5)*subinterval;  // S1
  400c80:	1e653821 	fsub	d1, d1, d5
  400c84:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400c88:	1f411021 	fmadd	d1, d1, d1, d4
  400c8c:	1e611841 	fdiv	d1, d2, d1
  400c90:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400c94:	eb02029f 	cmp	x20, x2
  400c98:	5400068d 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400c9c:	9e620041 	scvtf	d1, x2
  for (i = 0; i < nsubintervals; i++){
  400ca0:	91000822 	add	x2, x1, #0x2
    double x = (i-0.5)*subinterval;  // S1
  400ca4:	1e653821 	fsub	d1, d1, d5
  400ca8:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400cac:	1f411021 	fmadd	d1, d1, d1, d4
  400cb0:	1e611841 	fdiv	d1, d2, d1
  400cb4:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400cb8:	eb02029f 	cmp	x20, x2
  400cbc:	5400056d 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400cc0:	9e620041 	scvtf	d1, x2
  for (i = 0; i < nsubintervals; i++){
  400cc4:	91000c22 	add	x2, x1, #0x3
    double x = (i-0.5)*subinterval;  // S1
  400cc8:	1e653821 	fsub	d1, d1, d5
  400ccc:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400cd0:	1f411021 	fmadd	d1, d1, d1, d4
  400cd4:	1e611841 	fdiv	d1, d2, d1
  400cd8:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400cdc:	eb02029f 	cmp	x20, x2
  400ce0:	5400044d 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400ce4:	9e620041 	scvtf	d1, x2
  for (i = 0; i < nsubintervals; i++){
  400ce8:	91001022 	add	x2, x1, #0x4
    double x = (i-0.5)*subinterval;  // S1
  400cec:	1e653821 	fsub	d1, d1, d5
  400cf0:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400cf4:	1f411021 	fmadd	d1, d1, d1, d4
  400cf8:	1e611841 	fdiv	d1, d2, d1
  400cfc:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400d00:	eb02029f 	cmp	x20, x2
  400d04:	5400032d 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400d08:	9e620041 	scvtf	d1, x2
  for (i = 0; i < nsubintervals; i++){
  400d0c:	91001422 	add	x2, x1, #0x5
    double x = (i-0.5)*subinterval;  // S1
  400d10:	1e653821 	fsub	d1, d1, d5
  400d14:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400d18:	1f411021 	fmadd	d1, d1, d1, d4
  400d1c:	1e611841 	fdiv	d1, d2, d1
  400d20:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400d24:	eb02029f 	cmp	x20, x2
  400d28:	5400020d 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400d2c:	9e620041 	scvtf	d1, x2
  for (i = 0; i < nsubintervals; i++){
  400d30:	91001821 	add	x1, x1, #0x6
    double x = (i-0.5)*subinterval;  // S1
  400d34:	1e653821 	fsub	d1, d1, d5
  400d38:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400d3c:	1f411021 	fmadd	d1, d1, d1, d4
  400d40:	1e611841 	fdiv	d1, d2, d1
  400d44:	1e612863 	fadd	d3, d3, d1
  for (i = 0; i < nsubintervals; i++){
  400d48:	eb01029f 	cmp	x20, x1
  400d4c:	540000ed 	b.le	400d68 <main+0x2b8>
    double x = (i-0.5)*subinterval;  // S1
  400d50:	9e620021 	scvtf	d1, x1
  400d54:	1e653821 	fsub	d1, d1, d5
  400d58:	1e600821 	fmul	d1, d1, d0
    area = area + 4.0/(1.0 + x*x);   // S2
  400d5c:	1f411021 	fmadd	d1, d1, d1, d4
  400d60:	1e611842 	fdiv	d2, d2, d1
  400d64:	1e622863 	fadd	d3, d3, d2
  }

```

Nº instrucciones coma flotante es de 36, contando la conversion scvtf
Se pueden contar 6 iteraciones diferentes dentro del bucle, por lo tanto 36/6 = 6 operaciones por iteracion

**2. Ejecutar los binarios en -O0 y -O3 y anotar su tiempo de ejecucion. Calcular los MFLOPS alcanzados**

Ejecucion con **100.000.000** de subintervalos

- **Salida con -O0**

N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159267359042671
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.710685014724731445 sg
Wall Clock = 0.71071189899999998 sg

- **Salida con -O3**

N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159267359042671
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.711710989475250244 sg
Wall Clock = 0.711765928000000048 sg

MFLOPS de -O0 -> 100.000.000 \* 10 / 0.710685014724731445 \* 10⁶ = 1407,09 MFLOPS

MFLOPS de -03 ->  100.000.000 \* 6 / 0.711710989475250244 \* 10⁶ = 140,50 MFLOPS

## pi_unroll4

**1. Comparar el codigo generado por el compilador en -O0 y -O3 para le bucle principal**

**Binario de -O0**

```
for (i = 0; i < nsubintervals; i+=4){
  400d48:	f90057ff 	str	xzr, [sp, #168]
  400d4c:	1400004c 	b	400e7c <main+0x2c8>
    x = (i-0.5)*subinterval;  
  400d50:	fd4057e0 	ldr	d0, [sp, #168]
  400d54:	5e61d801 	scvtf	d1, d0
  400d58:	1e6c1000 	fmov	d0, #5.000000000000000000e-01
  400d5c:	1e603820 	fsub	d0, d1, d0
  400d60:	fd404be1 	ldr	d1, [sp, #144]
  400d64:	1e600820 	fmul	d0, d1, d0
  400d68:	fd003fe0 	str	d0, [sp, #120]
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400d6c:	910143e0 	add	x0, sp, #0x50
  400d70:	fd400001 	ldr	d1, [x0]
  400d74:	fd403fe0 	ldr	d0, [sp, #120]
  400d78:	1e600802 	fmul	d2, d0, d0
  400d7c:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400d80:	1e602840 	fadd	d0, d2, d0
  400d84:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400d88:	1e601840 	fdiv	d0, d2, d0
  400d8c:	1e602820 	fadd	d0, d1, d0
  400d90:	910143e0 	add	x0, sp, #0x50
  400d94:	fd000000 	str	d0, [x0]
    x = (i+0.5)*subinterval;  
  400d98:	fd4057e0 	ldr	d0, [sp, #168]
  400d9c:	5e61d801 	scvtf	d1, d0
  400da0:	1e6c1000 	fmov	d0, #5.000000000000000000e-01
  400da4:	1e602820 	fadd	d0, d1, d0
  400da8:	fd404be1 	ldr	d1, [sp, #144]
  400dac:	1e600820 	fmul	d0, d1, d0
  400db0:	fd003fe0 	str	d0, [sp, #120]
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
  400db4:	910143e0 	add	x0, sp, #0x50
  400db8:	fd400401 	ldr	d1, [x0, #8]
  400dbc:	fd403fe0 	ldr	d0, [sp, #120]
  400dc0:	1e600802 	fmul	d2, d0, d0
  400dc4:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400dc8:	1e602840 	fadd	d0, d2, d0
  400dcc:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400dd0:	1e601840 	fdiv	d0, d2, d0
  400dd4:	1e602820 	fadd	d0, d1, d0
  400dd8:	910143e0 	add	x0, sp, #0x50
  400ddc:	fd000400 	str	d0, [x0, #8]
    x = (i+1.5)*subinterval;  
  400de0:	fd4057e0 	ldr	d0, [sp, #168]
  400de4:	5e61d801 	scvtf	d1, d0
  400de8:	1e6f1000 	fmov	d0, #1.500000000000000000e+00
  400dec:	1e602820 	fadd	d0, d1, d0
  400df0:	fd404be1 	ldr	d1, [sp, #144]
  400df4:	1e600820 	fmul	d0, d1, d0
  400df8:	fd003fe0 	str	d0, [sp, #120]
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
  400dfc:	910143e0 	add	x0, sp, #0x50
  400e00:	fd400801 	ldr	d1, [x0, #16]
  400e04:	fd403fe0 	ldr	d0, [sp, #120]
  400e08:	1e600802 	fmul	d2, d0, d0
  400e0c:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400e10:	1e602840 	fadd	d0, d2, d0
  400e14:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400e18:	1e601840 	fdiv	d0, d2, d0
  400e1c:	1e602820 	fadd	d0, d1, d0
  400e20:	910143e0 	add	x0, sp, #0x50
  400e24:	fd000800 	str	d0, [x0, #16]
    x = (i+2.5)*subinterval;  
  400e28:	fd4057e0 	ldr	d0, [sp, #168]
  400e2c:	5e61d801 	scvtf	d1, d0
  400e30:	1e609000 	fmov	d0, #2.500000000000000000e+00
  400e34:	1e602820 	fadd	d0, d1, d0
  400e38:	fd404be1 	ldr	d1, [sp, #144]
  400e3c:	1e600820 	fmul	d0, d1, d0
  400e40:	fd003fe0 	str	d0, [sp, #120]
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400e44:	910143e0 	add	x0, sp, #0x50
  400e48:	fd400c01 	ldr	d1, [x0, #24]
  400e4c:	fd403fe0 	ldr	d0, [sp, #120]
  400e50:	1e600802 	fmul	d2, d0, d0
  400e54:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400e58:	1e602840 	fadd	d0, d2, d0
  400e5c:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400e60:	1e601840 	fdiv	d0, d2, d0
  400e64:	1e602820 	fadd	d0, d1, d0
  400e68:	910143e0 	add	x0, sp, #0x50
  400e6c:	fd000c00 	str	d0, [x0, #24]
  for (i = 0; i < nsubintervals; i+=4){
  400e70:	f94057e0 	ldr	x0, [sp, #168]
  400e74:	91001000 	add	x0, x0, #0x4
  400e78:	f90057e0 	str	x0, [sp, #168]
  400e7c:	f94057e1 	ldr	x1, [sp, #168]
  400e80:	f94053e0 	ldr	x0, [sp, #160]
  400e84:	eb00003f 	cmp	x1, x0
  400e88:	54fff64b 	b.lt	400d50 <main+0x19c>  // b.tstop
  }

```

Nº instrucciones coma flotante es de 10\*4=40, contando la conversion scvtf

**Binario de -O3**
```
for (i = 0; i < nsubintervals; i+=4){
  400be4:	f100029f 	cmp	x20, #0x0
  400be8:	540011cd 	b.le	400e20 <main+0x370>
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
    x = (i+0.5)*subinterval;  
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
    x = (i+1.5)*subinterval;  
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
    x = (i+2.5)*subinterval;  
  400bec:	b0000000 	adrp	x0, 401000 <__libc_csu_init>
  400bf0:	d1000682 	sub	x2, x20, #0x1
  for (i = 0; i < nsubintervals; i+=4){
  400bf4:	6f00e404 	movi	v4.2d, #0x0
  400bf8:	d2800001 	mov	x1, #0x0                   	// #0
    x = (i+2.5)*subinterval;  
  400bfc:	3dc08812 	ldr	q18, [x0, #544]
  400c00:	b0000000 	adrp	x0, 401000 <__libc_csu_init>
  400c04:	4e080668 	dup	v8.2d, v19.d[0]
  400c08:	d342fc42 	lsr	x2, x2, #2
  for (i = 0; i < nsubintervals; i+=4){
  400c0c:	4f000400 	movi	v0.4s, #0x0
  400c10:	91000442 	add	x2, x2, #0x1
  400c14:	4ea41c85 	mov	v5.16b, v4.16b
    x = (i+2.5)*subinterval;  
  400c18:	6f03f411 	fmov	v17.2d, #5.000000000000000000e-01
    x = (i-0.5)*subinterval;  
  400c1c:	6f07f410 	fmov	v16.2d, #-5.000000000000000000e-01
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c20:	6f03f607 	fmov	v7.2d, #1.000000000000000000e+00
  400c24:	6f00f606 	fmov	v6.2d, #4.000000000000000000e+00
  400c28:	3dc08c09 	ldr	q9, [x0, #560]
  400c2c:	d503201f 	nop
  400c30:	91000421 	add	x1, x1, #0x1
    x = (i-0.5)*subinterval;  
  400c34:	4e61d801 	scvtf	v1.2d, v0.2d
  400c38:	4ee98400 	add	v0.2d, v0.2d, v9.2d
  400c3c:	4e70d423 	fadd	v3.2d, v1.2d, v16.2d
    x = (i+2.5)*subinterval;  
  400c40:	4e71d422 	fadd	v2.2d, v1.2d, v17.2d
  400c44:	4e72d421 	fadd	v1.2d, v1.2d, v18.2d
  400c48:	6e184462 	mov	v2.d[1], v3.d[1]
  400c4c:	6e68dc21 	fmul	v1.2d, v1.2d, v8.2d
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c50:	4ea71ce3 	mov	v3.16b, v7.16b
    x = (i+2.5)*subinterval;  
  400c54:	6e68dc42 	fmul	v2.2d, v2.2d, v8.2d
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c58:	4e61cc23 	fmla	v3.2d, v1.2d, v1.2d
  400c5c:	4ea71ce1 	mov	v1.16b, v7.16b
  400c60:	4e62cc41 	fmla	v1.2d, v2.2d, v2.2d
  400c64:	6e63fcc2 	fdiv	v2.2d, v6.2d, v3.2d
  400c68:	6e61fcc1 	fdiv	v1.2d, v6.2d, v1.2d
  400c6c:	4e62d4a5 	fadd	v5.2d, v5.2d, v2.2d
  400c70:	4e61d484 	fadd	v4.2d, v4.2d, v1.2d
  400c74:	eb01005f 	cmp	x2, x1
  400c78:	54fffdc8 	b.hi	400c30 <main+0x180>  // b.pmore
  400c7c:	1e6040a0 	fmov	d0, d5
  400c80:	1e604081 	fmov	d1, d4
  400c84:	5e1804a5 	mov	d5, v5.d[1]
  400c88:	5e180488 	mov	d8, v4.d[1]
  }
```

Nº instrucciones coma flotante es de 18, contando la conversion scvtf

**2. Ejecutar los binarios en -O0 y -O3 y anotar su tiempo de ejecucion. Calcular los MFLOPS alcanzados**

Ejecucion con **100.000.000** de subintervalos

**Salida de -O0**

N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159267359021666
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.710121989250183105 sg
Wall Clock = 0.710143741999999967 sg

**Salida de -O3**
N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159267359021666
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.692525982856750488 sg
Wall Clock = 0.692941452000000013 sg

MFLOPS de -O0 -> 100.000.000 \* 10 / 0.710121989250183105 \* 10⁶ = 1408,21 MFLOPS

MFLOPS de -O3 -> 100.000.000 \* (18/4) / 0.692525982856750488 \* 10⁶ = 649,79 MFLOPS

## pi_unroll4_reduction

**1. Comparar el codigo generado por el compilador en -O0 y -O3 para le bucle principal**

**Binario de -O0**

```
for (i = 0; i < nsubintervals; i+=4){
  400d58:	f90053ff 	str	xzr, [sp, #160]
  400d5c:	14000040 	b	400e5c <main+0x2a8>
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400d60:	910143e0 	add	x0, sp, #0x50
  400d64:	fd400001 	ldr	d1, [x0]
  400d68:	fd4057e0 	ldr	d0, [sp, #168]
  400d6c:	1e600802 	fmul	d2, d0, d0
  400d70:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400d74:	1e602840 	fadd	d0, d2, d0
  400d78:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400d7c:	1e601840 	fdiv	d0, d2, d0
  400d80:	1e602820 	fadd	d0, d1, d0
  400d84:	910143e0 	add	x0, sp, #0x50
  400d88:	fd000000 	str	d0, [x0]
    x = x + subinterval;  
  400d8c:	fd4057e1 	ldr	d1, [sp, #168]
  400d90:	fd4047e0 	ldr	d0, [sp, #136]
  400d94:	1e602820 	fadd	d0, d1, d0
  400d98:	fd0057e0 	str	d0, [sp, #168]
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
  400d9c:	910143e0 	add	x0, sp, #0x50
  400da0:	fd400401 	ldr	d1, [x0, #8]
  400da4:	fd4057e0 	ldr	d0, [sp, #168]
  400da8:	1e600802 	fmul	d2, d0, d0
  400dac:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400db0:	1e602840 	fadd	d0, d2, d0
  400db4:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400db8:	1e601840 	fdiv	d0, d2, d0
  400dbc:	1e602820 	fadd	d0, d1, d0
  400dc0:	910143e0 	add	x0, sp, #0x50
  400dc4:	fd000400 	str	d0, [x0, #8]
    x = x + subinterval;  
  400dc8:	fd4057e1 	ldr	d1, [sp, #168]
  400dcc:	fd4047e0 	ldr	d0, [sp, #136]
  400dd0:	1e602820 	fadd	d0, d1, d0
  400dd4:	fd0057e0 	str	d0, [sp, #168]
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
  400dd8:	910143e0 	add	x0, sp, #0x50
  400ddc:	fd400801 	ldr	d1, [x0, #16]
  400de0:	fd4057e0 	ldr	d0, [sp, #168]
  400de4:	1e600802 	fmul	d2, d0, d0
  400de8:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400dec:	1e602840 	fadd	d0, d2, d0
  400df0:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400df4:	1e601840 	fdiv	d0, d2, d0
  400df8:	1e602820 	fadd	d0, d1, d0
  400dfc:	910143e0 	add	x0, sp, #0x50
  400e00:	fd000800 	str	d0, [x0, #16]
    x = x + subinterval;  
  400e04:	fd4057e1 	ldr	d1, [sp, #168]
  400e08:	fd4047e0 	ldr	d0, [sp, #136]
  400e0c:	1e602820 	fadd	d0, d1, d0
  400e10:	fd0057e0 	str	d0, [sp, #168]
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400e14:	910143e0 	add	x0, sp, #0x50
  400e18:	fd400c01 	ldr	d1, [x0, #24]
  400e1c:	fd4057e0 	ldr	d0, [sp, #168]
  400e20:	1e600802 	fmul	d2, d0, d0
  400e24:	1e6e1000 	fmov	d0, #1.000000000000000000e+00
  400e28:	1e602840 	fadd	d0, d2, d0
  400e2c:	1e621002 	fmov	d2, #4.000000000000000000e+00
  400e30:	1e601840 	fdiv	d0, d2, d0
  400e34:	1e602820 	fadd	d0, d1, d0
  400e38:	910143e0 	add	x0, sp, #0x50
  400e3c:	fd000c00 	str	d0, [x0, #24]
    x = x + subinterval;  
  400e40:	fd4057e1 	ldr	d1, [sp, #168]
  400e44:	fd4047e0 	ldr	d0, [sp, #136]
  400e48:	1e602820 	fadd	d0, d1, d0
  400e4c:	fd0057e0 	str	d0, [sp, #168]
  for (i = 0; i < nsubintervals; i+=4){
  400e50:	f94053e0 	ldr	x0, [sp, #160]
  400e54:	91001000 	add	x0, x0, #0x4
  400e58:	f90053e0 	str	x0, [sp, #160]
  400e5c:	f94053e1 	ldr	x1, [sp, #160]
  400e60:	f9404fe0 	ldr	x0, [sp, #152]
  400e64:	eb00003f 	cmp	x1, x0
  400e68:	54fff7cb 	b.lt	400d60 <main+0x1ac>  // b.tstop
  }

```

Nº instrucciones coma flotante es de 7\*4, contando la conversion scvtf

**Binario de -O3**

```
for (i = 0; i < nsubintervals; i+=4){
  400bf0:	f100029f 	cmp	x20, #0x0
  400bf4:	54000fed 	b.le	400df0 <main+0x340>
  varea[2] = 0.0;
  400bf8:	1e604232 	fmov	d18, d17
  varea[1] = 0.0;
  400bfc:	1e604228 	fmov	d8, d17
  varea[0] = 0.0;
  400c00:	1e604230 	fmov	d16, d17
  for (i = 0; i < nsubintervals; i+=4){
  400c04:	d2800001 	mov	x1, #0x0                   	// #0
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400c08:	1e621005 	fmov	d5, #4.000000000000000000e+00
  400c0c:	d503201f 	nop
    x = x + subinterval;  
  400c10:	1e632882 	fadd	d2, d4, d3
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400c14:	1f430063 	fmadd	d3, d3, d3, d0
  for (i = 0; i < nsubintervals; i+=4){
  400c18:	91001021 	add	x1, x1, #0x4
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
    x = x + subinterval;  
  400c1c:	1e622881 	fadd	d1, d4, d2
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
  400c20:	1f420042 	fmadd	d2, d2, d2, d0
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400c24:	1e6318b3 	fdiv	d19, d5, d3
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
    x = x + subinterval;  
  400c28:	1e612887 	fadd	d7, d4, d1
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
  400c2c:	1f410021 	fmadd	d1, d1, d1, d0
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c30:	1f4700e6 	fmadd	d6, d7, d7, d0
    x = x + subinterval;  
  400c34:	1e672883 	fadd	d3, d4, d7
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
  400c38:	1e6218a2 	fdiv	d2, d5, d2
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
  400c3c:	1e6118a1 	fdiv	d1, d5, d1
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c40:	1e6618a6 	fdiv	d6, d5, d6
    varea[0] = varea[0] + 4.0/(1.0 + x*x);   
  400c44:	1e732a10 	fadd	d16, d16, d19
    varea[1] = varea[1] + 4.0/(1.0 + x*x);   
  400c48:	1e622908 	fadd	d8, d8, d2
    varea[2] = varea[2] + 4.0/(1.0 + x*x);   
  400c4c:	1e612a52 	fadd	d18, d18, d1
    varea[3] = varea[3] + 4.0/(1.0 + x*x);   
  400c50:	1e662a31 	fadd	d17, d17, d6
  for (i = 0; i < nsubintervals; i+=4){
  400c54:	eb01029f 	cmp	x20, x1
  400c58:	54fffdcc 	b.gt	400c10 <main+0x160>
  }
```

Nº instrucciones coma flotante es de 20, contando la conversion scvtf

**2. Ejecutar los binarios en -O0 y -O3 y anotar su tiempo de ejecucion. Calcular los MFLOPS alcanzados**

Ejecucion con **100.000.000** de subintervalos

**Salida de -O0**

N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159265251650055
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.707742989063262939 sg
Wall Clock = 0.708106278000000033 sg

**Salida de -O3**

N� de procesadores: 1
N� de threads: 1
Resoluci�n de los relojes:
 - system_clock(std::clock): 1 us -- 1 MHz
***************************************
 PIcalculado =3.14159265251650055
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.698357999324798584 sg
Wall Clock = 0.698374281999999957 sg

MFLOPS de -O0 -> 100.000.000 \* 7 / 0.707742989063262939 \* 10⁶ = 989,06 MFLOPS

MFLOPS de -O3 -> 100.000.000 \* (20/4) / 0.698357999324798584 \* 10⁶ = 715,96 MFLOPS

## Ejecucion paralela
### Paralelizacion automatica de pi

Compilado con las siguientes opciones
``` 
g++ -O3 -fopenmp -ftree-parallelize-loops=1 -fopt-info-vec-optimized -fopt-info-omp-all  pi.cpp -o pi
```

NO HAY MANERA DE MOMENTO


Saber que con std::clock() este cuenta el tiempo de CPU utilizado por el programa para realizar su tarea, mientras que std:: ... ::now() cuenta el tiempo REAL que transcurre en el reloj del sistema

### Paralelizacion con directivas OpenMP

Se han añadido cosas que faltaban en el codigo, ademas de la siguiente directiva de OpenMP

```#pragma omp parallel for private(x) reduction(+:area) ```

Con esta podemos paralelizar el bucle sin problema alguno

Codigo del bucle principal 
```
00000000004015b0 <main._omp_fn.0>:
  #pragma omp parallel for private(x) reduction(+:area)
  4015b0:	a9bc7bfd 	stp	x29, x30, [sp, #-64]!
  4015b4:	910003fd 	mov	x29, sp
  4015b8:	a90153f3 	stp	x19, x20, [sp, #16]
  4015bc:	aa0003f3 	mov	x19, x0
  4015c0:	f90013f5 	str	x21, [sp, #32]
  4015c4:	fd0017e8 	str	d8, [sp, #40]
  4015c8:	97fffe4e 	bl	400f00 <omp_get_num_threads@plt>
  4015cc:	93407c14 	sxtw	x20, w0
  4015d0:	97fffe30 	bl	400e90 <omp_get_thread_num@plt>
  4015d4:	f9400275 	ldr	x21, [x19]
  4015d8:	93407c01 	sxtw	x1, w0
  4015dc:	fd400663 	ldr	d3, [x19, #8]
  4015e0:	9ad40ea2 	sdiv	x2, x21, x20
  4015e4:	9b14d440 	msub	x0, x2, x20, x21
  4015e8:	eb00003f 	cmp	x1, x0
  4015ec:	54000ecb 	b.lt	4017c4 <main._omp_fn.0+0x214>  // b.tstop
  4015f0:	9b010041 	madd	x1, x2, x1, x0
  4015f4:	2f00e408 	movi	d8, #0x0
  4015f8:	8b010043 	add	x3, x2, x1
  4015fc:	eb03003f 	cmp	x1, x3
  401600:	54000c4a 	b.ge	401788 <main._omp_fn.0+0x1d8>  // b.tcont
  401604:	d1000440 	sub	x0, x2, #0x1
  401608:	f100181f 	cmp	x0, #0x6
  40160c:	54000409 	b.ls	40168c <main._omp_fn.0+0xdc>  // b.plast
  401610:	f9001be1 	str	x1, [sp, #48]
  401614:	91000425 	add	x5, x1, #0x1
  401618:	90000000 	adrp	x0, 401000 <main+0xd0>
  40161c:	d341fc44 	lsr	x4, x2, #1
  401620:	3dc00fe2 	ldr	q2, [sp, #48]
  401624:	4e080470 	dup	v16.2d, v3.d[0]
    x = (i-0.5)*subinterval;  // S1
  401628:	6f07f406 	fmov	v6.2d, #-5.000000000000000000e-01
    area = area + 4.0/(1.0 + x*x);   // S2
  40162c:	6f03f605 	fmov	v5.2d, #1.000000000000000000e+00
  401630:	6f00f604 	fmov	v4.2d, #4.000000000000000000e+00
  401634:	4e181ca2 	mov	v2.d[1], x5
  401638:	3dc1f807 	ldr	q7, [x0, #2016]
  #pragma omp parallel for private(x) reduction(+:area)
  40163c:	d2800000 	mov	x0, #0x0                   	// #0
  401640:	4ea21c40 	mov	v0.16b, v2.16b
  401644:	91000400 	add	x0, x0, #0x1
    area = area + 4.0/(1.0 + x*x);   // S2
  401648:	4ea51ca1 	mov	v1.16b, v5.16b
  40164c:	4ee78442 	add	v2.2d, v2.2d, v7.2d
    x = (i-0.5)*subinterval;  // S1
  401650:	4e61d800 	scvtf	v0.2d, v0.2d
  401654:	4e66d400 	fadd	v0.2d, v0.2d, v6.2d
  401658:	6e70dc00 	fmul	v0.2d, v0.2d, v16.2d
    area = area + 4.0/(1.0 + x*x);   // S2
  40165c:	4e60cc01 	fmla	v1.2d, v0.2d, v0.2d
  401660:	6e61fc80 	fdiv	v0.2d, v4.2d, v1.2d
  401664:	1e604001 	fmov	d1, d0
  401668:	5e180400 	mov	d0, v0.d[1]
  40166c:	1e612901 	fadd	d1, d8, d1
  401670:	1e612808 	fadd	d8, d0, d1
  401674:	eb00009f 	cmp	x4, x0
  401678:	54fffe41 	b.ne	401640 <main._omp_fn.0+0x90>  // b.any
  40167c:	927ff840 	and	x0, x2, #0xfffffffffffffffe
  401680:	8b000021 	add	x1, x1, x0
  401684:	eb02001f 	cmp	x0, x2
  401688:	54000800 	b.eq	401788 <main._omp_fn.0+0x1d8>  // b.none
    x = (i-0.5)*subinterval;  // S1
  40168c:	9e620020 	scvtf	d0, x1
  401690:	1e6c1004 	fmov	d4, #5.000000000000000000e-01
    area = area + 4.0/(1.0 + x*x);   // S2
  401694:	1e6e1002 	fmov	d2, #1.000000000000000000e+00
  401698:	1e621001 	fmov	d1, #4.000000000000000000e+00
  40169c:	91000420 	add	x0, x1, #0x1
    x = (i-0.5)*subinterval;  // S1
  4016a0:	1e643800 	fsub	d0, d0, d4
  4016a4:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  4016a8:	1f400800 	fmadd	d0, d0, d0, d2
  4016ac:	1e601820 	fdiv	d0, d1, d0
  4016b0:	1e602908 	fadd	d8, d8, d0
  4016b4:	eb00007f 	cmp	x3, x0
  4016b8:	5400068d 	b.le	401788 <main._omp_fn.0+0x1d8>

			// CREO QUE ESTO ES UNA ITERACION NORMAL DEL BUCLE
		    x = (i-0.5)*subinterval;  // S1
		  4016bc:	9e620000 	scvtf	d0, x0
		  4016c0:	91000820 	add	x0, x1, #0x2
		  4016c4:	1e643800 	fsub	d0, d0, d4
		  4016c8:	1e630800 	fmul	d0, d0, d3
		    area = area + 4.0/(1.0 + x*x);   // S2
		  4016cc:	1f400800 	fmadd	d0, d0, d0, d2
		  4016d0:	1e601820 	fdiv	d0, d1, d0
		  4016d4:	1e602908 	fadd	d8, d8, d0
		  4016d8:	eb00007f 	cmp	x3, x0
		  4016dc:	5400056d 	b.le	401788 <main._omp_fn.0+0x1d8>
		  
    x = (i-0.5)*subinterval;  // S1
  4016e0:	9e620000 	scvtf	d0, x0
  4016e4:	91000c20 	add	x0, x1, #0x3
  4016e8:	1e643800 	fsub	d0, d0, d4
  4016ec:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  4016f0:	1f400800 	fmadd	d0, d0, d0, d2
  4016f4:	1e601820 	fdiv	d0, d1, d0
  4016f8:	1e602908 	fadd	d8, d8, d0
  4016fc:	eb00007f 	cmp	x3, x0
  401700:	5400044d 	b.le	401788 <main._omp_fn.0+0x1d8>
    x = (i-0.5)*subinterval;  // S1
  401704:	9e620000 	scvtf	d0, x0
  401708:	91001020 	add	x0, x1, #0x4
  40170c:	1e643800 	fsub	d0, d0, d4
  401710:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  401714:	1f400800 	fmadd	d0, d0, d0, d2
  401718:	1e601820 	fdiv	d0, d1, d0
  40171c:	1e602908 	fadd	d8, d8, d0
  401720:	eb00007f 	cmp	x3, x0
  401724:	5400032d 	b.le	401788 <main._omp_fn.0+0x1d8>
    x = (i-0.5)*subinterval;  // S1
  401728:	9e620000 	scvtf	d0, x0
  40172c:	91001420 	add	x0, x1, #0x5
  401730:	1e643800 	fsub	d0, d0, d4
  401734:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  401738:	1f400800 	fmadd	d0, d0, d0, d2
  40173c:	1e601820 	fdiv	d0, d1, d0
  401740:	1e602908 	fadd	d8, d8, d0
  401744:	eb00007f 	cmp	x3, x0
  401748:	5400020d 	b.le	401788 <main._omp_fn.0+0x1d8>
    x = (i-0.5)*subinterval;  // S1
  40174c:	9e620000 	scvtf	d0, x0
  401750:	91001821 	add	x1, x1, #0x6
  401754:	1e643800 	fsub	d0, d0, d4
  401758:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  40175c:	1f400800 	fmadd	d0, d0, d0, d2
  401760:	1e601820 	fdiv	d0, d1, d0
  401764:	1e602908 	fadd	d8, d8, d0
  401768:	eb01007f 	cmp	x3, x1
  40176c:	540000ed 	b.le	401788 <main._omp_fn.0+0x1d8>
    x = (i-0.5)*subinterval;  // S1
  401770:	9e620020 	scvtf	d0, x1
  401774:	1e643800 	fsub	d0, d0, d4
  401778:	1e630800 	fmul	d0, d0, d3
    area = area + 4.0/(1.0 + x*x);   // S2
  40177c:	1f400800 	fmadd	d0, d0, d0, d2
  401780:	1e601821 	fdiv	d1, d1, d0
  401784:	1e612908 	fadd	d8, d8, d1
  #pragma omp parallel for private(x) reduction(+:area)
  401788:	91004273 	add	x19, x19, #0x10
  40178c:	f9400274 	ldr	x20, [x19]
  401790:	9e670280 	fmov	d0, x20
  401794:	aa1303e2 	mov	x2, x19
  401798:	aa1403e0 	mov	x0, x20
  40179c:	1e602900 	fadd	d0, d8, d0
  4017a0:	9e660001 	fmov	x1, d0
  4017a4:	9400002f 	bl	401860 <__aarch64_cas8_acq_rel>
  4017a8:	eb00029f 	cmp	x20, x0
  4017ac:	54000121 	b.ne	4017d0 <main._omp_fn.0+0x220>  // b.any
  4017b0:	a94153f3 	ldp	x19, x20, [sp, #16]
  4017b4:	f94013f5 	ldr	x21, [sp, #32]
  4017b8:	fd4017e8 	ldr	d8, [sp, #40]
  4017bc:	a8c47bfd 	ldp	x29, x30, [sp], #64
  4017c0:	d65f03c0 	ret
  4017c4:	91000442 	add	x2, x2, #0x1
  4017c8:	d2800000 	mov	x0, #0x0                   	// #0
  4017cc:	17ffff89 	b	4015f0 <main._omp_fn.0+0x40>
  4017d0:	aa0003f4 	mov	x20, x0
  4017d4:	17ffffef 	b	401790 <main._omp_fn.0+0x1e0>
  4017d8:	d503201f 	nop
  4017dc:	d503201f 	nop
  4017e0:	00000002 	.word	0x00000002
  4017e4:	00000000 	.word	0x00000000
  4017e8:	00000002 	.word	0x00000002
  4017ec:	00000000 	.word	0x00000000

```

56 FLOP en total contados
diria que son 6 por iteracion del bucle normal

Threads:
**- 1 thread**
 PIcalculado = 3.14159267359042671
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.711733996868133545
Wall Clock = 0.711752692999999992
OMP time = 0.711751633323729038 sg


**- 2 threads**
 PIcalculado = 3.14159267358990979
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.706170976161956787
Wall Clock = 0.355970137999999992
OMP time = 0.355969108175486326 sg

**- 4 threads**
 PIcalculado = 3.14159267358968197
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.694557011127471924
Wall Clock = 0.17812038999999999
OMP time = 0.178119670134037733 sg

**- 8 threads**
 PIcalculado = 3.14159267358981475
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.712348997592926025
Wall Clock = 0.0892308160000000045
OMP time = 0.0892296764068305492 sg

**- 16 threads**
 PIcalculado = 3.1415926735898827
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.619467020034790039
Wall Clock = 0.0449324109999999985
OMP time = 0.0449316310696303844 sg

**- 32 threads**
 PIcalculado = 3.14159267358980765
 PIreal = 3.1415926535897932385

*** tiempos ***
CPU clock = 0.720860004425048828
Wall Clock = 0.045579175999999999
OMP time = 0.0455782758072018623 sg


PARA LOS CALCULOS SE COGE EL WALL CLOCK

MFLOPS con 1 thr -> 100.000.000 \* 6 / 0.711752692999999992 \* 10⁶ = 842,98

MFLOPS con 2 thrs -> 100.000.000 \* 6 / 0.355970137999999992 \* 10⁶ = 1685,53

MFLOPS con 4 thrs -> 100.000.000 \* 6 / 0.17812038999999999 \* 10⁶ = 3368,51

MFLOPS con 8 thrs -> 100.000.000 \* 6 / 0.0892308160000000045 \* 10⁶ = 6724,13

MFLOPS con 16 thrs -> 100.000.000 \* 6 / 0.0449324109999999985 \* 10⁶ = 13353,38

MFLOPS con 32 thrs -> 100.000.000 \* 6 / 0.045579175999999999 \* 10⁶ = 13163,90

#### Planificación de iteraciones 
Todo hecho con 4 threads

***static***

CPU clock = 0.699038982391357422
Wall Clock = 0.178296402999999992
OMP time = 0.178295113146305084 sg

***static con valores de chunk***
- 1
CPU clock = 0.688230991363525391
Wall Clock = 0.175545730000000011
OMP time = 0.1755447699688375 sg

- 100
CPU clock = 0.696615993976593018
Wall Clock = 0.17804702
OMP time = 0.178046179935336113 sg

- 10000
CPU clock = 0.700823009014129639
Wall Clock = 0.178083680999999994
OMP time = 0.178082980215549469 sg

***dynamic con valores de chunk***
- 1
CPU clock = 72.9921112060546875
Wall Clock = 18.2527384899999987
OMP time = 18.2527374499477446 sg

- 100
CPU clock = 0.838092982769012451
Wall Clock = 0.210586839999999997
OMP time = 0.210585750173777342 sg

- 10000
CPU clock = 0.688127994537353516
Wall Clock = 0.178499973000000006
OMP time = 0.178498992696404457 sg

***guided con valores de chunk***
- 1
CPU clock = 0.68320697546005249
Wall Clock = 0.17806829099999999
OMP time = 0.178067001048475504 sg

- 100
CPU clock = 0.698409974575042725
Wall Clock = 0.178223990999999998
OMP time = 0.178223190829157829 sg

- 10000
CPU clock = 0.687665998935699463
Wall Clock = 0.178110611000000002
OMP time = 0.17810934130102396 sg

### Multiplicación de matrices

**Resultados iniciales:**
Execute Sequential iterative matmult (M= 1000 N= 1000 P= 1000)
Time = 1.861200 seconds

Execute Sequential recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 1.279143 seconds

Execute parallel recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 1.281905 seconds

Modificaciones aplicadas
- Se ha cambiado el algoritmo de calculo de multiplicacion de matrices por el habitual por filas y columnas, paralelizando mediante tasks cada fila 
```
#pragma omp parallel
{
	#pragma omp single
	{
		int i, j, k;
		for (i = 0; i < m; i++) {
			#pragma omp task shared(A,B,C)
			for (j = 0; j < n; j++){
				C[i][j]=0;
				for (k = 0; k < p; k++)
				C[i][j] += A[i][k]*B[k][j];
			}
		//#pragma omp taskwait
		}
	}
}
```

SI se introduce la directiva \#pragma omp taskwait el tiempo que se consigue es cercano al de la version recursiva secuencial, ya que tiene que hacer esperas consatantes a que las tasks acaben, mientras que sin ella no espera y el algoritmo aumenta hasta un 15.80 de speedup (con 16 threads)

Tiempos:
- Con \#pragma omp taskwait
Execute Sequential iterative matmult (M= 1000 N= 1000 P= 1000)
Time = 1.861377 seconds

Execute Sequential recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 1.273448 seconds

Execute parallel recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 1.860929 seconds


- Sin \#pragma omp taskwait
Nº maximo de threads: 16
Execute Sequential iterative matmult (M= 1000 N= 1000 P= 1000)
Time = 1.859759 seconds

Execute Sequential recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 1.275543 seconds

Execute parallel recursive matmult (M= 1000 N= 1000 P= 1000)
OKAY
Time = 0.117725 seconds
