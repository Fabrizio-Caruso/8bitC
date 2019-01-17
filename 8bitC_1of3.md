---


---

<p><img src="8bitC.jpg" alt="8-bit C"></p>
<h1 id="c-portabile-e-ottimizzato-per-gli-8-bit">C portabile e ottimizzato per gli 8-bit</h1>
<h2 id="prima-parte-introduzione--e-scrittura-di-codice-c-portabile-per-8-bit"><em>Prima parte</em>: Introduzione  e Scrittura di codice C portabile per 8-bit</h2>
<p>Questa è la prima parte di una serie di tre articoli che descrivono tecniche per scrivere codice portabile e ottimizzato in ANSI C per <strong>tutti</strong> i sistemi 8-bit <em>vintage</em>, cioè computer, console, handheld, calcolatrici scientifiche e microcontrollori dalla fine degli anni 70 fino a metà degli anni 90.<br>
L’articolo completo è disponibile on-line su <a href="https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md">https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md</a></p>
<p>Il contenuto dell’articolo originale sarà trattato in tre parti:</p>
<ol>
<li>Introduzione e Scrittura di codice C portabile per 8-bit</li>
<li>Tecniche per ottimizzare il codice C per 8-bit</li>
<li>Tecniche avanzate per ottimizzare il codice C per 8-bit</li>
</ol>
<h2 id="architetture">Architetture</h2>
<p>In particolare ci occuperemo dei sistemi basati sulle seguenti <em>architetture</em> (e architetture derivate e retrocompatibili):</p>
<ul>
<li>Intel 8080 (*)</li>
<li>MOS 6502</li>
<li>Motorola 6809</li>
<li>Zilog Z80 (*)</li>
</ul>
<p>(*) Lo Zilog Z80 è una estensione dell’Intel 8080, quindi un binario Intel 8080 sarà utilizzabile anche su un sistema con Z80 ma il viceversa non è vero.</p>
<p>Buona parte di queste tecniche sono valide su altre architetture 8-bit come quella del COSMAC 1802 e quella del microcontrollore Intel 8051.</p>
<h2 id="obiettivi">Obiettivi</h2>
<p>Lo scopo di questa serie di articoli è duplice:</p>
<ol>
<li>descrivere tecniche generiche per scrivere codice <strong>portabile</strong>, cioè valido e compilabile su <strong>tutti</strong> i sistemi 8-bit indipendentemente dal fatto che un sistema sia supportato esplicitamente da un compilatore o meno</li>
<li>descrivere tecniche generali per <strong>ottimizzare</strong> il codice C su <strong>tutti</strong> i sistemi 8-bit</li>
</ol>
<h2 id="premesse">Premesse</h2>
<p>Questa serie di articoli <strong>non</strong> è un manuale introduttivo al linguaggio <em>C</em> e richiede</p>
<ul>
<li>conoscenza del linguaggio <em>C</em>;</li>
<li>conoscenza della programmazione strutturata e a oggetti;</li>
<li>conoscenza dell’uso di compilatori e linker.</li>
</ul>
<p>Inoltre <strong>non</strong> ci occuperemo in profondità di alcuni argomenti avanzati quali:</p>
<ul>
<li>alcuni ambiti specifici della programmazione come grafica, suono, input/output, etc.</li>
<li>l’interazione tra C e Assembly.</li>
</ul>
<p>Questi argomenti avanzati sono molto importanti e meriterebbero degli articoli separati a loro dedicati.</p>
<h2 id="terminologia">Terminologia</h2>
<p>Introduciamo alcuni termini che saranno ricorrenti in questa serie di articoli:</p>
<ul>
<li>Un <em>sistema</em> è un qualunque tipo di macchina dotata di processore come computer, console, handheld, calcolatrici, sistemi embedded, etc.</li>
<li>Un <em>target</em> di un compilatore è un sistema supportato dal compilatore, cioè un sistema per il quale il compilatore mette a disposizione supporto specifico come librerie e la generazione di un binario in formato specifico.</li>
<li>Un <em>architettura</em> è un tipo di processore (Intel 8080, 6502, Zilog Z80, Motorola 6809, etc.) . Un <em>target</em> appartiene quindi ad una sola architettura data dal suo processore (con rare eccezioni come il Commodore 128 che ha sia un processore derivato dal 6502 e uno Zilog Z80).</li>
</ul>
<h2 id="cross-compilatori-multi-target">Cross-compilatori multi-target</h2>
<p>Per produrre i nostri binari 8-bit consigliamo l’uso di <em>cross compilatori</em> <em>multi-target</em> (cioè compilatori eseguiti su PC che producono binari per diversi <em>target</em>).</p>
<h3 id="cross-compilatori-vs-compilatori-nativi">Cross-compilatori vs compilatori nativi</h3>
<p>Non consigliamo l’uso di compilatori <em>nativi</em> perché sarebbero molto scomodi (anche se usati all’interno di un emulatore accelerato al massimo) e non potrebbero mai produrre codice ottimizzato perché l’ottimizzatore sarebbe limitato dalla risorse della macchina 8-bit.</p>
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
<td>MOS 6502</td>
<td>CC65</td>
<td><a href="https://github.com/cc65/cc65">https://github.com/cc65/cc65</a></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><a href="https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html">https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html</a></td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td><a href="https://github.com/z88dk/z88dk">https://github.com/z88dk/z88dk</a></td>
</tr>
</tbody>
</table><p>Inoltre esistono altri <em>cross compilatori</em> C <em>multi-target</em> che non tratteremo qui ma per i quali buona parte delle stesse tecniche generiche rimangono valide:</p>
<ul>
<li>LCC1802 (<a href="https://sites.google.com/site/lcc1802/">https://sites.google.com/site/lcc1802/</a>) per il COSMAC 1802;</li>
<li>SDCC (<a href="http://sdcc.sourceforge.net/">http://sdcc.sourceforge.net/</a>) per svariate architetture di microprocessori come lo Z80 e di microcontrollori come l’Intel 8051;</li>
<li>GCC-6809 (<a href="https://github.com/bcd/gcc">https://github.com/bcd/gcc</a>) per 6809 (adattamento di GCC);</li>
<li>GCC-6502 (<a href="https://github.com/itszor/gcc-6502-bits">https://github.com/itszor/gcc-6502-bits</a>) per 6502 (adattamento di GCC);</li>
<li>SmallC-85 (<a href="https://github.com/ncb85/SmallC-85">https://github.com/ncb85/SmallC-85</a>) per Intel 8080/8085 ;</li>
<li>devkitSMS (<a href="https://github.com/sverx/devkitSMS">https://github.com/sverx/devkitSMS</a>) per le console Sega basate su Z80 come Sega Master System, Sega Game Gear e Sega SG-1000.</li>
</ul>
<p>Si noti come il dev-kit Z88DK disponga di due compilatori:</p>
<ul>
<li>l’affidabile SCCZ80 che è anche molto veloce nelle compilazione,</li>
<li>lo sperimentale ZSDCC (versione ottimizzata per Z80 di SDCC sopracitato) che però può produrre codice più efficiente e compatto di SCCZ80 a costo di compilazione più lenta e rischio di introdurre bug.</li>
</ul>
<p>Quasi tutti i compilatori che stiamo prendendo in considerazione generano codice per una sola architettura (sono <em>mono-architettura</em>) pur essendo <em>multi-target</em>. ACK è una eccezione essendo anche <em>multi-architettura</em> (con supporto per Intel 8080, Intel 8088/8086, I386, 68K, MIPS, PDP11, etc.).</p>
<p>Questo articolo <strong>non</strong> è né una introduzione né un manuale d’uso di questi compilatori e <strong>non</strong> tratterà:</p>
<ul>
<li>l’istallazione dei compilatori;</li>
<li>elementi specifici per l’uso di base di un compilatore</li>
</ul>
<p>Per i dettagli sull’istallazione e l’uso di base dei compilatori in questione, facciamo riferimento ai manuali e alle pagine web dei relativi compilatori. Una introduzione all’uso di CC65 è disponibile su <a href="https://www.retroacademy.it/author/fabrizio-lodi/">https://www.retroacademy.it/author/fabrizio-lodi/</a></p>
<p><strong>Sottoinsieme di ANSI C</strong><br>
In questa serie di articoli per ANSI C intendiamo sostanzialmente un grosso sotto-insieme dello standard C89 in cui i <code>float</code> e i <code>long long</code> sono opzionali ma i puntatori a funzioni e puntatori a <code>struct</code> sono presenti.<br>
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
<p>Una ragione non-razionale ma “sentimentale” per non usare il C sarebbe data dal fatto che il C è sicuramente meno <em>vintage</em> del BASIC e Assembly perché non era un linguaggio comune sugli home computer degli anni 80 (ma lo era sui computer professionali 8-bit come sulle macchine che usavano il sistema operativo CP/M).<br>
Credo che la programmazione in C abbia però il grosso vantaggio di poterci fare programmare l’hardware di quasi tutti i sistemi 8-bit.</p>
<h2 id="scrivere-codice-portabile">Scrivere codice portabile</h2>
<p>Scrivere codice facilmente portabile o addirittura direttammente compilabile per diverse piattaforme è possibile in C attraverso varie strategie:</p>
<ul>
<li>Scrivere codice <em>agnostico</em> dell’hardware e che quindi usi <em>interfacce astratte</em> (cioè delle API indipendenti dall’hardware).</li>
<li>Usare implementazioni diverse per le <em>interfacce</em> comuni da selezionare al momento della compilazione (per esempio attraverso <em>direttive al precompilatore</em> o fornendo file diversi al momento del linking).</li>
</ul>
<h3 id="codice-portabile-tra-target-dello-stesso-compilatore-o-dev-kit">Codice portabile tra target dello stesso compilatore o dev-kit</h3>
<p>Questo diventa banale se il nostro dev-kit multi-target mette a disposizione una libreria multi-target o se ci si limita a usare le librerie standard del C (stdio, stdlib, etc.). Se si è in queste condizioni, allora basterà ricompilare il codice per ogni target e la libreria multi-target del del dev-kit farà la “magia” per noi.</p>
<p>Solo CC65 e Z88DK propongono interfacce multi-target per input e output oltre le librerie C standard:</p>

<table>
<thead>
<tr>
<th>Dev-Kit</th>
<th>Architettura</th>
<th>librerie multi-target</th>
</tr>
</thead>
<tbody>
<tr>
<td>Z88DK</td>
<td>Zilog Z80</td>
<td>standard C lib, CONIO(testuale), vt52 (testuale), vt100 (testuale), sprite software, UDG, bitmap, joystick Z88DK</td>
</tr>
<tr>
<td>CC65</td>
<td>MOS 6502</td>
<td>standard C lib, CONIO(testuale), TGI (bitmap), joystick CC65</td>
</tr>
<tr>
<td>CMOC</td>
<td>Motorola 6809</td>
<td>standard C lib</td>
</tr>
<tr>
<td>ACK</td>
<td>Intel 8080</td>
<td>standard C lib</td>
</tr>
</tbody>
</table><h4 id="librerie-specifiche-di-cc65">Librerie specifiche di CC65</h4>
<p>CC65 fornisce la libreria <em>CONIO</em> per la visualizzazione di testo e la libreria grafica multi-target TGI per grafica bit-map tra alcuni dei suoi target e API proprie per leggere lo stato dei joystick.</p>
<h4 id="librerie-specifiche-di-z88dk">Librerie specifiche di Z88DK</h4>
<p>Anche Z88DK fornisce la libreria <em>CONIO</em> e API proprie per leggere lo stato dei joystick. Inoltre fornisce molti strumenti per la grafica multi-target tra cui API per grafica bitmap, per gli sprite software (<a href="https://github.com/z88dk/z88dk/wiki/monographics">https://github.com/z88dk/z88dk/wiki/monographics</a>) e per caratteri ridefiniti per buona parte dei suoi 80 target.</p>
<p><em>Esempio</em>:  Il gioco multi-piattaforma H-Tron è un esempio (<a href="https://sourceforge.net/projects/h-tron/">https://sourceforge.net/projects/h-tron/</a>) in cui si usano le API previste dal dev-kit Z88DK per creare un gioco su molti sistemi basati sull’architettura Z80.</p>
<h4 id="crearsi-una-libreria-multi-target-sulla-stessa-architettura">Crearsi una libreria multi-target sulla stessa architettura</h4>
<p>In tutti i casi in cui le librerie a disposizione non facciano al caso nostro bisognerà crearsi una libreria multi-target sfruttando eventualmente tutto quello che lo specifico dev-kit mette a disposizione.</p>
<h3 id="scrivere-codice-portabile-tra-architetture-diverse">Scrivere codice portabile tra architetture diverse</h3>
<p>Per potere avere codice portabile su target e eventualmente anche su architetture diverse bisogna usare eventuali librerie comuni a dev-kit diversi o scriversi una libreria multi-architettura e quindi anche compilabile da compilatori di dev-kit <strong>diversi</strong>.</p>
<h4 id="librerie-c-standard">Librerie C standard</h4>
<p>Se per l’input/output usassimo esclusivamente le librerie C standard (come <code>stdio.h</code>) potremmo avere codice compilabile con ACK, CMOC, CC65 e Z88DK ma saremmo limitati a input e output testaule  limitato a comandi come <code>printf</code> e <code>scanf</code>  senza controllo preciso della posizione del testo</p>
<h4 id="libreria-conio-comune-tra-cc65-e-z88dk">Libreria <em>CONIO</em> comune tra CC65 e Z88DK</h4>
<p>Se per l’input/output, oltre alle librerie standard C, usassimo solo la libreria <em>CONIO</em> (che nasce per input/output testuale su <em>MS-DOS</em>) avremmo codice compilabile con <em>CC65</em> e <em>Z88DK</em> ma avremmo input e output testuale limitato a comandi come <code>cprintf</code>, <code>cgetc</code> , <code>gotoxy</code> che consentono il controllo della posizione del testo. Per maggiori dettagli facciamo riferimento a <a href="https://www.cc65.org/doc/funcref-14.html">https://www.cc65.org/doc/funcref-14.html</a></p>
<h4 id="crearsi-una-libreria-multi-architettura">Crearsi una libreria multi-architettura</h4>
<p>In tutti gli altri casi se vogliamo scrivere codice portabile su architetture diverse bisognerà crearsi una libreria multi-target e multi-architettura che quindi non avrà alcuna dipendenza da un dev-kit.</p>
<h3 id="scrivere-una-libreria">Scrivere una libreria</h3>
<p>Scrivere una libreria multi-target o addirittura multi-architettura significa creare un <em>hardware abstraction layer</em> il cui scopo e di permettere la <strong>separazione</strong> tra:</p>
<ul>
<li>il codice che non dipende dall’hardware (per esempio la logica di un gioco) che usa l’interfaccia della libreria</li>
<li>dal codice della libreria la cui implementazione dipende dall’hardware (per esempio le funzioni per l’input, output in un gioco) ma la cui interfaccia non dipende dall’hardware.</li>
</ul>
<p>Questo <em>pattern</em> è assai comune nella programmazione moderna e non è una esclusiva del C ma il C fornisce una serie di strumenti utili per implementare questo <em>pattern</em> in maniera che che si possano supportare hardware diversi da selezione al momento della compilazione. In particolare il C prevede un potente precompilatore con comandi come:</p>
<ul>
<li><code>#define</code> -&gt; per definire una macro</li>
<li><code>#if</code> … <code>defined(...)</code> … <code>#elif</code> … <code>#else</code>…<code>#endif</code> -&gt; per selezione porzioni di codice che dipendono dal valore o esistenza di una macro.</li>
</ul>
<p>Inoltre tutti i compilatori prevedono l’opzione  <code>-D</code> per passare una variabile al precompilatore con eventuale valore. Alcuni compilatori come CC65 implicitamente definiscono una variabile col nome del target (per esempio <em><strong>VIC20</strong></em>) per il quale si intende compilare.</p>
<p>Nel codice avremo qualcosa come:</p>
<pre><code>...
		#if defined(__PV1000__)
			#define XSize 28
		#elif defined(__OSIC1P__) || defined(__G800__) || defined(__RX78__) 
			#define XSize 24
		#elif defined(__VIC20__) 
			#define XSize 22
...
		#endif
...
</code></pre>
<p>e al momento di compilare per il <em>Vic 20</em> il precompilatore selezionerà per noi la definizione di <code>XSize</code> specifica del <em>Vic 20</em>.</p>
<p>Questo permette al precompilatore non solo di selezionare le parti di codice specifiche per una macchina, ma anche di selezionare opzioni specifiche per configurazione delle macchina (memoria aggiuntiva, modo grafico, debug, etc.).</p>
<h4 id="cross-chase-e-crosslib"><em>Cross-Chase</em> e <em>CrossLib</em></h4>
<p>Il progetto <em>Cross-Chase</em>: <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a> propone il codice di un gioco compilabile su più di 100 sistemi 8-bit diversi con circa 200 configurazioni diverse usando sempre lo <strong>stesso codice</strong> del gioco grazie alla libreria universale <em>CrossLib</em>.</p>
<p>Se guardiamo il codice pubblicato sulla sopracitata pagina GitHub vediamo come sia stato separato in:</p>
<ul>
<li>codice del gioco (directory <code>src/chase</code>) totalmente indipendente dall’hardware,</li>
<li>codice della libreria <em>CrossLib</em> (directory <code>src/cross_lib</code>) cheimplementa i dettagli di ogni hardware possibile.</li>
</ul>
<h3 id="codice-portabile-su-sistemi-non-supportati">Codice portabile su sistemi non supportati</h3>
<p>I nostri dev-kit supportano una lista di target per ogni architettura attraverso la presenza di librerie specifiche per l’hardware. E’ comunque possibile sfruttare un dato dev-kit per altri sistemi con la stessa architettura ma saremo dovremo implementare tutta la parte di codice specifica del target:</p>
<ul>
<li>codice necessario per gestire l’input/output (grafica, tastiera, joystick, suoni, etc.)</li>
<li>codice necessario per inizializzare il sistema</li>
</ul>
<p>Per fare ciò potremo in molti casi usare le routine già presenti nella ROM (nel terzo articolo di questa serie diamo un semplice esempio che è anche su <a href="https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md">https://github.com/Fabrizio-Caruso/8bitC/blob/master/8bitC.md</a>).</p>
<p>Inoltre dovremo procurarci o scrivere un convertitore del binario in un formato accettabile per il nuovo sistema.</p>
<p>Per esempio CC65 non supporta né il <em>BBC Micro</em> né l’<em>Atari 7800</em> e CMOC non supporta l’<em>Olivetti Prodest PC128</em> ma è comunque possibile usare (o estendere) questi dev-kit per produrre binari per questi target:</p>
<ul>
<li>Cross Chase (<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a>) supporta (in principio) qualunque architettura anche non supportata direttamente dai compilatori come per esempio l’Olivetti Prodest PC128.</li>
<li>Il gioco Robotsfindskitten è stato portato per l’Atari 7800 usando CC65 (<a href="https://sourceforge.net/projects/rfk7800/files/rfk7800/">https://sourceforge.net/projects/rfk7800/files/rfk7800/</a>).</li>
<li>BBC è stato aggiunto come target sperimentale su CC65 (<a href="https://github.com/dominicbeesley/cc65">https://github.com/dominicbeesley/cc65</a>).</li>
</ul>
<h4 id="compilare-per-sistemi-non-supportati">Compilare per sistemi non supportati</h4>
<p>Per potere compilare per un sistema non supportato dobbiamo usare alcune opzioni che servono ad eliminare le dipendenze da un  target specifico. Qui diamo una lista di alcune opzioni utili a questo scopo. Per maggiori dettagli facciamo riferimento ai rispettivi manuali.</p>

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
<td>MOS 6502</td>
<td>CC65</td>
<td><code>+none</code></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>--nodefaultlibs</code></td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td><code>+test</code> (generico), <code>+embedded</code> (generico con nuove librerie),  <code>+cpm</code> (solo per generico CP/M)</td>
</tr>
</tbody>
</table><p>(*) ACK prevede solo il target CP/M-80 per l’architettura Intel 8080 ma è possibile almeno in principio usare ACK per produrre binari Intel 8080 generico ma non è semplice in quanto ACK usa una sequenze di comandi per produrre il Intel 8080 partendo dal C e passando da vari stai intermedi compreso un byte-code “EM”:</p>
<ol>
<li><code>ccp.ansi</code>:  precompilatore</li>
<li><code>em_cemcom.ansi</code>: compila C preprocompilato producendo bytecode</li>
<li><code>em_opt</code>: ottimizza il bytecode</li>
<li><code>cpm/ncg</code>: genera Assembly da bytecode</li>
<li><code>cpm/as</code>: genera codice Intel 80 da Assembly</li>
<li><code>em_led</code>: linker</li>
</ol>
<p>Un’alternativa a ACK per generare binari Intel 8080 generici è data dal già citato compilatore SmallC-85.</p>

