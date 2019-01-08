# Portable and Optimized C for 8-bit Systems

This article describes some optimizing techniques for ANSI C code for **all** 8-bit *vintage* systems, i.e., computers, consoles, hand-helds, scientific calculators from the end of the '70s until mid '90s and in particular for the systems based on the following *architectures* (and the derived and compatible architectures): 

 - Intel 8080 (*)
 - MOS 6502 
 - Motorola 6809
 - Zilog Z80 (*)

(*) The Zilog Z80 is an extension of the Intel 8080. Therefore an Intel 8080 binary is compatible with a Z80-based system but not the other way round.

Most of the techniques presented here remain valid on other 8-bit architectures such as the ones found in microcontrollers, e.g., on the Intel 8051.

The goal of this article is two-fold:

 1. present general techniques to **optimize** C code for **all** 8-bit systems;
 2. present general techniques to write **portable** C code, i.e., valid and compatible code for **all** 8-bit systems, including systems that are not natively and explicitly supported by C compilers 

## Preconditions
This article is **not** a introduction nor a manual for the *C* language and has the following preconditions:
 - knowledge of the *C* langiage;
 - knowledge of structured and object-oriented programming;
 - knowledge of basic use of compilers and linkers.

Besides this article does not cover in depth some advanced topics such as:

 - coding in specific domains such as graphics, sound, input/output;
 - interaction between C and Assembly.

These advanced topics are very important and would require separate articles.



## Definitions
In this article we will refer to *system*, *target* and *architecture* with the following meanings:

 - A *system* is any kind of processor-equipped machine such as computers, consoles, hand-helds, scientigic calculators, embedded systems, etc.
 - A *target* of a compiler is any kind of *system* supported by the compiler, i.e., a system for which the compiler provides specific support with libraries and the generator of a binary in the specific format.
 - An *architecture* is a processor family (e.g., Intel 8080, MOS 6502, etc.). Therefore a *target* has an *architecture* that corresponds to its processor. So it has only one architecture unless if it has more than two or more processors belonging to two different families (such as the Commodore 128, which has both a Z80 and a 6502-derived processor)

## Multi-target cross-compilers
In order to produce binaries from source code we recommend *multi-target* *cross-compilers* (i.e., compilers that are run on a modern PC and that produce binaries for multiple *targets*).

### Cross-compilers vs native compilers
We do not recommend the use of *native* compilers because they would be inconvenient (even if used inside an accellerated emulator) and would never produce the same kind of optimized code due to the limitated resources of the 8-bit system.

In particular we will refer to the following *multi-targer* *cross-compilers*:
 
We will refer in particular to the following *multi-target* *cross-compilers*:

| Architecture |Compiler/Dev-Kit  | Web Page 
|--|--|--|
| Intel 8080 |  ACK |  https://github.com/davidgiven/ack
| MOS 6502   |  CC65| https://github.com/cc65/cc65 
| Motorola 6809 | CMOC | https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html 
| Zilog 80   |  SCCZ80/ZSDCC (Z88DK) | https://github.com/z88dk/z88dk 

We mention other *multi-target* *cross-compilers* that we do not cover here but for which most of the described general techniques are valid:

- SDCC (http://sdcc.sourceforge.net/) per svariate architetture di microprocessori come lo Z80 e di microcontrollori come l'Intel 8051;
- GCC-6809 (https://github.com/bcd/gcc) per 6809 (adattamento di GCC);
- GCC-6502 (https://github.com/itszor/gcc-6502-bits) per 6502 (adattamento di GCC);
- SmallC-85 (https://github.com/ncb85/SmallC-85) per Intel 8080/8085 ;
- devkitSMS (https://github.com/sverx/devkitSMS) per Sega Master System, Sega Game Gear e Sega SG-1000.

We remark that the Z88DK dev-kit provides two compilers:
- the more reliable SCCZ80 that also offers fast compilation,
- the experimental ZSDCC (Z80-only optimized SDCC version) that can produce faster and more compact code than SCCZ80 at the cost of much slower compilation and the risk introducing erratic behavior.

Almost of considered compilers generate code for just one architecture (they are *mono-architecture*) even though they are *multi-target*. ACK is an exception because it is *multi-architecture* (Intel 8080, Intel 8088/8086, I386, 68K, MIPS, PDP11, etc.).

This article is *not* an introduction nor a manual for these compilers and it will *not* cover the following topics:
 - compiler installation
 - basic usage of the compiler
 
For details on these topics we refer to the compiler's manuals and web pages.

**A sub-set of ANSI C** 
In this article we will refer to *ANSI C* as to a large sub-set of the C89 standard where `float` and `long long` are optional but pointers to functions and pointers to `struct` are present.
We will not consider previous versions such as C in *K&R* syntax.



## Motivation
Why should we use C to code for vintage 8-bit systems?
Traditionally these systems were coded in either Assembly or interpreted BASIC or a mix of the two.
Given the limited resources, Assembly was ofter necessary. BASIC was convenient for its simplicity and because an interpreter was often present on the system.

If we limit our comparison to just Assembly, BASIC and C, the following tables summarizes the reasons of using C:

| language | simplicity |  portability |  efficiency 
|--|--|--|--|--|
| Assembly |  low | no | optimal
| BASIC    |  high |  low | low
| C        |  high |  high | good 


### Very high portability
Therefore the main reason for using ANSI C is its portability. 
In particolar ANSI C allows us:
 - to ease porting from different architectures
 - to write "universal" code, that is valid code for different targets *without* any necessary modification 

### Good performance
Someone declares C as a sort of universal Assembly language. I do not fully agree with this statement because optimally-written C will never beat optimally-written Assembly.
Nevertheless, C is the closest language to Assembly among the languages that allow high level programming.

### "Sentimental drawbacks"
One not fully rational reason for not using C in this context is the fact coding in C provides a less *vintage* experience compared to BASIC and Assembly because it was less common on the home computers from the 80s (but it was common on 8-bit business computers such as on computers that used the CP/M operating system).
A good reason for coding in C is that C allows us to code any 8-bit system. 



## Writing portable code
Writing easily portable code or even directly compilable for different architectures is possible is C through different strategies:
 - Write code that is *hardware-agnostic* through *abstract interfaces* (that is hardware-independent APIs)
 - Use different implementations of the *interfaces" and select them at compilation-time (by using *precompiler-directives* or by providing different files at linking-time)

 
### Writing portable code for targets of a dev-kit
This is trivial if our multi-target dev-kit provides a multi-target library or if we just use standard C libraries (e.g., stdio, stdlib, etc.). Under these conditions we just need to recompile our code. The multi-target library will do the "magic" for us.

Unfortunately only CC65 and Z88DK provide significant multi-target libraries for input and output other than standard C libraries:



|Dev-Kit| Architecture |  multi-target libraries
|--|--|--|
| Z88DK| Zilog Z80 |standard C lib, conio, vt52, vt100, sprite software, UDG, bitmap
| CC65| MOS 6502 | standard C lib, conio, tgi (bitmap)
| CMOC| Motorola 6809 | standard C lib
| ACK | Intel 8080 | standard C lib

In particular Z88DK has very powerful libraries for multi-target graphics and even provides APIs for software sprites (https://github.com/z88dk/z88dk/wiki/monographics) and redefined characters for most of its 80 targets.
**Example**: The code of the game H-Tron (https://sourceforge.net/projects/h-tron/) uses Z88DK's APIs for low resolution bitmap graphics for a multitude of Z80-based targets.

Therefore if we were to use exclusively the standard C libraries we could compile our code with ACK, CMOC, CC65 and Z88DK. If we used *conio* we could compile the code with CC65 and Z88DK. 

In all other cases, if we want to write portable code on different architectures and systems, we would need to write a "hardware abstraction layer" that allows us to **separate**:
 - the code that does not depend on the hardware (e.g., the logic part)
 - the code that depends on the harware (e.g., input/output in a videogame)
 
This *patterm* is very common in modern programming and it is not exclusive to C. For this purpose C provides a set of tools to implement this *pattern* in a way to select the different portions of code required by each hardware at compilation-time.
In particular C provides a powerful pre-compiler with commands such as:

 - `#define` -> to define a *macro*
 - `#if` ... `defined(...)` ... `#elif` ... `#else` -> to select code portions that depend on the existence of value of a given macro
 
Moreover all compilers provide the option `-D` to pass a macro to the pre-compiler. Some compilers such as CC65 implicitly define a macro that depends on the selected target (e.g., *__VIC20__*).

In our code we may have something like:
~~~~
...
		#elif defined(__PV1000__)
			#define XSize 28
		#elif defined(__OSIC1P__) || defined(__G800__) || defined(__RX78__) 
			#define XSize 24
		#elif defined(__VIC20__) 
			#define XSize 22
...
~~~~
When we compile for the *Vic 20* target, the pre-compiler will select for us the *Vic 20*-specific definition of `XSize`.
This also allows to select specific options for the configuration of the target (additional memory, video card, video mode, debug compilation, etc.).

As main example we refer to the *Cross-Chase* project:
https://github.com/Fabrizio-Caruso/CROSS-CHASE

The code of *Cross-Chase* provides an example of how to write *universal* code for any system and architecture:
 - the code of the game (*src/chase* directory) is hardware-independent
 - the code of the *crossLib* library  (*src/cross_lib* directory) implements all the hardware-sepcific details 

### Writing portable code even for non-supported systems

The dev-kits under our consideration support a list of targets for each architecture by providing specific libraries. Nevertheless it is possible to exploit these dev-kits for other systems with the same architecture but we will have to implement all the hardware-specific code:
 - the necessary code for input/output (e.g., graphics, sounds, keyboard, joystick, etc.)
 - the necessary code for correct machine initialization
 
Alternatively, it is possible to extend a dev-kit to support to new targets.

In many cases, we can use the ROM routines to do this (see the section on the ROM routines)

Moreover we may have to convert the binary to a format that can be acccepted by the system.

Therefore, we can indeed write portable code for even these unsupported systems.

For example CC&% does not support the *BBC Micro*, nor the *Atari 7800* and CMOC does not support the *Olivetti Prodest PC128*. Yet, it is possible to use the dev-kit to produce binaries for these targets:
 - Cross Chase (https://github.com/Fabrizio-Caruso/CROSS-CHASE) supports (theoretically) any architecture even the unsupported ones such as for example the l'Olivetti Prodest PC128.
 - The game Robotsfindskitten is been compiled for thr Atari 7800 with CC65 (https://sourceforge.net/projects/rfk7800/files/rfk7800/).
 - BBC has already been added unofficially as an experimental new target in CC65 (https://github.com/dominicbeesley/cc65).

#### Compilation for unsupported targets
We give a list of compilation options for generic target for each dev-kit so that the compiler is instructued to compile without any dependence on a specific target. For more details we refer to the manual of the dev-kits.

| Architecture |Dev-Kit  | Option(s)
|--|--|--|
| Intel 8080 |  ACK |  (*)
| MOS 6502   |  CC65| `+none` 
| Motorola 6809 | CMOC | `--nodefaultlibs` 
| Zilog 80   |  SCCZ80/ZSDCC (Z88DK) | `+test`, `+embedded` (new lib),  `+cpm` (generic CP/M target)

(\*) ACK officially only supports the CP/M-80 for the Intel 8080 architecture but it is possible to use ACK to build generic Intel 8080 binaries but it is not very simple because ACK uses a sequence of commands to produce intermediate results (including the "EM" byte-code):

 1. `ccp.ansi`:  C precompiler
 2. `em_cemcom.ansi`: compiles precompiled C code into "EM" byte-code
 3. `em_opt`: optimizes "EM" byte-code
 4.  `cpm/ncg`: generates Intel 8080 Assembly from "EM" bytecode 
 5.  `cpm/as`: generates Intel 8080 binary from Assembly 
 6. `em_led`: links object files

## General C code optimization
We describe some general rules to improve the code that do not depend on whether the architecture is 8-bit or not.

### Re-use same functions
In general, in whatever programming language we want to code, it is important to avoid code duplication and unnecessary code.

#### Structured programming
We have to examine each function in order to find common portions that we can *factor* by introducing *sub-functions* that are original function can call.
However we must take into account that, beyond a certain limit, excessive code granularity has negative effects because each function call has a computational and memory cost. 

#### Parametrized code
In some cases it is possible to generalize the code by passing a parameter to avoid writing very similar functions.
An advanced example is in https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h where, given a `struct` with two fields `_x` and `_y`,  
we want to be able to change the value of one field or the other in different situations:
~~~~
	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		...
	};
	typedef struct CharacterStruct Character;
~~~~
We avoid two different functions for `_x` and `_y` by creating one function to which we pass an *offset* to select the field:

~~~~
	unsigned char moveCharacter(Character* hunterPtr, unsigned char offset)
	{
		if((unsigned char) * ((unsigned char*)hunterPtr+offset) < ... )
		{
			++(*((unsigned char *) hunterPtr+offset));
		}
		else if((unsigned char) *((unsigned char *) hunterPtr+offset) > ... )
		{
			--(*((unsigned char *) hunterPtr+offset));
		}
	...
	}
~~~~

In this case, we use the fact that the field `_y` is exactly one byte after the field `_x`. Therefore with `offset=0` we access `_x` and with `offset=1` we access `_y`.

**Warning**: We must always remember that adding a parameter has a cost and we must verify that the cost of the parameter is lower than the cost of an extra function (e.g., by looking at the size of the obtained binary).


#### Same code on similar *objects* 
We can do even better and use the same code on *objects* that are not identical but shares some common features by using `offset` in `struct`, *pointers to functions*, etc.
In general this is possible through *object-oriented programming* whose light-weight implementation for 8-bit systems is described in a subsequent section in this article.


### Pre-increment/decrement  vs Post-increment/decrement
We must avoid post-increment/decrement operators (`i++`, `i--`) when they are not needed, i.e.,  when we do not need the original value and replace them with (`++i`, `--i`).
The reason is that the post-increment operator requires at least an extra operation to save the original value.
Remark: It is totally useless to use a post-increment in a `for` loop.

### Costant vs Variables
Any architecture will perform better if variables are replace with constants.

#### Use constants
Therefore if a variable has a known value at compilation-time, it is important to replace it with a constant.
If its value depends on some compilation option, then we should use a *macro* to set its value.

#### Help the compiler to recognize constants
Besides, for *single pass* compilers (the majority of 8-bit cross-compilers, e.g., CC65), it is important to help the compiler whether a given expression is a constant.

**_Example_** (from da https://www.cc65.org/doc/coding.html):
A *single pass* compiler may evaluate the following expression from left to right and miss the fact that `OFFS+3` is a constant:
~~~~
	#define OFFS   4
	int  i;
	i = i + OFFS + 3;
~~~~
In this case it would be better to re-write `i = i + OFFS+3` as `i = OFFS+3+i` or `i = i + (OFFS+3)`.



## Ottimizzare il codice per gli 8-bit
Il C è un linguaggio che presenta sia costrutti ad alto livello (come `struct`, le funzioni come parametri, etc.) sia costruiti a basso livello (come i puntatori e la loro manipolazione). Questo non basta per farne un linguaggio direttamente adatto alla programmazione su macchine 8-bit.

### I "tipi migliori" per gli 8-bit
Una premessa importante per la scelta dei tipi da preferire per architettura è data dal fatto che in generale abbiamo questa situazione:

 - tutte le operazioni aritmetiche sono solo a 8 bit
 - la maggior parte delle operazioni sono ad 8 bit, alcune sono a 16-bit e nessuna operazione è a 32 bit
 - le operazioni `signed` (cioè con segno) sono più lente di quelle `unsigned`
 - l'hardware non supporta operazioni in *virgola mobile*

#### Tipi interi vs tipi a virgola mobile
Il C prevede tipi numerici interi con segno (`char`, `short`, `int`, `long`, `long long` e loro equivalenti in versione `unsigned`). 
Molti compilatori (ma non CC65) prevedono il tipo `float` (numeri a *virgola mobile*) che qui non tratteremo. Bisogna considerare che i `float` delle architetture 8-bit sono tutti *software float* ed hanno quindi un costo computazionale notevole. Sarebbero quindi da usare solo se strettamente necessari.

#### Il nostro amico *unsigned* 
Siccome le architetture 8-bit che stiamo considerandno **NON** gestiscono ottimalmente tipi `signed`, dobbiamo evitare il più possibile l'uso di tipi numerici `signed`. 

#### "Size matters!"
La dimensione dei tipi numeri standard dipende dal compilatore e dall'architettura e non dal linguaggio.
Più recentemente sono stati introdotti dei tipi che fissano la dimensione in modo univoco (come per esempio `uint8_t` per l'intero `unsigend` a 8 bit).
Il modo standard per includere questi tipi a taglia fissa 
~~~~
	#include <stdint.h>
~~~~
Non tutti i compilatori 8-bit dispongono di questi tipi.

Fortunatamente per la stragrande maggioranza dei compilatori 8-bit abbiamo la seguente situazione:

|  tipo | numero bit | equivalente
|--|--|--|
| `unsigned char` | 8 | `uint8_t`
| `unsigned short` | 16 | `uint16_t`
| `unsigned int` | 16 | `uint16_t`
| `unsigned long` | 32 | `uint32_t`

Quindi dovremo:

 - usare il più possibile `unsigned char` (o `uint8_t`) per le operazioni aritmetiche;
- usare `unsigned char` (o `uint8_t`) e `unsigned short` (o `uint16_t`) per tutte le altre operazioni, evitando se possibile qualunque operazione a 32 bit.

Nota: In assenza di tipi con dimensione fissata, sarebbe una buona pratica creare dei `typedef` opportuni:
~~~~
	typedef unsigned char uint8_t;
	typedef unsigned short uint16_t;
	typedef unsigned long uint32_t;
~~~~

### Scelta delle operazioni
Quando scriviamo codice per una architettura 8-bit dobbiamo evitare se possibile codice con operazioni inefficienti o che ci obblighino a usare tipi non adatti (come i tipi `signed` o tipi a 16 o peggio 32 bit).

#### Non produciamo *signed*
In particolare, se possibile, spesso si può riscrivere il codice in maniera da evitare sottrazioni e quando questo non è possibile, si può almeno fare in modo che il risultato della sottrazione sia sempre non-negativo.

#### Evitiamo i prodotti espliciti
Tutte le architetture che abbiamo preso in considerazione, con la sola esclusione di Motorola 6809, non dispongono di una operazione per effettuare il prodotto di due valori a 8 bit.
Quindi, se possibile dobbiamo evitare i prodotti adattando il nostro codice, oppure limitarci a prodotti e divisioni che siano potenze di 2 e implementandoli con operazioni di shift con gli operatori *<<* e *>>*:
~~~~
	unsigned char foo, bar;
	...
	foo << 2; // moltiplicazione per 2^2=4
	bar >> 1; // divisione per 2^1=2
~~~~

#### Riscrivere certe operazioni
Molte operazioni come il modulo possono essere riscritte in maniera più efficiente per gli 8 bit usando operatori bit a bit. Non sempre il compilatore ottimizza nel modo migliore. Quando il compilatore non ce la fa, dobbiamo dargli una mano noi:
~~~~
	unsigned char foo;
	...
	if(foo&1) // equivalente a foo%2
	{
		...
	}
~~~~


### Variabili e parametri
Uno dei più grossi limiti dell'architettura MOS 6502 non è la penuria di registri come si potrebbe pensare ma è la dimensione limitata del suo *stack hardware* (in *pagina uno*: `$0100-01FF`) che lo rende inutilizzabile in C per la gestioni dello *scope* delle variabili e i parametri delle funzioni.
Quindi un compilatore ANSI C per 6502 sarà quasi sicuramente costretto a usare uno *stack software* per

 - gestire lo scope delle variabili,
 - gestire il passaggio dei parametri.
 
Le altre architetture 8-bit che stiamo considerando soffrono meno di questo problema ma la gestione delle scope delle variabili e dei parametri ha un costo anche quando si usa uno *stack hardware*.

#### Un *antipattern* può aiutarci
Un modo per ridurre il problema è limitare l'uso delle variabili locali e dei parametri passati alle funzioni. Questo è chiaramente un *antipattern* e se lo applicassimo a tutto il nostro codice otterremo il classico *spaghetti code*. Dobbiamo quindi scegliere sapientemente quali variabili sono assolutamente locali e quali possono essere usate come globali. Avremo codice meno generico di quello che avremmo voluto ma sarà più efficiente. **NON** sto suggerendo di rendere tutte le variabili globali e di non passare mai parametri alle funzioni.

#### [6502] Usare funzioni non re-entrant
Il compilatore CC65 per l'architettura MOS 6502 mette a disposizione l'opzione `-Cl` che rende tutte le variabili locali come `static`, quindi globali. Questo ha l'effetto di evitare l'uso dello *stack software* per il loro scope. Ha però l'effetto di rendere tutte le nostre funzioni non re-entrant. In pratica questo ci impedisce di usare funzioni recursive. Questa non è un grave perdita perché la ricorsione sarebbe comunque una operazione troppo costosa per una architettura 8-bit.

#### [6502] Usare la pagina zero
Il C standard prevede la keyword `register` per suggerire al compilatore di mettere una variabile in un registro.
In genere i compilatori moderni ignorano questa keyword perché lasciano questa scelta ai loro ottimizzatori. Questo è vero per i compilatori in questione ad eccezione di quello presenti in CC65 che la usa come suggerimento al compilatore per mettere una variabile in *pagina zero*. Il MOS 6502 accede in maniera più efficiente a tale pagina di memoria. Si può guadagnare memoria e velocità.
Per quanto riguarda l'architettura MOS 6502, il sistema operativo di queste macchine usa una parte della pagina zero. Resta comunque una manciata di byte a disposizione del programmatore. 
CC65 per default lascia 6 byte della pagina zero a disposizione delle variabili dichiarate con keyword `register`.
Potrebbe sembrare quindi ovvio dichiarare molte variabili come `register` ma **NON** è così semplice perché tutto ha un costo. Per mettere una variabile sulla *pagina zero* sono necessarie diverse operazioni. Quindi se ne avrà un vantaggio quando le variabili sono molto usate.
In pratica i due scenari in cui è conveniente sono:

 1. parametri di tipo puntatore a `struct` usati almeno 3 volte all'interno di una funzione
 2. variabile in un loop che si ripete almeno un centinaio di volte

Un riferimento più preciso è dato da: https://www.cc65.org/doc/cc65-8.html

Il mio consiglio è quello di compilare e vedere se il binario è divenuto più breve.

### Struttura ottimale del binario
Se il nostro programma prevede dei dati in una definita area di memoria, sarebbe meglio metterli direttamente nel binario che verrà copiato in memoria durante il caricamento. Se questi dati sono invece nel codice, saremo costretti a scrivere del codice che li copia nell'area di memoria in cui sono previsti.
Il caso più comune è forse quello degli sprites e dei caratteri/tiles ridefiniti.

Spesso (ma non sempre) le architetture basate su MOS 6502 prevedono video *memory mapped* in cui i dati della grafica si trovano nella stessa RAM a cui accede la CPU.

Molte architetture basate su Z80 (MSX, Spectravideo, Memotech, Tatung Einstein, etc.) usano il chip Texas VDP che invece ha una memoria video dedicata. Quindi non potremo mettere la grafica direttamente in questa memoria.

#### [CC65] Istruiamo il linker
Ogni compilatore mette a disposizioni strumenti diversi per definire la struttura del binario e quindi permetterci di costruirlo in maniera che i dati siano caricati in una determinata zona di memoria durante il load del programma senza uso di codice aggiuntivo.
In particolare su CC65 si può usare il file .cfg di configurazione del linker che descrive la struttura del binario che vogliamo produrre.
Il linker di CC65 non è semplicissimo da configurare ed una descrizione andrebbe oltre lo scopo di questo articolo.
Una descrizione dettagliata è presente su:
https://cc65.github.io/doc/ld65.html
Il mio consiglio è di leggere il manuale e di modificare i file di default .cfg già presenti in CC65 al fine di adattarli al proprio use-case.

##### Exomizer ci aiuta (anche) in questo caso
In alcuni casi se la nostra grafica deve trovarsi in un'area molto lontana dal codice, e vogliamo creare un unico binario, avremo un binario enorme e con un "buco". Questo è il caso per esempio del C64 in cui la grafica per caratteri e sprites può trovarsi lontana dal codice. In questo caso io suggerisco di usare *exomizer* sul risultato finale: https://bitbucket.org/magli143/exomizer/wiki/Home

#### [Z88DK] *Appmake* fa (quasi) tutto per noi
Z88DK fa molto di più e il suo potente tool *appmake* costuisce dei binari nel formato richiesto.
Z88DK consente comunque all'utente di definire sezioni di memoria e di definire il "packaging" del binario ma non è semplice.
Questo argomento è trattato in dettaglio in
https://github.com/z88dk/z88dk/issues/860



### Codice su file diversi?
In generale è bene separare in più file il proprio codice se il progetto è di grosse dimensioni.
Questa buona pratica può però avere degli effetti deleteri per gli ottimizzatori dei compilatori 8-bit perché in generale non eseguono *link-time optimization*, cioè non ottimizzeranno codice tra più file ma si limitano ad ottimizzare ogni file singolarmente.
Quindi se per esempio abbiamo una funzione che chiamiamo una sola volta e la funzione è definita nello stesso file in cui viene usata, l'ottimizzatore potre metterla *in line* ma non lo farà se la funzione è definita in un altro file.
Il mio consiglio **non** quello di creare file enormi con tutto ma è quello di tenere comunque conto di questo aspetto quando si decide di separare il codice su più file e di non abusare di questa buona pratica.


## Uso avanzato della memoria
Il compilatore C in genere produrrà un unico binario che conterrà codice e dati che verranno caricati in una specifica zona di memoria (è comunque possibile avere porzioni di codice non contigue).

In molte architetture alcune aree della memoria RAM sono usate come *buffer* oppure sono dedicate a usi specifici come alcune modalità grafiche.
Il mio consiglio è quindi di studiare le mappa della memoria di ogni hardware per trovare queste preziose aree. 
Per esempio per il Vic 20: http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt

In particolare consiglio di cercare:

- buffer della cassetta, della tastiera, della stampante, del disco, etc.
- memoria usata dal BASIC
- aree di memoria dedicate a modi grafici (che non si intendono usare)
- aree di memoria libere ma non contigue e che quindi non sarebbero parte del nostro binario

Queste aree di memoria potrebbero essere sfruttate dal nostro codice se nel nostro use-case non servono per il loro scopo originario (esempio: se non intendiamo caricare da cassetta dopo l'avvio del programma, possiamo usare il buffer della cassetta per metterci delle variabili da usare dopo l'avvio potendolo comunque usare prima dell'avvio per caricare il nostro stesso programma da cassetta).

*Esempi utili*
In questa tabella diamo alcuni esempi utili per sistemi che hanno poca memoria disponibile:
|computer|descrizione|area|
|--|--|--|
|Commodore 16|tape buffer| $0333-03F2
|Commodore 16|BASIC input buffer | $0200-0258
|Commodore Pet| system input buffer| $0200-0250
|Commodore Pet | tape buffer | $033A-03F9
|Commodore Vic 20 |tape buffer |$033C-03FB |
|Commodore Vic 20 | BASIC input buffer | $0200-0258
|Galaksija | variable a-z | $2A00-2A68
|Sinclair Spectrum 16K/48K | printer buffer | $5B00-5BFF
| Mattel Aquarius | random number space | $381F-3844
| Mattel Aquarius | input buffer | $3860-38A8
| Oric | alternate charset | $B800-B7FF |
| Oric | grabable memory per modo hires| $9800-B3FF 
| Oric | Page 4 | $0400-04FF
| TRS-80 Model I/III/IV | RAM for ROM routines (*) | $4000-41FF
|VZ200| printer buffer & misc| $7930-79AB
|VZ200| BASIC line input buffer | $79E8-7A28

(*): Diversi tipi di buffer e memoria ausiliare. Per maggiori dettagli fare riferimento a: http://www.trs-80.com/trs80-zaps-internals.htm

In C standard potremmo solo definire le variabili puntatore e gli array come locazioni in queste aree di memoria.

Di seguito diamo un esempio di mappatura delle variabili a partire da `0xC000` in cui abbiamo definito uno `struct` di tipo `Character` che occupa 5 byte, e abbiamo le seguenti variabili:

 - `player` di tipo `Character`, 
 - `ghosts` di tipo `array` di 8 `Character` (40=$28 byte)
 - `bombs` di tipo array di 4 `Character` (20=$14 byte)
~~~~
	Character *ghosts = 0xC000;
	Character *bombs = 0xC000+$28;
	Character *player = 0xC000+$28+$14;
~~~~
Questa soluzione generica con puntatori non sempre produce il codice ottimale perché obbliga a fare diverse *deferenziazioni* e comunque crea delle variabili puntatore (ognuna delle quali dovrebbe occupare 2 byte) che il compilatore potrebbe comunque allocare in memoria.

Non esiste un modo standard per dire al compilatore di mettere qualunque tipo di variabile in una specifica area di memoria.
I compilatori di CC65 e Z88DK invece prevedono una sintassi per permetterci di fare questo e guadagnare diverse centinaia o migliaia di byte preziosi.
Vari esempi sono presenti in:
https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory

In particolare bisogna creare un file Assembly .s (con CC65) o .asm (con Z88DK) da linkare al nostro eseguibile in cui assegnamo un indirizzo ad ogni nome di variabile a cui  **aggiungiamo** un prefisso *underscore*.


Sintassi CC65 (esempio Vic 20)
```
	.export _ghosts;
	_ghosts = $33c
	.export _bombs;
	_bombs = _ghosts + $28 
	.export _player;
	_player = _bombs + $14
```

Sintassi Z88DK (esempio Galaksija)
```
	PUBLIC _ghosts, _bombs, _player
	defc _ghosts = 0x2A00
	defc _bombs = _ghosts + $28 
	defc _player = _bombs + $14
```

CMOC mette a disposizione l'opzione `--data=<indirizzo>` che permette di allocare tutte le variabili globali scrivibili a partire da un indirizzo dato. 

La documentazione di ACK non dice nulla a riguardo. Potremo comunque definire i tipi puntatore e gli array nelle zone di memoria libera.



## La programmazione ad oggetti
Contrariamente a quello che si possa credere, la programmazione ad oggetti è possibile in ANSI C e può aiutarci a produrre codice più compatto in alcune situazioni. Esistono interi framework ad oggetti che usano ANSI C (es. Gnome è scritto usando *GObject* che è uno di questi framework).

Nel caso delle macchine 8-bit con vincoli di memoria molto forti, possiamo comunque implementare *classi*, *polimorfismo* ed *ereditarietà* in maniera molto efficiente.
Una trattazione dettagliata non è possibile in questo articolo e qui ci limitiamo a citare i due strumenti fondamentali:

- usare *puntatori a funzioni* per ottenere metodi *polimorfici*, cioè il cui *binding* (e quindi comportamento) è dinamicamente definito a *run-time*. Si può evitare l'implementazione di una *vtable* se ci si limita a classi con un solo metodo polimorfico.
- usare *puntatori a* `struct` e *composizione* per implementare sotto-classi: dato uno `struct` A, si implementa una sua sotto-classe con uno `struct` B definito come uno `struct` il cui **primo** campo è A. Usando puntatori a tali `struct`, il C garantisce che gli *offset* di B siano gli stessi degli offset di A. 


Esempio (preso da
https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase)
Definiamo `Item` come un sotto-classe di `Character` a cui aggiungiamo delle variabili ed il metodo polimorfico `_effect()`:
~~~~
	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		unsigned char _status;
		Image* _imagePtr;
	};
	typedef struct CharacterStruct Character;
...
 	struct ItemStruct
	{
		Character _character;
		void (*_effect)(void);
		unsigned short _coolDown;
		unsigned char _blink;
	};
	typedef struct ItemStruct Item;
~~~~
e poi potremo passare un puntatore a `Item` come se fosse un puntatore a `Character` (facendo un semplice *cast*):
~~~~
	Item *myItem;
	void foo(Character * aCharacter);
	...
	foo((Character *)myItem);
~~~~

Perché ci guadagniamo in termine di memoria?
Perché sarà possibile trattare più oggetti con lo stesso codice e quindi risparmiamo memoria.

## Compilazione ottimizzata
Non tratteremo in modo esaustivo le opzioni di compilazione dei cross-compilatori e consigliamo di fare riferimento ai loro rispettivi manuali per dettagli avanzati. Qui daremo una lista delle opzioni per compilare codice ottimizzato su ognuno dei compilatori che stiamo trattando. 

### Ottimizzazione "aggressiva" 
Le seguenti opzioni applicano il massimo delle ottimizzazioni per produrre codice veloce e soprattutto compatto:

| Architettura |Compilatore | Opzioni|
|--|--|--|
| Intel 8080 |  ACK |  `-O6` |
| Zilog Z80   |  SCCZ80 (Z88DK) | `-O3` |
| Zilog Z80   |  ZSDCC (Z88DK) | `-SO3` `--max-alloc-node20000`|
| MOS 6502   |  CC65| `-O` `-Cl`|
| Motorola 6809 | CMOC | `-O2` |

#### Velocità vs Memoria
In generale su molti target 8-bit il problema maggiore è la presenza di poca memoria per codice e dati. In generale il codice ottimizzato per la velocità sarà sia compatto che veloce ma non sempre le due cose andranno assieme. 
In alcuni altri casi l'obiettivo principale può essere la velocità anche a discapito della memoria.
Alcuni compilatori mettono a disposizioni delle opzioni per specificare la propria preferenza tra velocità e memoria:

| Architettura |Compilatore | Opzioni| Descrizione
|--|--|--|--|
| Zilog Z80   |  ZSDCC (Z88DK) | `--opt-code-size`| Ottimizza memoria
| Zilog Z80   |  SCCZ80 (Z88DK) | `--opt-code-speed`| Ottimizza velocità
| MOS 6502   |  CC65| `-Oi`, `-Os`| Ottimizza velocità



**Problemi noti**
- CC65: `-Cl` impedisce la ricorsione 
- CMOC: `-O2` ha dei bug
- ZSDCC: ha dei bug a prescindere dalle opzioni e ne ha altri presenti con `-SO3` in assenza di `--max-alloc-node20000`.


### Ottimizzazione più sicura 
Per ovviare a i problemi sopramenzionati e ridurre i tempi di compilazione (soprattutto per l'architettura Z80) si consiglia:

| Architettura |Compilatore | Opzioni|
|--|--|--|
| Zilog Z80   |  SCCZ80 (Z88DK) | `-O3` |
| MOS 6502   |  CC65| `-O`|
| Motorola 6809 | CMOC | `-O1` |

## Evitare il linking di codice inutile
I compilatori che trattiamo non sempre saranno capaci di eliminare il codice non usato. Dobbiamo quindi evitare di includere codice non utile per essere sicuri che non finisca nel binario prodotto.

Possiamo fare ancora meglio con alcuni dei nostri compilatori, istruendoli a non includere alcune librerie standard o persino alcune loro parti se siamo sicuri di non doverle usare.

### Evitare la standard lib
Evitare nel proprio codice la libraria standard nei casi in cui avrebbe senso, può ridurre la taglia del codice in maniera considerevole. 

#### [cp/m-80] Solo *getchar()* e *putchar\(c\)*
Questa regola è generale ma è particolarmente valida quando si usa ACK per produrre un binario per CP/M-80. In questo caso consiglio di usare esclusivamente `getchar()` e `putchar(c)` e implementare tutto il resto.

#### [z88dk] Pragmas per non generare codice
Z88DK mette a disposizione una serie di *pragma* per istruire il compilatore a non generare codice inutile.

Per esempio:
~~~~
#pragma printf = "%c %u"
~~~~
includerà solo i convertitori per `%c` e `%u` escludendo tutto il codice per gli altri.

~~~~
#pragma-define:CRT_INITIALIZE_BSS=0
~~~~
non genera codice per l'inizializzazione dell'area di memoria BSS.


~~~~
#pragma output CRT_ON_EXIT = 0x10001
~~~~
il programma non fa nulla alla sua uscita (non gestisce il ritorno al BASIC)


~~~~
#pragma output CLIB_MALLOC_HEAP_SIZE = 0
~~~~
elimina lo heap della memoria dinamica (nessuna malloc possibile)


~~~~
#pragma output CLIB_STDIO_HEAP_SIZE = 0
~~~~
elimina lo heap di stdio (non gestisce l'apertura di file)

Alcuni esempi sono in 
https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk


## Usare le routine presenti in ROM
La stragrande maggioranza dei sistemi 8-bit (quasi tutti i computer) prevede svariate routine nelle ROM. E' quindi importante conoscerle per usarle. Per usarle esplicitamente dovremo scrivere del codice Assembly da richiamare da C. Il modo d'uso dell'Assembly assieme al C può avvenire in modo *in line* (codice Assembly integrato all'interno di funzioni C) oppure con file separati da linkare al C ed è diverso in ogni dev-kit. Per i dettagli consigliamo di leggere i manuali dei vari dev-kit.

Questo è molto importante per i sistemi che non sono (ancora) supportati dai compilatori e per i quali bisogna scrivere da zero tutte le routine per l'input/output.

Esempio (preso da https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c)

Per il display di caratteri sullo schermo per i Thomson Mo5, Mo6 e Olivetti Prodest PC128 (sistemi non supportati da nessun compilatore) piuttosto che scrivere una routine da zero possiamo affidarci ad una routine Assembly presente nella ROM:
~~~~
	void PUTCH(unsigned char ch)
	{
		asm
		{
			ldb ch
			swi
			.byte 2
		}
	}
~~~~

#### Le librerie spesso lo fanno per noi
Fortunatamente spesso potremo usare le routine della ROM implicitamente senza fare alcuna fatica perché le librerie di supporto ai target dei nostri dev-kit, lo fanno già per noi. Usare una routine della ROM ci fa risparmiare codice ma può imporci dei vincoli perché per esempio potrebbero non fare esattamente quello che vogliamo oppure usano alcune aree della RAM (buffer) che noi potremmo volere usare in modo diverso.




## Sfruttare l'hardware specifico
Come visto nelle sezioni precedenti, anche se programmiamo in C non dobbiamo dimenticare l'hardware specifico per il quale stiamo scrivendo del codice.
In alcuni casi conoscere l'hardware può aiutarci a scrivere codice molto più compatto e/o più veloce.

### Usare le estensioni ASCII specifiche
Per esempio, è inutile ridefinire dei caratteri per fare della grafica se il sistema dispone già di caratteri utili al nostro scopo sfruttando l'estensione specifica dei caratteri ASCII (ATASCII, PETSCII, SHARPSCII, etc.).

### Sfruttare i chip grafici
Conoscere il chip grafico può aiutarci a risparmiare tanta ram.

Esempio (Chip della serie VDP tra cui il TMS9918A presente su MSX, Spectravideo, Memotech MTX, Sord M5, etc.)
I sistemi basati su questo chip prevedono una modalità video testuale (*Mode 1*)  in cui il colore del carattere è implicitamente dato dal codice del carattere. Se usiamo questo speciale modo video, sarà quindi sufficiente un singolo byte per definire il carattere ed il suo colore con un notevole risparmio in termini di memoria.

Esempio (Commodore Vic 20)
Il Commodore Vic 20 è un caso veramente speciale perché prevede dei limiti hardware (RAM totale: 5k, RAM disponibile per il codice: 3,5K) ma anche dei trucchi per superarli almeno in parte:

- In realtà dispone anche di 1024 nibble di RAM aggiuntiva speciale per gli attributi colore
- Pur avendo soltanto 3,5k di memoria RAM contigua per il codice, molta altra RAM è facilmente sfruttabile per dati (buffer cassetta, buffer comando INPUT del BASIC)
- La caratteristica più sorprendente è che il chip grafico VIC può mappare una parte dei caratteri in RAM lasciandone metà definiti dalla ROM

Quindi, sfruttiamo implicitamente la prima caratteristica accedendo ai colori senza usare la RAM comune.
Possiamo mappare le nostre variabili nei vari buffer non utilizzati.
Se ci bastano n (n<=64) caratteri ridefiniti possiamo mapparne solo 64 con `POKE(0x9005,0xFF);` Ne potremo usare anche meno di 64 lasciando il resto per il codice ma mantenendo in aggiunta 64 caratteri standard senza alcun dispendio di memoria per i caratteri standard.

