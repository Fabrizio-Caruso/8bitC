---


---

<h1 id="c-per-gli-8-bit">C per gli 8-bit</h1>
<p>Questo articolo descrive alcune tecniche e consigli per sfruttare al massimo il linguaggio ANSI C su sistemi 8-bit <em>vintage</em>, cioè computer, console, handheld, calcolatrici scientifiche dalla fine degli anni 70 fino a metà degli anni 90 ed in particolare sistemi basati sulle seguenti architetture (e architetture derivate e retrocompatibili):</p>
<ul>
<li>Intel 8080</li>
<li>Zilog Z80</li>
<li>MOS 6502</li>
<li>Motorola 6809</li>
</ul>
<h2 id="cross-compilatori-multi-target">Cross-compilatori multi-target</h2>
<p>Per produrre i nostri binari 8-bit useremo dei <em>cross compilatori</em> <em>multi-target</em> (cioè compilatori eseguiti su PC che producono binari per diversi sistemi come computer, console, handheld, calcolatrici, sistemi embedded, etc.).  Non useremo dei compilatori <em>nativi</em> perché sarebbero molto scomodi (anche se usati all’interno di un emulatore accellerato al massimo) e non potrebbero mai produrre codice ottimizzato perché l’ottimizzatore sarebbe limitato dalla risorse della macchina 8-bit.</p>
<p>Faremo particolare riferimento ai seguenti <em>cross compilatori</em> <em>multi-target</em>:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore/Dev-Kit</th>
<th>Pagina</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td><a href="https://github.com/davidgiven/ack">https://github.com/davidgiven/ack</a></td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td><a href="https://github.com/z88dk/z88dk">https://github.com/z88dk/z88dk</a></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><a href="https://github.com/cc65/cc65">https://github.com/cc65/cc65</a></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><a href="https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html">https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html</a></td>
</tr>
</tbody>
</table><p>Inoltre esistono altri <em>cross compilatori</em> C <em>multi-target</em> che non tratteremo qui come per esempio</p>
<ul>
<li>SDCC (<a href="http://sdcc.sourceforge.net/">http://sdcc.sourceforge.net/</a>) per svariate architetture di microprocessori come lo Z80 e di microcontrollori come l’Intel 8051;</li>
<li>versioni speciali di GCC per architetture 8-bit (GCC6809, GCC6502).</li>
</ul>
<p>Si noti come il dev-kit Z88DK disponga di due compilatori:</p>
<ul>
<li>l’affidabile SCCZ80 che è anche molto veloce nelle compilazione,</li>
<li>lo sperimentale ZSDCC (versione ottimizzata per Z80 di SDCC sopracitato) che però può produrre codice più efficiente e compatto di SCCZ80 a costo di compilazione più lenta e rischio di introdurre bug.</li>
</ul>
<p>Quasi tutti i compilatori che stiamo prendendo in considerazione generano codice per una sola architettura (sono <em>mono-architettura</em>) pur essendo <em>multi-target</em>.<br>
ACK è una eccezione essendo anche <em>multi-architettura</em> (con supporto per Intel 8080, Intel 8088/8086, I386, 68K, MIPS, PDP11, etc.).</p>
<p><strong>Sottoinsieme di ANSI C</strong><br>
In questo articolo per ANSI C intendiamo sostanzialmente un grosso sotto-insieme dello standard C89 in cui i <em>float</em> e i <em>long long</em> sono opzionali ma i puntatori a funzioni e puntatori a <em>struct</em> sono presenti.<br>
Non stiamo considerando versioni precedenti del C come per esempio C in sintassi <em>K&amp;R</em>.</p>
<h2 id="motivazione">Motivazione</h2>
<p>Per quale motivo dovremmo usare il C per programmare dei sistemi 8-bit?<br>
Tradizionalmente queste macchine vengono programmate in Assembly o in BASIC interpretato o in un mix dei due.<br>
Data la limitatezza delle risorse è spesso necessario ricorrere all’Assembly. Il BASIC è invece comodo per la sua semplicità e perché spesso un interprete è già presente sulla macchina.</p>
<p>Volendo limitare il confronto a questi soli tre linguaggi il seguente schema ci dà una idea delle ragione per l’uso del C.</p>

<table>
<thead>
<tr>
<th></th>
<th>facilità</th>
<th>portabilità</th>
<th>efficienza</th>
</tr>
</thead>
<tbody>
<tr>
<td>BASIC</td>
<td>SI</td>
<td>parziale</td>
<td>poca</td>
</tr>
<tr>
<td>Assembly</td>
<td>NO</td>
<td>NO</td>
<td>ottima</td>
</tr>
<tr>
<td>C</td>
<td>SI</td>
<td>SI</td>
<td>buona</td>
</tr>
</tbody>
</table><h3 id="portabilità-estrema">Portabilità estrema</h3>
<p>Quindi la ragione principale per l’uso del C è la sua portabilità assoluta. In particolare se si usa un sottoinsieme dell’ANSI C che è uno standard.<br>
In particolare l’ANSI C ci pemette di:</p>
<ul>
<li>fare porting semplificato tra una architettura all’altra</li>
<li>scrivere codice “universale”, cioè valido per diversi target <strong>senza</strong> alcuna modifica</li>
</ul>
<h3 id="buone-performance">Buone performance</h3>
<p>Qualcuno si spinge a dichiarare che il C sia una sorta di Assembly universale. Questa è una affermazione un po’ troppo ottimistica perché del C scritto molto bene non batterà mai dell’Assembly scritto bene.<br>
Ciò nonostante il C è probabilmente il linguaggio più vicino all’Assembly tra i linguaggi che permettono anche la programmazione ad alto livello.</p>
<h3 id="controindicazioni-sentimentali">Controindicazioni “sentimentali”</h3>
<p>Una ragione non-razionale ma “sentimentale” per non usare il C sarebbe data dal fatto che il C è sicuramente meno <em>vintage</em> del BASIC e Assembly perché non era un linguaggio comune sugli home computer degli anni 80 (ma lo era sui computer professionali 8-bit come come sulle macchine che usavano il sistema operativo CP/M).<br>
Credo che la programmazione in C abbia però il grosso vantaggio di poterci fare programmare l’hardware di quasi tutti i sistemi 8-bit.</p>
<h2 id="scrivere-codice-portabile">Scrivere codice portabile</h2>
<p>Scrivere codice facilmente portabile o addirittura diretammente ricompilabile per diverse piattaforme è possibile in C attraverso varie strategie:</p>
<ul>
<li>Scrivere codice <em>agnostico</em> dell’hardware e che quindi usi <em>interfacce astratte</em> (cioè delle API indipendenti dall’hardware).</li>
<li>Usare implementazioni diverse per le <em>interfacce</em> comuni da selezionare al momento della compilazione (per esempio attraverso <em>direttive al precompilatore</em> o fornendo file diversi al momento del linking).</li>
</ul>
<p>Questo diventa banale se il nostro dev-kit multi-target mette a disposizione una libreria multi-target o se ci si limita a usare le librerie standard del C (stdio, stdlib, etc.). Se si è in queste condizioni, allora basterà ricompilare il codice per ogni target e la libreria multi-target del del dev-kit farà la “magia” per noi.</p>
<p>Solo CC65 e Z88DK propongono interfacce multi-target per input e output ad accezione delle librerie standard C:</p>

<table>
<thead>
<tr>
<th>Dev-Kit</th>
<th>conio</th>
<th>tgi</th>
<th>vt52</th>
<th>vt100</th>
<th>sprites</th>
<th>UDG</th>
</tr>
</thead>
<tbody>
<tr>
<td>CC65</td>
<td>[x]</td>
<td>[x]</td>
<td>[  ]</td>
<td>[ ]</td>
<td>[ ]</td>
<td>[]</td>
</tr>
<tr>
<td>Z88DK</td>
<td>[x]</td>
<td>[ ]</td>
<td>[x]</td>
<td>[x]</td>
<td>[x]</td>
<td>[x]</td>
</tr>
</tbody>
</table><p>Quindi se usassimo esclusivamente le librerie standard C potremmo avere codice valido per ACK, CMOC, CC65 e Z88DK. Mentre se usassimo anche <em>conio</em> avremmo codice valido per <em>CC65</em> e <em>Z88DK</em>. Se usassimo altre <em>API</em> saremmo costretti a doverle implementare per i dev-kit che non le forniscono direttamente.</p>
<p><strong>Esempio</strong>:  Il gioco multi-piattaforma H-Tron è un esempio (<a href="https://sourceforge.net/projects/h-tron/">https://sourceforge.net/projects/h-tron/</a>) in cui si usano le API previste dal dev-kit Z88DK per creare un gioco su diversi sistemi ma tutti basati sull’architettura Z80.</p>
<p>Se invece non si hanno a dispozione delle API per tutti i target del proprio progetto, allora bisognerà costruirsele.<br>
Sostanzialmente si deve creare un <em>hardware abstraction layer</em> che permette di <strong>separare</strong> il codice che non dipende dall’hardware dal codice che dipende dall’hardware (per esempio l’input, output in un gioco).</p>
<p>Questo <em>pattern</em> è assai comune nella programmazione moderna e non è una esclusiva del C ma il C fornisce una serie di strumenti utili per implementare questo <em>pattern</em>. In particolare il C prevede un potente precompilatore con comandi come:</p>
<ul>
<li>#define -&gt; per definire una macro</li>
<li>#if … defined(…) … #else … #elseif -&gt; per selezione porzioni di codice che dipendono dal valore o esistenza di una macro.</li>
</ul>
<p>Inoltre tutti i compilatori prevedono una opzione (in genere “-D”) per passare una variabile al precompilatore con eventuale valore. Alcuni compilatori come CC65 implicitamente definiscono una variabile col nome del target (per esempio <em><strong>VIC20</strong></em>) per il quale si intende compilare.</p>
<p>Nel codice avremo qualcosa come:</p>
<pre><code>...
	#elif defined(__PV1000__)
		#define XSize 28
	#elif defined(__OSIC1P__) || defined(__G800__) || defined(__RX78__) 
		#define XSize 24
	#elif defined(__VIC20__) 
		#define XSize 22
...
</code></pre>
<p>per cui al momento di compilare per il <em>Vic 20</em> il precompilatore selezionerà per noi la definizione di <em>XSize</em> specifica del <em>Vic 20</em>.</p>
<p>Questo permette al precompilatore non solo di selezionare le parti di codice specifiche per una macchina, ma anche di selezionare opzioni specifiche per configurazione delle macchina (memoria aggiuntiva, scheda grafica aggiuntivo, modo grafica, compilazione di debug, etc.).</p>
<p><strong>Esempio di progetto in C per scrivere codice multi-target e multi-architettura:</strong><br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a></p>
<h2 id="ottimizzare-il-codice-per-gli-8-bit">Ottimizzare il codice per gli 8-bit</h2>
<p>Il C è un linguaggio che presenta sia costrutti ad alto livello (come gli <em>struct</em>, le funzioni come parametri, etc.) sia costruitti a basso livello (come i puntatori e la loro manipolazione). Questo non basta per farne un linguaggio direttamente adatto alla programmazione su macchine 8-bit.</p>
<h3 id="i-tipi-migliori">I “tipi migliori”</h3>
<p>Il C prevede tipi numerici interi (<em>char</em>, <em>short</em>, <em>int</em>, <em>long</em>, <em>long long</em> e loro equivalenti in versione <em>unsigned</em>).<br>
Alcuni compilatori prevedono anche tipi <em>float</em> che qui non tratteremo. Bisogna però ricordarsi che i <em>float</em> delle architetture 8-bit sono tutti <em>software</em> ed hanno quindi un costo computazionale notevole. Sono quindi da usare solo se strettamente necessari.</p>
<h4 id="il-nostro-amico-unsigned">Il nostro amico <em>unsigned</em></h4>
<p>Innanzitutto dobbiamo tenere conto che le architetture 8-bit che stiamo considerandno <strong>NON</strong> gestiscono bene tipi <em>signed</em> quindi dobbiamo evitare il più possibile l’uso di tipi numerici <em>signed</em>.</p>
<h4 id="size-matterns">“Size matterns!”</h4>
<p>Inutile soffermarsi che un’archiettura 8-bit prevede quasi solo operazioni a 8-bit e quindi è meglio limitarsi a tipi di taglia 8-bit. Se il nostro use-case ci obbligo possiamo usare tipi a 16-bit ma oltre, rischiamo di avere codice inefficiente.</p>
<h4 id="taglie-diverse-su-architetture-diverse">Taglie diverse su architetture diverse</h4>
<p>La dimensione dei tipi numeri standard dipende dal compilatore e dall’architettura e non dal linguaggio.<br>
Più recentemente sono stati introdotti dei tipi che fissano la dimensione in modo univoco (come per esempio <em>uint8_t</em> per l’intero <em>unsigend</em> a 8 bit). Non tutti i compilatori 8-bit dispongono di questi tipi ma per la stragrande maggioranza dei compilatori 8-bit abbiano la seguente situazione:</p>

<table>
<thead>
<tr>
<th></th>
<th>dimensione in bit</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<td><em>unsigned char</em></td>
<td>8</td>
<td></td>
</tr>
<tr>
<td><em>unsigned short</em></td>
<td>16</td>
<td></td>
</tr>
<tr>
<td><em>unsigned int</em></td>
<td>16</td>
<td></td>
</tr>
<tr>
<td><em>unsigned long</em></td>
<td>32</td>
<td></td>
</tr>
</tbody>
</table><p>Il tipo che dovremo usare il più possibile è quindi <em>unsigned char</em> (o qualora sia disponibile <em>uint8_t</em>).<br>
Quando costretti potremo usare <em>unsigned short</em> (o <em>uint16_t</em>). Consiglio di evitare qualunque altro tipo numerico.</p>
<h3 id="scelta-delle-operazioni">Scelta delle operazioni</h3>
<p>Quando scriviamo codice per una architettura 8-bit dobbiamo evitare se possibile codice con operazioni inefficienti o che ci obblighino a usare tipi non adatti (come i tipi <em>signed</em> o tipi a 16 o peggio 32 bit).</p>
<h4 id="applichiamo-le-buone-regole-generali">Applichiamo le buone regole generali</h4>
<p>Bisogna scegliere con cura le operazioni come si fa in C per qualunque architettura.<br>
Per esempio bisogna evitare operatori di post-incremento/decremento (<em>i++</em>, <em>i–</em>) quando non servono (cioè quando non serve il valore pre-incremento) e sostituirli con (<em>++i</em>, <em>–i</em>).</p>
<h4 id="non-produciamo-signed">Non produciamo <em>signed</em></h4>
<p>In particolare, se possibile, spesso si può riscrivere il codice in maniera da evitare sottrazioni e quando questo non è possibile, si può almeno fare in modo che il risultato della sottrazione sia sempre non-negativo.</p>
<h4 id="evitiamo-i-prodotti-espliciti">Evitiamo i prodotti espliciti</h4>
<p>Tutte le architetture che abbiamo preso in considerazione, con la sola esclusione di Motorola 6809, non dispongono di una operazione per effettuare il prodotto di due valori a 8 bit.<br>
Quindi, se possibile dobbiamo evitare i prodotti adattando il nostro codice, oppure limitarci a prodotti e divisioni che siano potenze di 2 e implementandoli con operazioni di shift con gli operatori <em>&lt;&lt;</em> e <em>&gt;&gt;</em>:</p>
<pre><code>unsigned char foo, bar;
...
foo &lt;&lt; 2; // moltiplicazione per 2^2=4
bar &gt;&gt; 1; // divisione per 2^1=2
</code></pre>
<h4 id="riscrivere-certe-operazioni">Riscrivere certe operazioni</h4>
<p>Molte operazioni come il modulo possono essere riscritte in maniera più efficiente per gli 8 bit usando operatori bit a bit. Non sempre il compilatore ottimizza nel modo migliore. Quando il compilatore non ce la fa, dobbiamo dargli una mano noi:</p>
<pre><code>unsigned char foo;
...
if(foo&amp;1) // equivalente a foo%2
{
...
}
</code></pre>
<h3 id="variabili-e-parametri">Variabili e parametri</h3>
<p>Uno dei più grossi limiti dell’architettura MOS 6502 non è la penuria di registri come si potrebbe pensare ma è la dimensione limitata del suo stack che lo rende inutilizzabile in C per la gestioni dello <em>scope</em> delle variabili.<br>
Quindi un compilatore ANSI C per 6502 sarà quasi sicuramente costretto a usare uno stack software per gestire lo scope delle variabili.<br>
Le altre architetture 8-bit che stiamo considerando soffrono meno di questo problema ma la gestione delle scope delle variabili ha un costo anche quando si usa uno stack hardware.</p>
<h4 id="un-antipattern-può-aiutarci">Un <em>antipattern</em> può aiutarci</h4>
<p>Un modo per ridurre il problema è limitare l’uso delle variabili locali e dei parametri passati alle funzioni. Questo è chiaramente un <em>antipattern</em> e se lo applicassimo a tutto il nostro codice otterremo il classico <em>spaghetti code</em>. Dobbiamo quindi scegliere sapientemente quali variabili sono assolutamente locali e quali possono essere usate come globali. Avremo codice meno generico di quello che avremmo voluto ma sarà più efficiente. <strong>NON</strong> sto suggerendo di rendere tutte le variabili globali e di non passare mai parametri alle funzioni.</p>
<h4 id="usare-funzioni-non-re-entrant">[6502] Usare funzioni non re-entrant</h4>
<p>Il compilatore CC65 per l’architettura MOS 6502 mette a disposizione l’opzione <em>-Cl</em> che rende tutte le variabili locali come <em>static</em>, quindi globali. Questo ha l’effetto di evitare l’uso dello stack per il loro scope. Ha però l’effetto di rendere tutte le nostre funzioni non re-entrant. In pratica questo ci impedisce di usare funzioni recursive. Questa non è un grave perdita perché la ricorsione sarebbe comunque una operazione troppo costosa per una architettura 8-bit.</p>
<h4 id="usare-la-pagina-zero">[6502] Usare la pagina zero</h4>
<p>Il C standard prevede la keyword <em>register</em> per suggerire al compilatore di mettere una variabile in un registro.<br>
In genere i compilatori moderni ignorano questa keyword perché lasciano questa scelta ai loro ottimizzatori. Questo è vero per i compilatori in questione ad eccezione di quello presenti in CC65 che la usa come suggerimento al compilatore per mettere una variabile in <em>pagina zero</em>. Il MOS 6502 accede in maniera più efficiente a tale pagina di memoria. Si può guadagnare memoria e velocità.<br>
Per quanto riguarda l’architettura MOS 6502, il sistema operativo di queste macchine usa una parte della pagina zero. Resta comunque una manciata di byte a disposizione del programmatore.<br>
CC65 per default lascia 6 byte della pagina zero a disposizione delle variabili dichiarate con keyword <em>register</em>.<br>
Potrebbe sembrare quindi ovvio dichiarare molte variabili come <em>register</em> ma <strong>NON</strong> è così semplice perché tutto ha un costo. Per mettere una variabile sulla <em>pagina zero</em> sono necessarie diverse operazioni. Quindi se ne avrà un vantaggio quando le variabili sono molto usate.<br>
In pratica i due scenari in cui è conveniente sono:</p>
<ol>
<li>parametri di tipo puntatore a <em>struct</em> usati almeno 3 volte all’interno di una funzione</li>
<li>variabile in un loop che si ripete almeno un centinaio di volte</li>
</ol>
<p>Un riferimento più preciso è dato da: <a href="https://www.cc65.org/doc/cc65-8.html">https://www.cc65.org/doc/cc65-8.html</a></p>
<p>Il mio consiglio è quello di compilare e vedere se il binario è più breve.</p>
<h3 id="dove-mettere-i-dati">Dove mettere i dati</h3>
<p>Se il nostro programma prevede dei dati in una definita area di memoria, sarebbe meglio metterli direttamente nel binario che verrà copiato in memoria durante il caricamento. Se questi dati sono invece nel codice, saremo costretti a scrivere del codice che li copia nell’area di memoria in cui sono previsti.<br>
Il caso più comune è forse quello degli sprites e dei caratteri/tiles ridefiniti.</p>
<p>Spesso (ma non sempre) le architetture basate su MOS 6502 prevedono video <em>memory mapped</em> in cui i dati della grafica si trovano nella stessa RAM a cui accede la CPU.</p>
<p>Molte architetture basate su Z80 (MSX, Spectravideo, Memotech, Tatung Einstein, etc.) usano il chip Texas VDP che invece ha una memoria video dedicata. Quindi non potremo mettere la grafica direttamente in questa memoria.</p>
<h4 id="istruiamo-il-linker-di-cc65">[6502] Istruiamo il linker di CC65</h4>
<p>Ogni compilatore mette a disposizioni strumenti diversi per definire la struttura del binario e quindi permetterci di costruirlo in maniera che i dati siano caricati in una determinata zona di memoria durante il load del programma senza uso di codice aggiuntivo.<br>
In particolare su CC65 si può usare il file .cfg di configurazione del linker che descrive la struttura del binario che vogliamo produrre.<br>
Il linker di CC65 non è semplicissimo da configurare ed una descrizione andrebbe oltre lo scopo di questo articolo.<br>
Una dettagliata descrizione è presente su:<br>
<a href="https://cc65.github.io/doc/ld65.html">https://cc65.github.io/doc/ld65.html</a><br>
Il mio consiglio è di leggere il manuale e di modificare i file di default .cfg già presenti in CC65 al fine di adattarli al proprio use-case.</p>
<h4 id="exomizer-ci-aiuta-anche-in-questo-caso">Exomizer ci aiuta (anche) in questo caso</h4>
<p>In alcuni casi se la nostra grafica deve trovarsi in un’area molto lontana dal codice, avremo un binario enorme e con un “buco”. Questo è il caso per esempio del C64. In questo caso io suggerisco di usare <em>exomizer</em> sul risultato finale: <a href="https://bitbucket.org/magli143/exomizer/wiki/Home">https://bitbucket.org/magli143/exomizer/wiki/Home</a></p>
<h4 id="z80-z88dk-ci-aiuta-molto-nella-grafica">[Z80] Z88DK ci aiuta molto nella grafica</h4>
<p>Il dev-kit Z88DK possiede strumenti potentissimi per la grafica multi-piattaforma e fornisce diverse API sia per gli sprite software (<a href="https://github.com/z88dk/z88dk/wiki/monographics">https://github.com/z88dk/z88dk/wiki/monographics</a>) che per i caratteri ridefiniti indipendentemente dalla presenza del chip Texas VDP. Se usiamo queste API sarà il compilatore a decidere dove mettere questi dati e eventualmente a copiarli in memoria video.</p>
<h2 id="uso-avanzato-della-memoria">Uso avanzato della memoria</h2>
<p>Il compilatore C produrrà un unico binario che conterrà codice e dati che verranno caricati in una specifica zona di memoria (è comunque possibile avere porzioni di codice non contigue).</p>
<p>In molte architetture alcune aree della memoria RAM sono usate come <em>buffer</em> oppure sono dedicate a usi specifici come alcune modalità grafiche.<br>
Il mio consiglio è quindi di studiare le mappa della memoria di ogni hardware per trovare queste preziose aree.<br>
In particolare consiglio:</p>
<ul>
<li>buffer della cassetta, della tastiera, della stampante, del disco</li>
<li>memoria usata dal BASIC</li>
<li>aree di memoria dedicate a modi grafici (che non si intendono usare)</li>
<li>aree di memoria libere ma non contigue e che quindi non sarebbero parte del nostro binario</li>
</ul>
<p>Queste aree di memoria potrebbero essere sfruttate dal nostro codice se nel nostro use-case non servono per il loro scopo originario (esempio: se non intendiamo caricare da cassetta dopo l’avvio del programma, possiamo usare il buffer della cassetta per metterci delle variabili da usare dopo l’avvio potendolo comunque usare prima dell’avvio per caricare il nostro stesso programma da cassetta).</p>
<p>In C standard potremmo solo definire le variabili puntatore e gli array come locazioni in queste aree di memoria.<br>
Non esiste però un modo standard per dire al compilatore di mettere tutte le variabili in una specifica area di memoria.<br>
I compilatori di CC65 e Z88DK invece prevedono una sintassi per permetterci di fare questo e guadagnare diverse centinaia di byte preziosi.<br>
Vari esempi sono presenti in:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory</a></p>
<p>In particolare bisogna creare un file Assembly .s (con CC65) o .asm (con Z88DK) da linkare al nostro eseguibile in cui assegnamo un indirizzo ad ogni nome di variabile a cui  <strong>aggiungiamo</strong> un prefisso <em>underscore</em>.</p>
<p>Di seguito diamo un esempio di mappatura delle variabili <em>ghosts</em>, <em>bombs</em>, <em>player</em>.</p>
<p>Sintassi CC65</p>
<pre><code>.export _ghosts;
_ghosts = $33c

.export _bombs;
_bombs = _ghosts + $28 

.export _player;
_player = _bombs + $14
...
</code></pre>
<p>Sintassi Z88DK</p>
<pre><code>PUBLIC _ghosts, _bombs, _player, 
...

defc _ghosts = 0x2A00
defc _bombs = _ghosts + $28 
defc _player = _bombs + $14
...
</code></pre>
<p>CMOC mette a dispozione l’opzione <em>--data=&lt;indirizzo&gt;</em> che permette di allocare tutte le variabili globali scrivibili a partire da un indirizzo dato.</p>
<p>La documentazione di ACK non dice nulla a riguardo. Potremo comunque definire i tipi puntatore e gli array nelle zone di memoria libera.</p>
<h2 id="sfruttare-lhardware-specifico">Sfruttare l’hardware specifico</h2>
<p>Come visto nelle sezioni precedenti, anche se programmiamo in C non dobbiamo dimenticare l’hardware specifico per il quale stiamo scrivendo del codice.</p>
<p>In alcuni casi conoscere l’hardware può aiutarci a scrivere codice molto più compatto e/o più veloce.</p>
<p>Per esempio, è inutile ridefinire dei caratteri per fare della grafica se il sistema dispone già di caratteri utili al nostro scopo.</p>
<h3 id="vic20-caso-molto-speciale-commodore-vic-20">[Vic20] Caso molto speciale: Commodore Vic 20</h3>
<p>Il Commodore Vic 20 è un caso veramente speciale perché prevede dei limiti hardware (RAM totale: 5k, RAM disponibile per il codice: 3,5K) ma anche dei trucchi per superarli almeno in parte:</p>
<ul>
<li>In realtà dispone anche di 1024 nibble di RAM aggiuntiva speciale per gli attributi colore</li>
<li>Pur avendo soltanto 3,5k di memoria RAM contigua per il codice, molta altra RAM è facilmente sfruttabile per dati (buffer cassetta, buffer comando INPUT del BASIC)</li>
<li>La caratteristica più sorprendente è che il chip grafico VIC può mappare una parte dei caratteri in RAM lasciandone metà definiti dalla ROM</li>
</ul>
<p>Quindi, sfruttiamo implicitamente la prima caratteristica accedendo ai colori senza usare la RAM comune.<br>
Possiamo mappare le nostre variabili nei vari buffer non utilizzati.<br>
Se ci bastano n (n&lt;=64) caratteri ridefiniti possiamo mapparne solo 64 con <em>POKE(0x9005,0xFF);</em> Ne potremo usare anche meno di 64 lasciando il resto per il codice ma mantenendo in aggiunta 64 caratteri standard senza alcun dispoendio di memoria per i caratteri standard.</p>
<h2 id="la-programmazione-ad-oggetti">La programmazione ad oggetti</h2>
<p>Contrariamente a quello che si possa credere, la programmazione ad oggetti è possibile in ANSI C e può aiutarci a produrre codice più compatto in alcune situazioni.Esistono interi framework ad oggetti che usano ANSI C (es. Gnome è scritto in uno di questi framework).</p>
<p>Nel caso delle macchine 8-bit con vincoli di memoria molto forti, possiamo comunque implementare <em>classi</em>, <em>polimorfismo</em> ed <em>ereditarietà</em> in maniera molto efficiente.<br>
Una trattazione dettagliata non è possibile in questo articolo e qui ci limitiamo a citare i due strumenti fondamentali:</p>
<ul>
<li>usare <em>puntatori a funzioni</em> per ottenere methodi polimorfici (cioè il cui comportamento è definito a run-time). Si può evitare l’implementazione di una <em>vtable</em> se ci si limita a classi con un solo metodo polimorfico.</li>
<li>usare puntatori a <em>struct</em> e <em>composizione</em> per implementare sotto-classi: dato uno <em>struct</em> A, si implementa una sua sotto-classe con uno <em>struct</em> B definito come uno <em>struct</em> il cui <strong>primo</strong> campo è A. Usando puntatori a tali <em>struct</em>, il C garantisce che gli offset di B siano gli stessi degli offset di A.</li>
</ul>
<p>Esempio preso da<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase</a><br>
Definiamo <em>Item</em> è un sotto-classe di <em>Character</em> con metodo polimorfico <em>_effect</em>:</p>
<pre><code>	struct CharacterStruct
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
</code></pre>
<p>Perché ci guadagnamo in termine di memoria?<br>
Perché sarà possibile trattare più oggetti con lo stesso codice e quindi risparmiamo memoria.</p>
<h2 id="compilazione-ottimizzata">Compilazione ottimizzata</h2>
<p>Non tratteremo in modo esaustivo le opzioni di compilazione dei cross-compilatori e consigliamo di fare riferimento ai loro rispettivi manuali per dettagli avanzati. Qui daremo una lista delle opzioni per compilare codice ottimizzato su ognuno dei compilatori che stiamo trattando.</p>
<h3 id="ottimizzazione-aggressiva">Ottimizzazione “aggressiva”</h3>
<p>Le seguenti opzioni applicano il massimo delle ottimizzazioni</p>

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
<td><em>-O6</em></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><em>-O3</em></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><em>-SO3</em> <em>–max-alloc-node20000</em></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><em>-O</em> <em>-Cl</em></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><em>-O2</em></td>
</tr>
</tbody>
</table><p><strong>Problemi noti</strong></p>
<ul>
<li>CC65: <em>-Cl</em> impedisce la ricorsione</li>
<li>CMOC: <em>-O2</em> ha dei bug</li>
<li>ZSDCC: ha dei bug a prescindere dalle opzioni e ne ha altri presenti con <em>-SO3</em> in assenza di <em>–max-alloc-node20000</em>.</li>
</ul>
<h3 id="ottimizzazione-più-sicura">Ottimizzazione più sicura</h3>

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
<td><em>-O3</em></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><em>-O</em></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><em>-O1</em></td>
</tr>
</tbody>
</table><h2 id="evitare-il-linking-di-codice-inutile">Evitare il linking di codice inutile</h2>
<p>I compilatori che trattiamo non sempre saranno capaci di eliminare il codice non usato. Dobbiamo quindi evitare di includere codice non utile per essere sicuri che non finisca nel binario prodotto.</p>
<p>Possiamo fare ancora di meglio con alcuni dei nostri compilatori, istruendoli a non includere alcune librerie standard o persino alcune loro parti se siamo sicuri di non doverle usare.</p>
<h2 id="sistemi-che-non-sono-supportati">Sistemi che non sono supportati</h2>
<p>I nostri dev-kit supportano una lista di target per ogni architettura attraverso la presenza di librerie specifiche per l’hardware. E’ comunque possibile sfruttare questi dev-kit per altri target ma dovremo fare più lavoro: saremo costretti ad implementare tutta la parte di codice specifica del target.</p>
<p>Qui diamo una lista delle opzioni di compilazione per target generico per ogni dev-kit ma per maggiori dettagli facciamo riferimento ai rispettivi manuali.</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Dev-Kit</th>
<th>Opzione</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td>(*)</td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td>+test  +embedded  +cpm</td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td>+none</td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td>–nodefaultlibs</td>
</tr>
</tbody>
</table><p>(*) ACK prevede solo il target CP/M-80 per l’architettura Intel 8080 ma è possibile almeno in principio usare ACK per produrre binari Intel 8080 generico ma non è semplice: bisogna usare la sequenza di commanti chiamati da ACK e che sono visibili quando si compila con opzione <em>-v -v</em> (<em>ack +mcpm -v -v …</em>) come <em>cemcom.ansi</em> per produrre codice <em>byte code</em> da C.</p>

