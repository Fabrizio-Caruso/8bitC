---


---

<p><img src="8bitC.jpg" alt="8-bit C"></p>
<h1 id="c-portabile-e-ottimizzato-per-gli-8-bit">C portabile e ottimizzato per gli 8-bit</h1>
<h2 id="seconda-parte-tecniche-per-ottimizzare-il-codice-c-per-8-bit"><em>Seconda parte</em>: Tecniche per ottimizzare il codice C per 8-bit</h2>
<p>Questa è la seconda parte di una serie di tre articoli che descrivono tecniche per scrivere codice portabile e ottimizzato in ANSI C per <strong>tutti</strong> i sistemi 8-bit <em>vintage</em>, cioè computer, console, handheld, calcolatrici scientifiche e microcontrollori dalla fine degli anni 70 fino a metà degli anni 90.<br>
L’articolo completo è disponibile on-line su <a href="https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md">https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md</a></p>
<p>Consigliamo la lettura del primo articolo in cui abbiamo presentato i vari cross compilatori C per architetture 8 bit e abbiamo dato alcune indicazioni su come scrivere codice C portabile su tutte le architetture 8 bit.</p>
<h2 id="ottimizzare-il-codice-in-generale">Ottimizzare il codice in generale</h2>
<p>Ci sono alcune regole generali per scrivere codice migliore indipendentemente dal fatto che l’architettura sia 8-bit o meno.<br>
Non tutte le buone pratiche di programmazione saranno ottimali per gli 8 bit. In questa sezione diamo esempi di pratiche generali che rimangono valide anche su architetture 8 bit vintage.</p>
<h3 id="riutilizziamo-le-stesse-funzioni">Riutilizziamo le stesse funzioni</h3>
<p>In generale, in qualunque linguaggio di programmazione si voglia programmare, è importante evitare la duplicazione del codice o la scrittura di codice superfluo.</p>
<h4 id="programmazione-strutturata">Programmazione strutturata</h4>
<p>Spesso guardando bene le funzioni che abbiamo scritto scopriremo che condividono delle parti comuni e che quindi potremo <em>ri-fattorizzare</em> il codice introducendo delle <em>sotto-funzioni</em> che le nostre funzioni chiameranno.<br>
Dobbiamo però tenere conto che, oltre un certo limite, una eccessiva granularità del codice ha effetti deleteri perché una chiamata ad una funzione ha un costo computazionale e di memoria.</p>
<h4 id="generalizzare-il-codice-parametrizzandolo">Generalizzare il codice parametrizzandolo</h4>
<p>In alcuni casi è possibile generalizzare il codice passando un parametro per evitare di scrivere due funzioni diverse molto simili.</p>
<h5 id="passiamo-delle-variabili">Passiamo delle variabili</h5>
<p>Se due funzioni fanno la stessa cosa su oggetti diversi, sarebbe meglio avere una unica funzione a cui si passi l’oggetto su cui agire.</p>
<h5 id="passiamo-delle-funzioni">Passiamo delle funzioni</h5>
<p>In altri casi avremo porzioni di codice simili la cui unica differenza è l’applicazione di una funzione diversa. In questo caso possiamo scrivere un’unica funzione a cui si passa un puntatore a funzione.<br>
Non tutti sono familiari con la sintassi dei puntatori a funzione e quindi ne diamo un esempio in cui definiamo la funzione <code>sumOfSomething(range, something)</code> che somma il valore di <code>something(i)</code> per i che va da zero a <code>i-1</code>:</p>
<pre><code>unsigned short sumOfSomething(unsigned char range, unsigned short (* something) (unsigned char))
{
    unsigned char i;
    unsigned short res =0;
    for(i=0;i&lt;range;++i)
    {
        res+=something(i);
    }
    return res;
}

</code></pre>
<p>Quindi date due funzioni:</p>
<pre><code>unsigned short square(unsigned char val)
{
        return val*val;
}
</code></pre>
<pre><code>unsigned short next(unsigned char val)
{
    return ++val;
}
</code></pre>
<p>potremo usare <code>sumOfSomething</code> con l’una o l’altra funzione evitando di scrivere il codice che fa la somma due volte:</p>
<pre><code>printf("%d\n",sumOfSomething(4,square));
</code></pre>
<p>mostrerà 14, cioè la somma dei quadrati di 0,1,2,3.</p>
<pre><code>printf("%d\n",sumOfSomething(4,next));
</code></pre>
<p>mostrerà 10, cioè la somma di 1,2,3,4.</p>
<h5 id="passiamo-degli-offset-di-struct">Passiamo degli offset di <code>struct</code></h5>
<p>In altri casi possiamo avere due funzioni quasi identiche la cui unica differenza è il campo di uno <code>struct</code> che si modifica. In questo caso possiamo scrivere un’unica funzione a cui passiamo l’<em>offset</em> dello <code>struct</code>.</p>
<p>Un esempio avanzato si trova in <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h</a> dove, dato uno <code>struct</code> con due campi <code>_x</code> e <code>_y</code>,  vogliamo potere agire sul valore di uno o dell’altro in situazioni diverse:</p>
<pre><code>	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		...
	};
	typedef struct CharacterStruct Character;
</code></pre>
<p>Possiamo evitare di scrivere due diverse funzioni per agire su <code>_x</code> e su <code>_y</code> creando una unica funzione a cui si passa un <em>offset</em> che faccia da selettore:</p>
<pre><code>	unsigned char moveCharacter(Character* hunterPtr, unsigned char offset)
	{
		if((unsigned char) * ((unsigned char*)hunterPtr+offset) &lt; ... )
		{
			++(*((unsigned char *) hunterPtr+offset));
		}
		else if((unsigned char) *((unsigned char *) hunterPtr+offset) &gt; ... )
		{
			--(*((unsigned char *) hunterPtr+offset));
		}
	...
	}
</code></pre>
<p>Stiamo sfruttando il fatto che il secondo campo <code>_y</code> si trova esattamente un byte dopo il primo campo <code>_x</code>. Quindi con <code>offset=0</code> accediamo al campo <code>_x</code> e con <code>offset=1</code> accediamo al campo <code>_y</code>. I vari <em>cast</em> a <code>unsigned char *</code> servono per accedere a byte in posizioni diverse all’interno dello <code>struct</code>.</p>
<p><strong>Avvertenze</strong>: Dobbiamo però considerare sempre che aggiungere un parametro ha un costo e quindi dovremo verificare (anche guardando la taglia del binario ottenuto) se nel nostro caso ha un costo inferiore al costo di una funzione aggiuntiva.</p>
<h4 id="stesso-codice-su-oggetti-simili">Stesso codice su <em>oggetti</em> simili</h4>
<p>Si può anche fare di più e usare lo stesso codice su <em>oggetti</em> che non sono esattamente dello stesso tipo ma che condividono solo alcuni aspetti comuni per esempio sfruttando gli <code>offset</code> dei campi negli <code>struct</code>, <em>puntatori a funzioni</em>, etc.<br>
Questo è possibile in generale tramite la <em>programmazione ad oggetti</em> di cui descriveremo una implementazione <em>light</em> per gli 8-bit nel prossimo articolo.</p>
<h3 id="pre-incrementodecremento-vs-post-incrementodecremento">Pre-incremento/decremento vs Post-incremento/decremento</h3>
<p>Bisogna evitare operatori di post-incremento/decremento (<code>i++</code>, <code>i--</code>) quando non servono (cioè quando non serve il valore prima dell’incremento) e sostituirli con (<code>++i</code>, <code>--i</code>).<br>
Il motivo è che l’operatore di post-incremento richiede almeno una operazione in più dovendo conservare il valore originario.<br>
Nota: E’ totalmente inutile usare un operatore di post-incremento in un ciclo <code>for</code>.</p>
<h3 id="costanti-vs-variabili">Costanti vs Variabili</h3>
<p>Una qualunque architettura potrà ottimizzare meglio del codice in cui delle variabili sono sostituite con delle costanti. Quindi se una data variabile ha un valore noto al momento della compilazione, è importante che sia rimpiazzata con una costante.<br>
Se il suo valore, pur essendo noto al momento della compilazione, dovesse dipendere da una opzione di compilazione (per esempio il tipo di target), allora la sostituiremo con una <em>macro</em> da settare attraverso una opzione di compilazione, in maniera tale che sia trattata come una costante dal compilatore.</p>
<h2 id="ottimizzare-il-codice-per-gli-8-bit">Ottimizzare il codice per gli 8-bit</h2>
<p>Il C è un linguaggio che presenta sia costrutti ad alto livello (come <code>struct</code>, le funzioni come parametri, etc.) sia costruiti a basso livello (come i puntatori e la loro manipolazione). Questo non basta per farne un linguaggio direttamente adatto alla programmazione su macchine 8-bit.</p>
<h3 id="aiutiamo-il-compilatore-a-ottimizzare-le-costanti">Aiutiamo il compilatore a ottimizzare le costanti</h3>
<p>Per compilatori <em>single pass</em> (come la maggioranza dei cross-compilatori 8-bit come per esempio CC65), può essere importante aiutare il compilatore a capire che una data espressione sia una costante.</p>
<p><strong><em>Esempio</em></strong> (preso da <a href="https://www.cc65.org/doc/coding.html">https://www.cc65.org/doc/coding.html</a>):<br>
Un compilatore <em>single pass</em> valuterà la seguente espressione da sinistra a destra non capendo che <code>OFFS+3</code> è una costante:</p>
<pre><code>	#define OFFS   4
	int  i;
	i = i + OFFS + 3;
</code></pre>
<p>Quindi sarebbe meglio riscrivere <code>i = i + OFFS+3</code> come <code>i = OFFS+3+i</code> oppure <code>i = i + (OFFS+3)</code>:</p>
<pre><code>	#define OFFS   4
	int  i;
	i = OFFS+3+i;
</code></pre>
<h3 id="implementare-peek-e-poke-in-c">Implementare <code>peek</code> e <code>poke</code> in C</h3>
<p>Quasi sicuramente su una architettura 8-bit  avremo bisogno di scrivere e leggere dei singoli byte su alcune specifiche locazioni di memoria.<br>
Il modo per fare questo in BASIC sarebbe stato attraverso i comando <code>peek</code> e <code>poke</code>.<br>
In C dobbiamo farlo attraverso dei puntatori la cui sintassi non è leggibilissima. Potremo però costruirci delle utili macro che useremo nel nostro codice:</p>
<pre><code>    #define POKE(addr,val)  (*(unsigned char*) (addr) = (val))
    #define PEEK(addr)      (*(unsigned char*) (addr))
</code></pre>
<p>Nota: I compilatori scriveranno codice ottimale nel caso in cui si passino delle costanti come parametri.</p>
<p>Per maggiori dettagli facciamo riferimento a: <a href="https://github.com/cc65/wiki/wiki/PEEK-and-POKE">https://github.com/cc65/wiki/wiki/PEEK-and-POKE</a></p>
<h3 id="i-tipi-migliori-per-gli-8-bit">I “tipi migliori” per gli 8-bit</h3>
<p>Una premessa importante per la scelta dei tipi da preferire per architettura è data dal fatto che in generale abbiamo questa situazione:</p>
<ul>
<li>tutte le operazioni aritmetiche sono solo a 8 bit</li>
<li>la maggior parte delle operazioni sono ad 8 bit, alcune sono a 16-bit e nessuna operazione è a 32 bit</li>
<li>le operazioni <code>signed</code> (cioè con segno) sono più lente di quelle <code>unsigned</code></li>
<li>l’hardware non supporta operazioni in <em>virgola mobile</em></li>
</ul>
<h4 id="evitiamo-conversioni-inutili">Evitiamo conversioni inutili</h4>
<p>Le conversioni tra tipi e soprattutto le conversioni tra tipi <code>signed</code> e <code>unsigned</code> sono costose.</p>
<h4 id="tipi-interi-vs-tipi-a-virgola-mobile">Tipi interi vs tipi a virgola mobile</h4>
<p>Il C prevede tipi numerici interi con segno (<code>char</code>, <code>short</code>, <code>int</code>, <code>long</code>, <code>long long</code> e loro equivalenti in versione <code>unsigned</code>).<br>
Molti compilatori (ma non CC65) prevedono il tipo <code>float</code> (numeri a <em>virgola mobile</em>) che qui non tratteremo. Bisogna considerare che i <code>float</code> delle architetture 8-bit sono tutti <em>software float</em> ed hanno quindi un costo computazionale notevole. Sarebbero quindi da usare solo se strettamente necessari.</p>
<h4 id="il-nostro-amico-unsigned">Il nostro amico <em>unsigned</em></h4>
<p>Siccome le architetture 8-bit che stiamo considerandno <strong>NON</strong> gestiscono ottimalmente tipi <code>signed</code>, dobbiamo evitare il più possibile l’uso di tipi numerici <code>signed</code>.</p>
<h4 id="size-matters">“Size matters!”</h4>
<p>La dimensione dei tipi numeri standard dipende dal compilatore e dall’architettura e non dal linguaggio.<br>
Più recentemente sono stati introdotti dei tipi che fissano la dimensione in modo univoco (come per esempio <code>uint8_t</code> per l’intero <code>unsigend</code> a 8 bit). Per includere questi tipi a taglia fissata dallo standard usiamo:</p>
<pre><code>	#include &lt;stdint.h&gt;
</code></pre>
<p>Non tutti i compilatori 8-bit dispongono di questo header.</p>
<p>Fortunatamente per la stragrande maggioranza dei compilatori 8-bit abbiamo la seguente situazione:</p>

<table>
<thead>
<tr>
<th>tipo</th>
<th>numero bit</th>
<th>nome in <code>stdint.h</code></th>
<th>nome alternativo</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>unsigned char</code></td>
<td>8</td>
<td><code>uint8_t</code></td>
<td><code>byte</code></td>
</tr>
<tr>
<td><code>unsigned short</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
<td><code>word</code></td>
</tr>
<tr>
<td><code>unsigned int</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
<td><code>word</code></td>
</tr>
<tr>
<td><code>unsigned long</code></td>
<td>32</td>
<td><code>uint32_t</code></td>
<td><code>dword</code></td>
</tr>
</tbody>
</table><p>Quindi dovremo:</p>
<ul>
<li>usare il più possibile <code>unsigned char</code> (o <code>uint8_t</code>) per le operazioni aritmetiche;</li>
<li>usare <code>unsigned char</code> (o <code>uint8_t</code>) e <code>unsigned short</code> (o <code>uint16_t</code>) per tutte le altre operazioni, evitando se possibile qualunque operazione a 32 bit.</li>
</ul>
<p>Nota: In assenza dell’header <code>stdint.h</code>, sarebbe bene creare dei <code>typedef</code> opportuni:</p>
<pre><code>	typedef unsigned char uint8_t;
	typedef unsigned short uint16_t;
	typedef unsigned long uint32_t;
</code></pre>
<h3 id="scelta-delle-operazioni">Scelta delle operazioni</h3>
<p>Quando scriviamo codice per una architettura 8-bit dobbiamo evitare se possibile codice con operazioni inefficienti o che ci obblighino a usare tipi non adatti (come i tipi <code>signed</code> o tipi a 16 o peggio 32 bit).</p>
<h4 id="non-produciamo-signed">Non produciamo <em>signed</em></h4>
<p>In particolare, se possibile, spesso si può riscrivere il codice in maniera da evitare sottrazioni e quando questo non è possibile, si può almeno fare in modo che il risultato della sottrazione sia sempre non-negativo.</p>
<h4 id="evitiamo-i-prodotti-espliciti">Evitiamo i prodotti espliciti</h4>
<p>Tutte le architetture che abbiamo preso in considerazione, con la sola esclusione di Motorola 6809, non dispongono di una operazione per effettuare il prodotto di due valori a 8 bit.<br>
Quindi, se possibile dobbiamo evitare i prodotti adattando il nostro codice, oppure limitarci a prodotti e divisioni che siano potenze di 2 e implementandoli con operazioni di shift con gli operatori <em>&lt;&lt;</em> e <em>&gt;&gt;</em>:</p>
<pre><code>	unsigned char foo, bar;
	...
	foo &lt;&lt; 2; // moltiplicazione per 2^2=4
	bar &gt;&gt; 1; // divisione per 2^1=2
</code></pre>
<h4 id="riscrivere-certe-operazioni">Riscrivere certe operazioni</h4>
<p>Molte operazioni come il modulo possono essere riscritte in maniera più efficiente per gli 8 bit usando operatori bit a bit perché non sempre il compilatore ottimizza nel modo migliore. Quindi dobbiamo dargli una mano noi:</p>
<pre><code>	unsigned char foo;
	...
	if(foo&amp;1) // equivalente a foo%2
	{
		...
	}
</code></pre>
<h3 id="variabili-e-parametri">Variabili e parametri</h3>
<p>Uno dei più grossi limiti dell’architettura MOS 6502 non è la penuria di registri come si potrebbe pensare ma è la dimensione limitata del suo <em>stack hardware</em> (in <em>pagina uno</em>: <code>$0100-01FF</code>) che lo rende inutilizzabile in C per la gestioni dello <em>scope</em> delle variabili e i parametri delle funzioni.<br>
Quindi un compilatore ANSI C per 6502 sarà quasi sicuramente costretto a usare uno <em>stack software</em> per</p>
<ul>
<li>gestire lo scope delle variabili,</li>
<li>gestire il passaggio dei parametri.</li>
</ul>
<p>Le altre architetture 8-bit che stiamo considerando soffrono meno di questo problema ma la gestione delle scope delle variabili e dei parametri ha un costo anche quando si usa uno <em>stack hardware</em>.</p>
<h4 id="un-antipattern-può-aiutarci">Un <em>antipattern</em> può aiutarci</h4>
<p>Un modo per ridurre il problema è limitare l’uso delle variabili locali e dei parametri passati alle funzioni. Questo è chiaramente un <em>antipattern</em> e se lo applicassimo a tutto il nostro codice otterremo il classico <em>spaghetti code</em>. Dobbiamo quindi scegliere sapientemente quali variabili sono necessariamente locali e quali possono essere usate come globali. Avremo codice meno generico di quello che avremmo voluto ma sarà più efficiente. <strong>NON</strong> sto suggerendo di rendere tutte le variabili globali e di non passare mai parametri alle funzioni.</p>
<h4 id="usare-funzioni-non-re-entrant">[6502] Usare funzioni non re-entrant</h4>
<p>Il compilatore CC65 per l’architettura MOS 6502 mette a disposizione l’opzione <code>-Cl</code> che rende tutte le variabili locali come se fossero <code>static</code>, quindi globali. Questo ha l’effetto di evitare l’uso dello <em>stack software</em> per il loro scope. Ha però l’effetto di rendere tutte le nostre funzioni non re-entrant. In pratica questo ci impedisce di usare funzioni ricorsive. Questa non è un grave perdita perché la ricorsione sarebbe comunque una operazione troppo costosa per una architettura 8-bit.</p>
<h4 id="usare-la-pagina-zero">[6502] Usare la pagina zero</h4>
<p>Il C standard prevede la keyword <code>register</code> per suggerire al compilatore di mettere una variabile in un registro.<br>
In genere i compilatori moderni ignorano questa keyword perché lasciano questa scelta ai loro ottimizzatori. Questo è vero per i compilatori in questione ad eccezione di CC65 che la usa come suggerimento al compilatore per mettere una variabile in <em>pagina zero</em>. Il MOS 6502 accede in maniera più efficiente a tale pagina di memoria. Si può guadagnare memoria e velocità.<br>
Per quanto riguarda l’architettura MOS 6502, il sistema operativo di queste macchine usa una parte della pagina zero. Resta comunque una manciata di byte a disposizione del programmatore. CC65 per default lascia 6 byte della pagina zero a disposizione delle variabili dichiarate con keyword <code>register</code>.<br>
Potrebbe sembrare quindi ovvio dichiarare molte variabili come <code>register</code> ma <strong>NON</strong> è così semplice perché tutto ha un costo. Per mettere una variabile sulla <em>pagina zero</em> sono necessarie diverse operazioni. Quindi se ne avrà un vantaggio quando le variabili sono molto usate.<br>
In pratica i due scenari in cui è conveniente sono:</p>
<ol>
<li>parametri di tipo puntatore a <code>struct</code> usati almeno 3 volte all’interno di una funzione,</li>
<li>variabile in un loop che si ripete almeno un centinaio di volte.</li>
</ol>
<p>Un riferimento più preciso è dato da: <a href="https://www.cc65.org/doc/cc65-8.html">https://www.cc65.org/doc/cc65-8.html</a></p>
<p>Il mio consiglio è quello di compilare e vedere se il binario è divenuto più breve o, ancora meglio, ispezionare il codice Assembly generato.</p>
<h4 id="evitare-lallocazione-dinamica-della-memoria">Evitare l’allocazione dinamica della memoria</h4>
<p>I compilatori che stiamo considerando consentono di allocare e deallocare la memoria dinamicamente (con comandi come <code>malloc</code> e <code>free</code>) ma questo ha un ovvio costo computazionale. Se possibile è preferibile allocare tutta la memoria staticamente.</p>
<h3 id="codice-su-file-diversi">Codice su file diversi?</h3>
<p>In generale è bene separare in più file il proprio codice se il progetto è di grosse dimensioni.<br>
Questa buona pratica può però avere degli effetti deleteri per gli ottimizzatori dei compilatori 8-bit perché in generale non eseguono <em>link-time optimization</em>, cioè non ottimizzeranno codice tra più file ma si limitano ad ottimizzare ogni file singolarmente.<br>
Quindi se per esempio abbiamo una funzione che chiamiamo una sola volta e la funzione è definita nello stesso file in cui viene usata, l’ottimizzatore potrebbe metterla <em>in line</em> ma non lo farà se la funzione è definita in un altro file.<br>
Il mio consiglio <strong>NON</strong> è quello di creare file enormi con tutto ma è quello di tenere comunque conto di questo aspetto quando si decide di separare il codice su più file e di non abusare di questa buona pratica.</p>

