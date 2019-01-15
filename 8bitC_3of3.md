---


---

<p><img src="8bitC.jpg" alt="8-bit C"></p>
<h1 id="c-portabile-e-ottimizzato-per-gli-8-bit">C portabile e ottimizzato per gli 8-bit</h1>
<h2 id="terza-parte-tecniche-avanzate-per-ottimizzare-il-codice-c-per-8-bit"><em>Terza parte</em>: Tecniche avanzate per ottimizzare il codice C per 8-bit</h2>
<p>Questa è la terza parte di una serie di tre articoli che descrivono tecniche per scrivere codice portabile e ottimizzato in ANSI C per <strong>tutti</strong> i sistemi 8-bit <em>vintage</em>, cioè computer, console, handheld, calcolatrici scientifiche e microcontrollori dalla fine degli anni 70 fino a metà degli anni 90.<br>
L’articolo completo è disponibile on-line su <a href="https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md">https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md</a></p>
<p>Consigliamo la lettura dei primi due articoli in cui abbiamo presentato i vari cross compilatori C, abbiamo dato alcune indicazioni su come scrivere codice C portabile e ottimizzato su tutte le architetture 8 bit.</p>
<h2 id="uso-avanzato-della-memoria">Uso avanzato della memoria</h2>
<p>Il compilatore C in genere produrrà un unico binario che conterrà codice e dati che verranno caricati in una specifica zona di memoria (è comunque possibile avere porzioni di codice non contigue).</p>
<p>In molte architetture alcune aree della memoria RAM sono usate come <em>buffer</em> oppure sono dedicate a usi specifici come alcune modalità grafiche.<br>
Il mio consiglio è quindi di studiare le mappa della memoria di ogni hardware per trovare queste preziose aree.<br>
Per esempio per il Vic 20: <a href="http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt">http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt</a></p>
<p>In particolare consiglio di cercare:</p>
<ul>
<li>buffer della cassetta, della tastiera, della stampante, del disco, etc.</li>
<li>memoria usata dal BASIC</li>
<li>aree di memoria dedicate a modi grafici (che non si intendono usare)</li>
<li>aree di memoria libere ma non contigue e che quindi non sarebbero parte del nostro binario</li>
</ul>
<p>Queste aree di memoria potrebbero essere sfruttate dal nostro codice se nel nostro use-case non servono per il loro scopo originario (esempio: se non intendiamo caricare da cassetta dopo l’avvio del programma, possiamo usare il buffer della cassetta per metterci delle variabili da usare dopo l’avvio potendolo comunque usare prima dell’avvio per caricare il nostro stesso programma da cassetta).</p>
<p><em>Esempi utili</em><br>
In questa tabella diamo alcuni esempi utili per sistemi che hanno poca memoria disponibile:</p>

<table>
<thead>
<tr>
<th>computer</th>
<th>descrizione</th>
<th>area</th>
</tr>
</thead>
<tbody>
<tr>
<td>Commodore 16</td>
<td>tape buffer</td>
<td>$0333-03F2</td>
</tr>
<tr>
<td>Commodore 16</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore 64 &amp; Vic 20</td>
<td>tape buffer</td>
<td>$033C-03FB</td>
</tr>
<tr>
<td>Commodore 64 &amp; Vic 20</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore Pet</td>
<td>system input buffer</td>
<td>$0200-0250</td>
</tr>
<tr>
<td>Commodore Pet</td>
<td>tape buffer</td>
<td>$033A-03F9</td>
</tr>
<tr>
<td>Galaksija</td>
<td>variable a-z</td>
<td>$2A00-2A68</td>
</tr>
<tr>
<td>Sinclair Spectrum 16K/48K</td>
<td>printer buffer</td>
<td>$5B00-5BFF</td>
</tr>
<tr>
<td>Mattel Aquarius</td>
<td>random number space</td>
<td>$381F-3844</td>
</tr>
<tr>
<td>Mattel Aquarius</td>
<td>input buffer</td>
<td>$3860-38A8</td>
</tr>
<tr>
<td>Oric</td>
<td>alternate charset</td>
<td>$B800-B7FF</td>
</tr>
<tr>
<td>Oric</td>
<td>grabable memory per modo hires</td>
<td>$9800-B3FF</td>
</tr>
<tr>
<td>Oric</td>
<td>Page 4</td>
<td>$0400-04FF</td>
</tr>
<tr>
<td>Sord M5</td>
<td>RAM for ROM routines (*)</td>
<td>$7000-73FF</td>
</tr>
<tr>
<td>TRS-80 Model I/III/IV</td>
<td>RAM for ROM routines (*)</td>
<td>$4000-41FF</td>
</tr>
<tr>
<td>VZ200</td>
<td>printer buffer &amp; misc</td>
<td>$7930-79AB</td>
</tr>
<tr>
<td>VZ200</td>
<td>BASIC line input buffer</td>
<td>$79E8-7A28</td>
</tr>
</tbody>
</table><p>(*): Multiple buffer and auxiliary ram for ROM routiens. For more details please refer to:<br>
<a href="http://m5.arigato.cz/m5sysvar.html">http://m5.arigato.cz/m5sysvar.html</a> and <a href="http://www.trs-80.com/trs80-zaps-internals.htm">http://www.trs-80.com/trs80-zaps-internals.htm</a></p>
<p>In C standard potremmo solo definire le variabili puntatore e gli array come locazioni in queste aree di memoria.</p>
<p>Di seguito diamo un esempio di mappatura delle variabili a partire da <code>0xC000</code> in cui abbiamo definito uno <code>struct</code> di tipo <code>Character</code> che occupa 5 byte, e abbiamo le seguenti variabili:</p>
<ul>
<li><code>player</code> di tipo <code>Character</code>,</li>
<li><code>ghosts</code> di tipo <code>array</code> di 8 <code>Character</code> (40=$28 byte)</li>
<li><code>bombs</code> di tipo array di 4 <code>Character</code> (20=$14 byte)</li>
</ul>
<pre><code>	Character *ghosts = 0xC000;
	Character *bombs = 0xC000+$28;
	Character *player = 0xC000+$28+$14;
</code></pre>
<p>Questa soluzione generica con puntatori non sempre produce il codice ottimale perché obbliga a fare diverse <em>deferenziazioni</em> e comunque crea delle variabili puntatore (ognuna delle quali dovrebbe occupare 2 byte) che il compilatore potrebbe comunque allocare in memoria.</p>
<p>Non esiste un modo standard per dire al compilatore di mettere qualunque tipo di variabile in una specifica area di memoria.<br>
I compilatori di CC65 e Z88DK invece prevedono una sintassi per permetterci di fare questo e guadagnare diverse centinaia o migliaia di byte preziosi.<br>
Vari esempi sono presenti in:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory</a></p>
<p>In particolare bisogna creare un file Assembly .s (con CC65) o .asm (con Z88DK) da linkare al nostro eseguibile in cui assegnamo un indirizzo ad ogni nome di variabile a cui  <strong>aggiungiamo</strong> un prefisso <em>underscore</em>.</p>
<p>Sintassi CC65 (esempio Vic 20)</p>
<pre><code>	.export _ghosts;
	_ghosts = $33c
	.export _bombs;
	_bombs = _ghosts + $28 
	.export _player;
	_player = _bombs + $14
</code></pre>
<p>Sintassi Z88DK (esempio Galaksija)</p>
<pre><code>	PUBLIC _ghosts, _bombs, _player
	defc _ghosts = 0x2A00
	defc _bombs = _ghosts + $28 
	defc _player = _bombs + $14
</code></pre>
<p>CMOC mette a disposizione l’opzione <code>--data=&lt;indirizzo&gt;</code> che permette di allocare tutte le variabili globali scrivibili a partire da un indirizzo dato.</p>
<p>La documentazione di ACK non dice nulla a riguardo. Potremo comunque definire i tipi puntatore e gli array nelle zone di memoria libera.</p>
<h2 id="compilazione-ottimizzata">Compilazione ottimizzata</h2>
<p>Non tratteremo in modo esaustivo le opzioni di compilazione dei cross-compilatori e consigliamo di fare riferimento ai loro rispettivi manuali per dettagli avanzati. Qui daremo una lista delle opzioni per compilare codice ottimizzato su ognuno dei compilatori che stiamo trattando.</p>
<h3 id="ottimizzazione-aggressiva">Ottimizzazione “aggressiva”</h3>
<p>Le seguenti opzioni applicano il massimo delle ottimizzazioni per produrre codice veloce e soprattutto compatto:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td><code>-O6</code></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>-O3</code></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><code>-compiler=sdcc</code> <code>-SO3</code> <code>--max-alloc-node20000</code></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-O</code> <code>-Cl</code></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>-O2</code></td>
</tr>
</tbody>
</table><h4 id="velocità-vs-memoria">Velocità vs Memoria</h4>
<p>In generale su molti target 8-bit il problema maggiore è la presenza di poca memoria per codice e dati. In generale il codice ottimizzato per la velocità sarà sia compatto che veloce ma non sempre le due cose andranno assieme.<br>
In alcuni altri casi l’obiettivo principale può essere la velocità anche a discapito della memoria.<br>
Alcuni compilatori mettono a disposizioni delle opzioni per specificare la propria preferenza tra velocità e memoria:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
<th>Descrizione</th>
</tr>
</thead>
<tbody>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><code>-compiler=sdcc</code> <code>--opt-code-size</code></td>
<td>Ottimizza memoria</td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>--opt-code-speed</code></td>
<td>Ottimizza velocità</td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-Oi</code>, <code>-Os</code></td>
<td>Ottimizza velocità</td>
</tr>
</tbody>
</table><p><strong>Problemi noti</strong></p>
<ul>
<li>CC65: <code>-Cl</code> impedisce la ricorsione</li>
<li>ZSDCC: ha dei bug a prescindere dalle opzioni e ne ha altri presenti con <code>-SO3</code> in assenza di <code>--max-alloc-node20000</code>.</li>
</ul>
<p>Per ovviare a questi problemi e ridurre i tempi di compilazione (soprattutto per l’architettura Z80) si consiglia:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
</tr>
</thead>
<tbody>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>-O3</code></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-O</code></td>
</tr>
</tbody>
</table><h2 id="evitare-il-linking-di-codice-inutile">Evitare il linking di codice inutile</h2>
<p>I compilatori che trattiamo non sempre saranno capaci di eliminare il codice non usato. Dobbiamo quindi evitare di includere codice non utile per essere sicuri che non finisca nel binario prodotto.</p>
<p>Possiamo fare ancora meglio con alcuni dei nostri compilatori, istruendoli a non includere alcune librerie standard o persino alcune loro parti se siamo sicuri di non doverle usare.</p>
<h3 id="evitare-la-standard-lib">Evitare la standard lib</h3>
<p>Evitare nel proprio codice la libraria standard nei casi in cui avrebbe senso, può ridurre la taglia del codice in maniera considerevole.</p>
<h4 id="cpm-80-solo-getchar-e-putcharc">[cp/m-80] Solo <em>getchar()</em> e <em>putchar(c)</em></h4>
<p>Questa regola è generale ma è particolarmente valida quando si usa ACK per produrre un binario per CP/M-80. In questo caso consiglio di usare esclusivamente <code>getchar()</code> e <code>putchar(c)</code> e implementare tutto il resto.</p>
<h4 id="z88dk-pragmas-per-non-generare-codice">[z88dk] Pragmas per non generare codice</h4>
<p>Z88DK mette a disposizione una serie di <em>pragma</em> per istruire il compilatore a non generare codice inutile.</p>
<p>Per esempio:</p>
<pre><code>#pragma printf = "%c %u"
</code></pre>
<p>includerà solo i convertitori per <code>%c</code> e <code>%u</code> escludendo tutto il codice per gli altri.</p>
<pre><code>#pragma-define:CRT_INITIALIZE_BSS=0
</code></pre>
<p>non genera codice per l’inizializzazione dell’area di memoria BSS.</p>
<pre><code>#pragma output CRT_ON_EXIT = 0x10001
</code></pre>
<p>il programma non fa nulla alla sua uscita (non gestisce il ritorno al BASIC)</p>
<pre><code>#pragma output CLIB_MALLOC_HEAP_SIZE = 0
</code></pre>
<p>elimina lo heap della memoria dinamica (nessuna malloc possibile)</p>
<pre><code>#pragma output CLIB_STDIO_HEAP_SIZE = 0
</code></pre>
<p>elimina lo heap di stdio (non gestisce l’apertura di file)</p>
<p>Alcuni esempi sono in<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk</a></p>
<h2 id="usare-le-routine-presenti-in-rom">Usare le routine presenti in ROM</h2>
<p>La stragrande maggioranza dei sistemi 8-bit (quasi tutti i computer) prevede svariate routine nelle ROM. E’ quindi importante conoscerle per usarle. Per usarle esplicitamente dovremo scrivere del codice Assembly da richiamare da C. Il modo d’uso dell’Assembly assieme al C può avvenire in modo <em>in line</em> (codice Assembly integrato all’interno di funzioni C) oppure con file separati da linkare al C ed è diverso in ogni dev-kit. Per i dettagli consigliamo di leggere i manuali dei vari dev-kit.</p>
<p>Questo è molto importante per i sistemi che non sono (ancora) supportati dai compilatori e per i quali bisogna scrivere da zero tutte le routine per l’input/output.</p>
<p>Esempio (preso da <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c</a>)</p>
<p>Per il display di caratteri sullo schermo per i Thomson Mo5, Mo6 e Olivetti Prodest PC128 (sistemi non supportati da nessun compilatore) piuttosto che scrivere una routine da zero possiamo affidarci ad una routine Assembly presente nella ROM:</p>
<pre><code>	void PUTCH(unsigned char ch)
	{
		asm
		{
			ldb ch
			swi
			.byte 2
		}
	}
</code></pre>
<h4 id="le-librerie-spesso-lo-fanno-per-noi">Le librerie spesso lo fanno per noi</h4>
<p>Fortunatamente spesso potremo usare le routine della ROM implicitamente senza fare alcuna fatica perché le librerie di supporto ai target dei nostri dev-kit, lo fanno già per noi. Usare una routine della ROM ci fa risparmiare codice ma può imporci dei vincoli perché per esempio potrebbero non fare esattamente quello che vogliamo oppure usano alcune aree della RAM (buffer) che noi potremmo volere usare in modo diverso.</p>
<h2 id="sfruttare-lhardware-specifico">Sfruttare l’hardware specifico</h2>
<p>Come visto nelle sezioni precedenti, anche se programmiamo in C non dobbiamo dimenticare l’hardware specifico per il quale stiamo scrivendo del codice.<br>
In alcuni casi conoscere l’hardware può aiutarci a scrivere codice molto più compatto e/o più veloce.</p>
<h3 id="usare-le-estensioni-ascii-specifiche">Usare le estensioni ASCII specifiche</h3>
<p>Per esempio, è inutile ridefinire dei caratteri per fare della grafica se il sistema dispone già di caratteri utili al nostro scopo sfruttando l’estensione specifica dei caratteri ASCII (ATASCII, PETSCII, SHARPSCII, etc.).</p>
<h3 id="sfruttare-i-chip-grafici">Sfruttare i chip grafici</h3>
<p>Conoscere il chip grafico può aiutarci a risparmiare tanta ram.</p>
<p>Esempio (Chip della serie VDP tra cui il TMS9918A presente su MSX, Spectravideo, Memotech MTX, Sord M5, etc.)<br>
I sistemi basati su questo chip prevedono una modalità video testuale (<em>Mode 1</em>)  in cui il colore del carattere è implicitamente dato dal codice del carattere. Se usiamo questo speciale modo video, sarà quindi sufficiente un singolo byte per definire il carattere ed il suo colore con un notevole risparmio in termini di memoria.</p>
<p>Esempio (Commodore Vic 20)<br>
Il Commodore Vic 20 è un caso veramente speciale perché prevede dei limiti hardware (RAM totale: 5k, RAM disponibile per il codice: 3,5K) ma anche dei trucchi per superarli almeno in parte:</p>
<ul>
<li>In realtà dispone anche di 1024 nibble di RAM aggiuntiva speciale per gli attributi colore</li>
<li>Pur avendo soltanto 3,5k di memoria RAM contigua per il codice, molta altra RAM è facilmente sfruttabile per dati (buffer cassetta, buffer comando INPUT del BASIC)</li>
<li>La caratteristica più sorprendente è che il chip grafico VIC può mappare una parte dei caratteri in RAM lasciandone metà definiti dalla ROM</li>
</ul>
<p>Quindi, sfruttiamo implicitamente la prima caratteristica accedendo ai colori senza usare la RAM comune.<br>
Possiamo mappare le nostre variabili nei vari buffer non utilizzati.<br>
Se ci bastano n (n&lt;=64) caratteri ridefiniti possiamo mapparne solo 64 con <code>POKE(0x9005,0xFF);</code> Ne potremo usare anche meno di 64 lasciando il resto per il codice ma mantenendo in aggiunta 64 caratteri standard senza alcun dispendio di memoria per i caratteri standard.</p>

