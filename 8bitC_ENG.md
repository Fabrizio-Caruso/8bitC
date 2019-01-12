---


---

<p><img src="8bitC.jpg" alt="8-bit C"></p>
<h1 id="portable-and-optimized-c-for-8-bit-systems">Portable and Optimized C for 8-bit Systems</h1>
<p>This article describes some optimizing techniques for ANSI C code for <strong>all</strong> 8-bit <em>vintage</em> systems, i.e., computers, consoles, hand-helds, scientific calculators from the end of the '70s until mid '90s and in particular for the systems based on the following <em>architectures</em> (and the derived and compatible architectures):</p>
<ul>
<li>Intel 8080 (*)</li>
<li>MOS 6502</li>
<li>Motorola 6809</li>
<li>Zilog Z80 (*)</li>
</ul>
<p>(*) The Zilog Z80 is an extension of the Intel 8080. Therefore an Intel 8080 binary is compatible with a Z80-based system but not the other way round.</p>
<p>Most of the techniques presented here remain valid on other 8-bit architectures such as the COSMAC 1802 and the Intel 8051.</p>
<p>The goal of this article is two-fold:</p>
<ol>
<li>present general techniques to <strong>optimize</strong> C code for <strong>all</strong> 8-bit systems;</li>
<li>present general techniques to write <strong>portable</strong> C code, i.e., valid and compatible code for <strong>all</strong> 8-bit systems, including systems that are not natively and explicitly supported by C compilers</li>
</ol>
<h2 id="preconditions">Preconditions</h2>
<p>This article is <strong>not</strong> a introduction nor a manual for the <em>C</em> language and has the following preconditions:</p>
<ul>
<li>knowledge of the <em>C</em> language;</li>
<li>knowledge of structured and object-oriented programming;</li>
<li>familiarity with compilers and linkers.</li>
</ul>
<p>Besides this article does not cover in depth some advanced topics such as:</p>
<ul>
<li>coding in specific domains such as graphics, sound, input/output;</li>
<li>interaction between C and Assembly.</li>
</ul>
<p>These advanced topics are very important and would require separate articles.</p>
<h2 id="definitions">Definitions</h2>
<p>In this article we will refer to <em>system</em>, <em>target</em> and <em>architecture</em> with the following meanings:</p>
<ul>
<li>A <em>system</em> is any kind of processor-equipped machine such as computers, consoles, hand-helds, scientigic calculators, embedded systems, etc.</li>
<li>A <em>target</em> of a compiler is any kind of <em>system</em> supported by the compiler, i.e., a system for which the compiler provides specific support with libraries and the generator of a binary in the specific format.</li>
<li>An <em>architecture</em> is a processor family (e.g., Intel 8080, MOS 6502, etc.). Therefore a <em>target</em> has an <em>architecture</em> that corresponds to its processor. So it has only one architecture unless if it has more than two or more processors belonging to two different families (such as the Commodore 128, which has both a Z80 and a 6502-derived processor)</li>
</ul>
<h2 id="multi-target-cross-compilers">Multi-target cross-compilers</h2>
<p>In order to produce binaries from source code we recommend <em>multi-target</em> <em>cross-compilers</em> (i.e., compilers that are run on a modern PC and that produce binaries for multiple <em>targets</em>).</p>
<h3 id="cross-compilers-vs-native-compilers">Cross-compilers vs native compilers</h3>
<p>We do not recommend the use of <em>native</em> compilers because they would be inconvenient (even if used inside an accellerated emulator) and would never produce the same kind of optimized code due to the limitated resources of the 8-bit system.</p>
<p>In particular we will refer to the following <em>multi-targer</em> <em>cross-compilers</em>:</p>
<p>We will refer in particular to the following <em>multi-target</em> <em>cross-compilers</em>:</p>

<table>
<thead>
<tr>
<th>Architecture</th>
<th>Compiler/Dev-Kit</th>
<th>Web Page</th>
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
</table><p>We mention other <em>multi-target</em> <em>cross-compilers</em> that we do not cover here but for which most of the described general techniques are valid:</p>
<ul>
<li>SDCC (<a href="http://sdcc.sourceforge.net/">http://sdcc.sourceforge.net/</a>) for several architectures including the 8-bit Zilog Z80 and Intel 8051;</li>
<li>LCC1802 (<a href="https://sites.google.com/site/lcc1802/">https://sites.google.com/site/lcc1802/</a>) for the COSMAC 1802 8-bit processor;</li>
<li>GCC-6809 (<a href="https://github.com/bcd/gcc">https://github.com/bcd/gcc</a>) for the Motorola 6809 (GCC adaptation);</li>
<li>GCC-6502 (<a href="https://github.com/itszor/gcc-6502-bits">https://github.com/itszor/gcc-6502-bits</a>) for the MOS 6502 (GCC adaptation);</li>
<li>SmallC-85 (<a href="https://github.com/ncb85/SmallC-85">https://github.com/ncb85/SmallC-85</a>) for the Intel 8080/8085 ;</li>
<li>devkitSMS (<a href="https://github.com/sverx/devkitSMS">https://github.com/sverx/devkitSMS</a>) for Sega  Z80-based consoles (Sega Master System, Sega Game Gear, Sega SG-1000).</li>
</ul>
<p>We remark that the Z88DK dev-kit provides two compilers:</p>
<ul>
<li>the more reliable SCCZ80 that also offers fast compilation,</li>
<li>the experimental ZSDCC (Z80-only optimized SDCC version) that can produce faster and more compact code than SCCZ80 at the cost of much slower compilation and the risk introducing erratic behavior.</li>
</ul>
<p>Almost of considered compilers generate code for just one architecture (they are <em>mono-architecture</em>) even though they are <em>multi-target</em>. ACK is an exception because it is <em>multi-architecture</em> (Intel 8080, Intel 8088/8086, I386, 68K, MIPS, PDP11, etc.).</p>
<p>This article is <em>not</em> an introduction nor a manual for these compilers and it will <em>not</em> cover the following topics:</p>
<ul>
<li>compiler installation</li>
<li>basic usage of the compiler</li>
</ul>
<p>For details on these topics we refer to the compiler’s manuals and web pages.</p>
<p><strong>A sub-set of ANSI C</strong><br>
In this article we will refer to <em>ANSI C</em> as to a large sub-set of the C89 standard where <code>float</code> and <code>long long</code> are optional but pointers to functions and pointers to <code>struct</code> are present.<br>
We will not consider previous versions such as C in <em>K&amp;R</em> syntax.</p>
<h2 id="motivation">Motivation</h2>
<p>Why should we use C to code for vintage 8-bit systems?<br>
Traditionally these systems were coded in either Assembly or interpreted BASIC or a mix of the two.<br>
Given the limited resources, Assembly was ofter necessary. BASIC was convenient for its simplicity and because an interpreter was often present on the system.</p>
<p>If we limit our comparison to just Assembly, BASIC and C, the following tables summarizes the reasons of using C:</p>

<table>
<thead>
<tr>
<th>language</th>
<th>simplicity</th>
<th>portability</th>
<th>efficiency</th>
</tr>
</thead>
<tbody>
<tr>
<td>Assembly</td>
<td>low</td>
<td>no</td>
<td>optimal</td>
</tr>
<tr>
<td>BASIC</td>
<td>high</td>
<td>low</td>
<td>low</td>
</tr>
<tr>
<td>C</td>
<td>high</td>
<td>high</td>
<td>good</td>
</tr>
</tbody>
</table><h3 id="very-high-portability">Very high portability</h3>
<p>Therefore the main reason for using ANSI C is its portability.<br>
In particolar ANSI C allows us:</p>
<ul>
<li>to ease porting from different architectures</li>
<li>to write “universal” code, that is valid code for different targets <em>without</em> any necessary modification</li>
</ul>
<h3 id="good-performance">Good performance</h3>
<p>Someone declares C as a sort of universal Assembly language. I do not fully agree with this statement because optimally-written C will never beat optimally-written Assembly.<br>
Nevertheless, C is the closest language to Assembly among the languages that allow high level programming.</p>
<h3 id="sentimental-drawbacks">“Sentimental drawbacks”</h3>
<p>One not fully rational reason for not using C in this context is the fact coding in C provides a less <em>vintage</em> experience compared to BASIC and Assembly because it was less common on the home computers from the 80s (but it was common on 8-bit business computers such as on computers that used the CP/M operating system).<br>
A good reason for coding in C is that C allows us to code any 8-bit system.</p>
<h2 id="writing-portable-code">Writing portable code</h2>
<p>Writing easily portable code or even directly compilable for different architectures is possible is C through different strategies:</p>
<ul>
<li>Write code that is <em>hardware-agnostic</em> through <em>abstract interfaces</em> (that is hardware-independent APIs)</li>
<li>Use different implementations of the *interfaces" and select them at compilation-time (by using <em>precompiler-directives</em> or by providing different files at linking-time)</li>
</ul>
<h3 id="writing-portable-code-for-targets-of-a-dev-kit">Writing portable code for targets of a dev-kit</h3>
<p>This is trivial if our multi-target dev-kit provides a multi-target library or if we just use standard C libraries (e.g., stdio, stdlib, etc.). Under these conditions we just need to recompile our code. The multi-target library will do the “magic” for us.</p>
<p>Unfortunately only CC65 and Z88DK provide significant multi-target libraries for input and output other than standard C libraries:</p>

<table>
<thead>
<tr>
<th>Dev-Kit</th>
<th>Architecture</th>
<th>multi-target libraries</th>
</tr>
</thead>
<tbody>
<tr>
<td>Z88DK</td>
<td>Zilog Z80</td>
<td>standard C lib, conio, vt52, vt100, sprite software, UDG, bitmap</td>
</tr>
<tr>
<td>CC65</td>
<td>MOS 6502</td>
<td>standard C lib, conio, tgi (bitmap)</td>
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
</table><p>In particular Z88DK has very powerful libraries for multi-target graphics and even provides APIs for software sprites (<a href="https://github.com/z88dk/z88dk/wiki/monographics">https://github.com/z88dk/z88dk/wiki/monographics</a>) and redefined characters for most of its 80 targets.<br>
<strong>Example</strong>: The code of the game H-Tron (<a href="https://sourceforge.net/projects/h-tron/">https://sourceforge.net/projects/h-tron/</a>) uses Z88DK’s APIs for low resolution bitmap graphics for a multitude of Z80-based targets.</p>
<p>Therefore if we were to use exclusively the standard C libraries we could compile our code with ACK, CMOC, CC65 and Z88DK. If we used <em>conio</em> we could compile the code with CC65 and Z88DK.</p>
<p>In all other cases, if we want to write portable code on different architectures and systems, we would need to write a “hardware abstraction layer” that allows us to <strong>separate</strong>:</p>
<ul>
<li>the code that does not depend on the hardware (e.g., the logic part)</li>
<li>the code that depends on the harware (e.g., input/output in a videogame)</li>
</ul>
<p>This <em>patterm</em> is very common in modern programming and it is not exclusive to C. For this purpose C provides a set of tools to implement this <em>pattern</em> in a way to select the different portions of code required by each hardware at compilation-time.<br>
In particular C provides a powerful pre-compiler with commands such as:</p>
<ul>
<li><code>#define</code> -&gt; to define a <em>macro</em></li>
<li><code>#if</code> … <code>defined(...)</code> … <code>#elif</code> … <code>#else</code> -&gt; to select code portions that depend on the existence of value of a given macro</li>
</ul>
<p>Moreover all compilers provide the option <code>-D</code> to pass a macro to the pre-compiler. Some compilers such as CC65 implicitly define a macro that depends on the selected target (e.g., <em><strong>VIC20</strong></em>).</p>
<p>In our code we may have something like:</p>
<pre><code>...
		#elif defined(__PV1000__)
			#define XSize 28
		#elif defined(__OSIC1P__) || defined(__G800__) || defined(__RX78__) 
			#define XSize 24
		#elif defined(__VIC20__) 
			#define XSize 22
...
</code></pre>
<p>When we compile for the <em>Vic 20</em> target, the pre-compiler will select for us the <em>Vic 20</em>-specific definition of <code>XSize</code>.<br>
This also allows to select specific options for the configuration of the target (additional memory, video card, video mode, debug compilation, etc.).</p>
<p>As main example we refer to the <em>Cross-Chase</em> project:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a></p>
<p>The code of <em>Cross-Chase</em> provides an example of how to write <em>universal</em> code for any system and architecture:</p>
<ul>
<li>the code of the game (<em>src/chase</em> directory) is hardware-independent</li>
<li>the code of the <em>crossLib</em> library  (<em>src/cross_lib</em> directory) implements all the hardware-sepcific details</li>
</ul>
<h3 id="writing-portable-code-even-for-non-supported-systems">Writing portable code even for non-supported systems</h3>
<p>The dev-kits under our consideration support a list of targets for each architecture by providing specific libraries. Nevertheless it is possible to exploit these dev-kits for other systems with the same architecture but we will have to implement all the hardware-specific code:</p>
<ul>
<li>the necessary code for input/output (e.g., graphics, sounds, keyboard, joystick, etc.)</li>
<li>the necessary code for correct machine initialization</li>
</ul>
<p>Alternatively, it is possible to extend a dev-kit to support to new targets.</p>
<p>In many cases, we can use the ROM routines to do this (see the section on the ROM routines)</p>
<p>Moreover we may have to convert the binary to a format that can be acccepted by the system.</p>
<p>Therefore, we can indeed write portable code for even these unsupported systems.</p>
<p>For example CC&amp;% does not support the <em>BBC Micro</em>, nor the <em>Atari 7800</em> and CMOC does not support the <em>Olivetti Prodest PC128</em>. Yet, it is possible to use the dev-kit to produce binaries for these targets:</p>
<ul>
<li>Cross Chase (<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a>) supports (theoretically) any architecture even the unsupported ones such as for example the l’Olivetti Prodest PC128.</li>
<li>The game Robotsfindskitten is been compiled for thr Atari 7800 with CC65 (<a href="https://sourceforge.net/projects/rfk7800/files/rfk7800/">https://sourceforge.net/projects/rfk7800/files/rfk7800/</a>).</li>
<li>BBC has already been added unofficially as an experimental new target in CC65 (<a href="https://github.com/dominicbeesley/cc65">https://github.com/dominicbeesley/cc65</a>).</li>
</ul>
<h4 id="compilation-for-unsupported-targets">Compilation for unsupported targets</h4>
<p>We give a list of compilation options for generic target for each dev-kit so that the compiler is instructued to compile without any dependence on a specific target. For more details we refer to the manual of the dev-kits.</p>

<table>
<thead>
<tr>
<th>Architecture</th>
<th>Dev-Kit</th>
<th>Option(s)</th>
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
<td><code>+test</code>, <code>+embedded</code> (new lib),  <code>+cpm</code> (generic CP/M target)</td>
</tr>
</tbody>
</table><p>(*) ACK officially only supports the CP/M-80 for the Intel 8080 architecture but it is possible to use ACK to build generic Intel 8080 binaries but it is not very simple because ACK uses a sequence of commands to produce intermediate results (including the “EM” byte-code):</p>
<ol>
<li><code>ccp.ansi</code>:  C precompiler</li>
<li><code>em_cemcom.ansi</code>: compiles precompiled C code into “EM” byte-code</li>
<li><code>em_opt</code>: optimizes “EM” byte-code</li>
<li><code>cpm/ncg</code>: generates Intel 8080 Assembly from “EM” bytecode</li>
<li><code>cpm/as</code>: generates Intel 8080 binary from Assembly</li>
<li><code>em_led</code>: links object files</li>
</ol>
<h2 id="general-c-code-optimization">General C code optimization</h2>
<p>We describe some general rules to improve the code that do not depend on whether the architecture is 8-bit or not.</p>
<h3 id="re-use-same-functions">Re-use same functions</h3>
<p>In general, in whatever programming language we want to code, it is important to avoid code duplication and unnecessary code.</p>
<h4 id="structured-programming">Structured programming</h4>
<p>We have to examine each function in order to find common portions that we can <em>factor</em> by introducing <em>sub-functions</em> that are original function can call.<br>
However we must take into account that, beyond a certain limit, excessive code granularity has negative effects because each function call has a computational and memory cost.</p>
<h4 id="parametrized-code">Parametrized code</h4>
<p>In some cases it is possible to generalize the code by passing a parameter to avoid writing very similar functions.<br>
An advanced example is in <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h</a> where, given a <code>struct</code> with two fields <code>_x</code> and <code>_y</code>,<br>
we want to be able to change the value of one field or the other in different situations:</p>
<pre><code>	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		...
	};
	typedef struct CharacterStruct Character;
</code></pre>
<p>We avoid two different functions for <code>_x</code> and <code>_y</code> by creating one function to which we pass an <em>offset</em> to select the field:</p>
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
<p>In this case, we use the fact that the field <code>_y</code> is exactly one byte after the field <code>_x</code>. Therefore with <code>offset=0</code> we access <code>_x</code> and with <code>offset=1</code> we access <code>_y</code>.</p>
<p><strong>Warning</strong>: We must always remember that adding a parameter has a cost and we must verify that the cost of the parameter is lower than the cost of an extra function (e.g., by looking at the size of the obtained binary).</p>
<h4 id="same-code-on-similar-objects">Same code on similar <em>objects</em></h4>
<p>We can do even better and use the same code on <em>objects</em> that are not identical but shares some common features by using <code>offset</code> in <code>struct</code>, <em>pointers to functions</em>, etc.<br>
In general this is possible through <em>object-oriented programming</em> whose light-weight implementation for 8-bit systems is described in a subsequent section in this article.</p>
<h3 id="pre-incrementdecrement--vs-post-incrementdecrement">Pre-increment/decrement  vs Post-increment/decrement</h3>
<p>We must avoid post-increment/decrement operators (<code>i++</code>, <code>i--</code>) when they are not needed, i.e.,  when we do not need the original value and replace them with (<code>++i</code>, <code>--i</code>).<br>
The reason is that the post-increment operator requires at least an extra operation to save the original value.<br>
Remark: It is totally useless to use a post-increment in a <code>for</code> loop.</p>
<h3 id="costant-vs-variables">Costant vs Variables</h3>
<p>Any architecture will perform better if variables are replace with constants.</p>
<h4 id="use-constants">Use constants</h4>
<p>Therefore if a variable has a known value at compilation-time, it is important to replace it with a constant.<br>
If its value depends on some compilation option, then we should use a <em>macro</em> to set its value.</p>
<h4 id="help-the-compiler-to-recognize-constants">Help the compiler to recognize constants</h4>
<p>Besides, for <em>single pass</em> compilers (the majority of 8-bit cross-compilers, e.g., CC65), it is important to help the compiler whether a given expression is a constant.</p>
<p><strong><em>Example</em></strong> (from <a href="https://www.cc65.org/doc/coding.html">https://www.cc65.org/doc/coding.html</a>):<br>
A <em>single pass</em> compiler may evaluate the following expression from left to right and miss the fact that <code>OFFS+3</code> is a constant:</p>
<pre><code>	#define OFFS   4
	int  i;
	i = i + OFFS + 3;
</code></pre>
<p>In this case it would be better to re-write <code>i = i + OFFS+3</code> as <code>i = OFFS+3+i</code> or <code>i = i + (OFFS+3)</code>.</p>
<h2 id="bit-specific-code-optimization">8-bit specific code optimization</h2>
<p>The C language has both high level constructs (such as <code>struct</code>, functions as parameters, etc.) and low level constructs (such as pointers, bitsise operators, etc.).<br>
This is not enough to make C a programming language well-suited for programming 8-bit systems.</p>
<h3 id="implement-peek-and-poke-in-c">Implement <code>peek</code> and <code>poke</code> in C</h3>
<p>Mosr probably we will need to read and write single bytes from and to specific memory locations.<br>
In old BASIC this was done through <code>peek</code> and <code>poke</code> commands.<br>
In C we must do this through poniters whose syntax is not very readable. In order to make our code more readable we can create the following macros:</p>
<pre><code>    #define POKE(addr,val)  (*(unsigned char*) (addr) = (val))
    #define PEEK(addr)      (*(unsigned char*) (addr))
</code></pre>
<p>Remark: The compilers will produce optimal code when we use constants as parameters of these macros.</p>
<p>For more details we refer to: <a href="https://github.com/cc65/wiki/wiki/PEEK-and-POKE">https://github.com/cc65/wiki/wiki/PEEK-and-POKE</a></p>
<h3 id="the-best-types-for-8-bit-systems">The “best types” for 8-bit systems</h3>
<p>First of all we must take into account that we have the following situation:</p>
<ul>
<li>all arithmetic operations are just 8-bit</li>
<li>most of other operations use 8 bits while some may use 16 bits and none uses 32 bits</li>
<li><code>signed</code> operations are slower than <code>unsigned</code> operations</li>
<li>the hardware does not support <em>floating point</em> operations</li>
</ul>
<h4 id="integer-vs-floating-point-types">Integer vs floating point types</h4>
<p>The C languages provides <code>signed</code> integer types (<code>char</code>, <code>short</code>, <code>int</code>, <code>long</code>, <code>long long</code>, etc.) and their <code>unsigned</code> counterparts.<br>
Most cross compiler (but not CC65) support the <code>float</code> type (for <em>floating point</em> numbers), which we do not cover here.<br>
We only remark that <code>float</code> numbers in 8-bit architecture are always <em>software float</em> and therefore have a high computational cost.<br>
Hence we should only use them when strictly necessary.</p>
<h4 id="our-friend-unsigned">Our friend <em>unsigned</em></h4>
<p>Since the 8-bit architectures under consideration do <strong>NOT</strong> handle <code>signed</code> types well, we must avoid them whenever possible.</p>
<h4 id="size-matters">“Size matters!”</h4>
<p>The size of the standard integer types depend on the compiler and architecture and not on the C standard.<br>
Recently the C99 standard has introduced some types that have an unambiguous size (e.g., <code>uint8_t</code> for an 8-bit <code>unsigend</code> integer).</p>
<p>In order to use these types in our code we should include <code>stdint.h</code> with:</p>
<pre><code>	#include &lt;stdint.h&gt;
</code></pre>
<p>Not all 8-bit cross compiler support these types.</p>
<p>Fortunately for most 8-bit compilers we have the following situation:</p>

<table>
<thead>
<tr>
<th>type</th>
<th>number of bits</th>
<th>equivalent type</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>unsigned char</code></td>
<td>8</td>
<td><code>uint8_t</code></td>
</tr>
<tr>
<td><code>unsigned short</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
</tr>
<tr>
<td><code>unsigned int</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
</tr>
<tr>
<td><code>unsigned long</code></td>
<td>32</td>
<td><code>uint32_t</code></td>
</tr>
</tbody>
</table><p>Therefore we must:</p>
<ul>
<li>whenever possible use <code>unsigned char</code> (or <code>uint8_t</code>) for arithmetic operatoins;</li>
<li>use <code>unsigned char</code> (or <code>uint8_t</code>) and <code>unsigned short</code> (or <code>uint16_t</code>) for all other operations and avoid all 32-bit operations.</li>
</ul>
<p>Remark: When the fixed-size types are not available we can introduce them by using <code>typedef</code>:</p>
<pre><code>	typedef unsigned char uint8_t;
	typedef unsigned short uint16_t;
	typedef unsigned long uint32_t;
</code></pre>
<h3 id="choice-of-the-operations">Choice of the operations</h3>
<p>When writing code for an 8-bit architecture we must avoid inefficient operations or operations that force us to use inefficient types (such as <code>signed</code> or 32-bit types).</p>
<h4 id="avoid-signed">Avoid <em>signed</em></h4>
<p>In particolar, it is often possible to rewrite the code in a way to avoid subtractions or when this is not possible, we can at least have a code that does not produce negative results.</p>
<h4 id="avoid-explicit-products">Avoid explicit products</h4>
<p>All the architectures under consideration, with the only exception of the Motorola 6809, do not have a product operation between two 8-bit values.<br>
Therefore, if possible, we should avoid the products or limit ourselves to products and divisions by power of 2 that we can implement with the <em>&lt;&lt;</em> e <em>&gt;&gt;</em> operators:</p>
<pre><code>	unsigned char foo, bar;
	...
	foo &lt;&lt; 2; // multiply by 2^2=4
	bar &gt;&gt; 1; // divide by 2^1=2
</code></pre>
<h4 id="rewrite-some-operations">Rewrite some operations</h4>
<p>Other operations such as <em>modulo</em> can be rewritten in a more efficient way for the 8-bit systems by using bit-wise operators because the compiler is not always capable of optimizing these operations:</p>
<pre><code>	unsigned char foo;
	...
	if(foo&amp;1) // equivalent to foo%2
	{
		...
	}
</code></pre>
<h3 id="variables-and-parameters">Variables and parameters</h3>
<p>One of the greatest limitations of the MOS 6502-architecture is not the lack of registers as someone may think but it is the small size of its <em>hardware stack</em> (in <em>page one</em>: <code>$0100-01FF</code>),<br>
which makes unusable in C for managing the <em>scope</em> of variables and parameter passing.<br>
Therefore a C compiler for the MOS 6502 will most probably be forced to implement a <em>software stack</em>:</p>
<ul>
<li>to manage the scope of local variables,</li>
<li>to manage parameter passing.</li>
</ul>
<p>The other 8-bit architectures under our consideration may suffer less from this problem but the scope of local variables and parameter passing also have a cost when a <em>hardware stack</em> can be used.</p>
<h4 id="an-antipattern-may-help-us">An <em>antipattern</em> may help us</h4>
<p>One way to mitigate this problem is to reduce the use of local variables and passed paramters. This is clearly an <em>antipattern</em> and if we were to apply it to all our code we would get some <em>spaghetti code</em>.<br>
We must therefore wisely choose which variables deserve to be local and which variables can be declared as global.<br>
We would then have less re-usable code but we will gain in efficiency. I am <strong>NOT</strong> suggesting the use of just global variables and to renounce to all parameters in functions.</p>
<h4 id="do-no-use-re-entrant-function">[6502] Do no use re-entrant function</h4>
<p>The CC65 compiler for the MOS 6502 architecture provides the <code>-Cl</code> that interprets all local variables as <code>static</code>, i.e., global.<br>
This has the effect of avoiding the <em>software stack</em> for their scope. This also has the effect of making all the functions <em>non-reentrant</em>.<br>
In practice this prevents us from using recursive function. This is not a serious loss because recursion would be a costly operation, which we should avoid on 8-bit systems.</p>
<h4 id="use-page-zero">[6502] Use page zero</h4>
<p>Standard C provides the <code>register</code> keyword to give a hint to the compiler to use a register for a given variable.<br>
Most modern compiler simply ignore this keyword because their optimizers can choose better than the programmer.<br>
This is also true for the compilers under consideration but not for CC65 that uses this keyword to tell the compiler to use <em>pagina zero</em> for a given variable.<br>
The MOS 6502 can access this page more efficiently than any other memory area.<br>
The operating system already uses this page but the CC65 compilers leaves a few available bytes for the programmer.<br>
By default CC65 reserves 6 bytes in page zero for variables declared as <code>register</code>.<br>
One may think that all variables should be declared as <code>register</code> but things are <strong>NOT</strong> so simple because everything has a cost.<br>
In order to store a variable in page zero, some extra operations are required. Hence, page zero provides an advantage only for variables that are heavily used.<br>
In practice the two most common scenarios where this is the case are:</p>
<ol>
<li>parameters of type pointer to <code>struct</code> that are used at least 3 times within the function scope;</li>
<li>variables inside a loop that is repeated at least about 100 times.</li>
</ol>
<p>A reference with more details is: <a href="https://www.cc65.org/doc/cc65-8.html">https://www.cc65.org/doc/cc65-8.html</a></p>
<p>My personal advice is to compile and verify if the produced binary is shorter/faster.</p>
<h3 id="binary-structure">Binary structure</h3>
<p>If our program uses data in a specific memory area, it would be better to have the data already stored in the binary and have the load process of the binary copy the data at the expected locations without any code to do actual copying of the data.<br>
If the data is in the source code instead, we will have to copy them and we will also end up having them twice in memory.<br>
The most common case is the data for sprites and redefined characters or tiles.</p>
<p>Often but not always many MOS 6502 architectures use <em>memory mapped</em> graphics memory and so use common RAM for graphics data.<br>
On the other hand many Z80-based systems such as (MSZ, Specravideo, Memotech MTX, Sord M5, Tatung Einstein, etc.) use the Texas VDP graphics chip that has its own video memory.</p>
<p>Different compilers provide different tools to define the final binary structure.</p>
<h4 id="cc65-let-us-instruct-the-linker">[CC65] Let us instruct the linker</h4>
<p>It is possible to configure CC65’s linker through a .cfg file that describes the structure of the binary that we want to produce.<br>
This is not very simple and a description of the linker would go beyond the scope of this article. For details we refer to<br>
<a href="https://cc65.github.io/doc/ld65.html">https://cc65.github.io/doc/ld65.html</a><br>
We advice to read the manual and start by modyfing the default .cfg file in order to adapt it to one’s use-case.</p>
<h5 id="exomizer-can-help-us-also-on-this">Exomizer can help us (also) on this</h5>
<p>In some cases we may have graphics data in a memory area far from the code and have them both on the same binary. If we do this, we may end up with a “hole” between the two areas.<br>
A common example is provided by the C64 where graphics data may be in higher memory than the code.<br>
In this case I recommend the <em>exomizer</em> tool to compress the binary:  <a href="https://bitbucket.org/magli143/exomizer/wiki/Home">https://bitbucket.org/magli143/exomizer/wiki/Home</a></p>
<h4 id="z88dk-appmake-does-almost-everything-for-us">[Z88DK] <em>Appmake</em> does almost everything for us</h4>
<p>Z88DK makes our life easier and its power <em>appmake</em> tool automatically builds binaries in the correct format for most scenarios.<br>
Z88DK also allows the user to define <em>memory sections</em> and to redefine the binary “packagging” but doing this is quite complicated.<br>
This topic is treated in detail in:<br>
<a href="https://github.com/z88dk/z88dk/issues/860">https://github.com/z88dk/z88dk/issues/860</a></p>
<h3 id="code-on-multiple-files">Code on multiple files</h3>
<p>Usually separating a large code into multiple files is a good practice but it may produce poorer code for 8-bit optimizer because they do not perform <em>link-time optimization</em>, i.e., they cannot optimize code between two or more files but also optimize each file separately.<br>
For example we have a function that is called only once and the function is defined in the same file where it is invoked, then the optimizer may be able to <em>inline</em> it but this would never be possible if the functions were defined in a separate file.<br>
My advice is <strong>not</strong> to create one or few huge files but to take into account how separating the code into multiple files can affect the optimization.</p>
<h2 id="advanced-memory-use">Advanced memory use</h2>
<p>The C compiler usually produces a unique binary that contains both code and data, which will be loaded in specific memory locations (even with non contiguous memory areas).</p>
<p>In many architectures some RAM areas are use as <em>buffers</em> for the ROM routines or are used only in some special cases (e.g., some graphics modes).<br>
My advice is to study the memory map. For example for the Vic 20 we would have to look at:<br>
<a href="http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt">http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt</a></p>
<p>In particular we should look for:</p>
<ul>
<li>cassette buffer, keyboard buffer, printer buffer disk buffer, etc.</li>
<li>memory used by ROM routines and in particular by BASIC routines</li>
<li>memory areas used by special graphics modes</li>
<li>free small portions of free memory that are not usually used by code because they are not contiguous with the main code memory area.</li>
</ul>
<p>There memory areas could be used by our code if they do not serve their standard purpose in our use-case, e.g., if we do not intend to use the tape after the program has been loaded (including from the tape), then we can use the tape buffer in our code to store some variables.</p>
<p><em>Useful cases</em><br>
We list some useful memory areas for the Commodore 64 and a few memory-limited systems:</p>

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
<td>Commodore 16/116/+4</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore 16/116/+4</td>
<td>tape buffer</td>
<td>$0333-03F2</td>
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
<td>Commodore 64 &amp; Vic 20</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore 64 &amp; Vic 20</td>
<td>tape buffer</td>
<td>$033C-03FB</td>
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
<td>grabable hires memory</td>
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
<td>$7000-$73FF</td>
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
<p>In standard C we can only define some pointer and array variables at some specific memory locations.</p>
<p>In the following with give a theoretical example on how to define some of these pointer and array variables at address starting at <code>0xC000</code> where given a 5-byte <code>struct</code> type <code>Character</code> we want to also handle the following variables:</p>
<ul>
<li><code>player</code> of type  <code>Character</code>,</li>
<li><code>ghosts</code>, an <code>array</code> with 8 <code>Character</code> elements (40=$28 bytes)</li>
<li><code>bombs</code>, an array with 4 <code>Character</code> elements (20=$14 bytes)</li>
</ul>
<pre><code>	Character *ghosts = 0xC000;
	Character *bombs = 0xC000+$28;
	Character *player = 0xC000+$28+$14;
</code></pre>
<p>This generic solution with pointers does not always produce optimal code because it forces us to <em>dereference</em> our pointers and creates pointer variables (usually 2 bytes per pointer) that the compiler has to allocate in memory.</p>
<p>No standard solution exist to store any other type of variables in a specific memory area but the CC65 and Z88DK linkers provide a special syntax to do this and let us save hundreds or even thousands of precious bytes. Some examples are in<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory</a></p>
<p>In particolar we will have to create an Assembly file: a .s file (underCC65) or .asm file (under Z88DK) that we will link to our binary. In this file we will be able to assign each variable to a specific memory area.<br>
Remark: We need to <strong>add</strong> an <em>underscore</em> prefix to each variable.</p>
<p>CC65 syntax (Commodore Vic 20 example)</p>
<pre><code>	.export _ghosts;
	_ghosts = $33c
	.export _bombs;
	_bombs = _ghosts + $28 
	.export _player;
	_player = _bombs + $14
</code></pre>
<p>Z88DK syntax (Galaksija example)</p>
<pre><code>	PUBLIC _ghosts, _bombs, _player
	defc _ghosts = 0x2A00
	defc _bombs = _ghosts + $28 
	defc _player = _bombs + $14
</code></pre>
<p>CMOC provides the <code>--data=&lt;address&gt;</code> option to allocate all writable global variables at a given starting memory address.</p>
<p>ACK documentation does not say anything about this. We could nevertheless define pointer and array types at given free memory locations through the generic standard syntax.</p>
<h2 id="object-oriented-programming">Object-oriented programming</h2>
<p>Contrary to common belief, object-oriented programming is possible in ANSI C and can help up produce more compact compact in certain situations. There are complete object-oriented frameworks for ANSI C (e.g., Gnome is writtwn with <em>GObject</em>, which is one of these frameworks).</p>
<p>We can implement <em>classes</em>, <em>polymorphim</em> and <em>inheritance</em> very efficiently even for memory-limited 8-bit systems.</p>
<p>A detailed description of object-oriented programming goes beyond the purpose of this articile.<br>
Here we decribe how to implement its main features:</p>
<ul>
<li>Use<em>pointers to functions</em> to implement *polymorphic" methods, i.e., methods with <em>dynamic binding</em>, whose behavior is defined at <em>run-time</em>. It is possible to avoid the implementation of a <em>vtable</em> if we limit ourselves to classes with just one polymorphic method.</li>
<li>Use <em>pointers to <code>struct</code></em> and <em>composition</em> to implelent <em>sub-classes</em>: given a  <code>struct</code> A, we implement a sub-class with a <code>struct</code> B defined as a  <code>struct</code> whose <strong>first</strong> field is of type A. When passing pointers to such new <code>struct</code>, the C language guarantees that the <em>offset</em> of B are the same as the ones of A and therefore a pointer to B can be <em>cast</em> into a pointer to A.</li>
</ul>
<p>Example (taken from<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase</a>)<br>
Let us dfine <code>Item</code> as a sub-class of<code>Character</code> to which we add some variables and a polymorphic method <code>_effect()</code>:</p>
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
<p>We can then pass a pointer to  <code>Item</code> as if it were a pointer to  <code>Character</code> (by performing a simple <em>cast</em>):</p>
<pre><code>	Item *myIem;
	void foo(Character * aCharacter);
	...
	foo((Character *)myItem);
</code></pre>
<p>Why can we save memory by doing this?<br>
Because we may treat different, yet similar, objects with the same code and so avoid code duplication.</p>
<h2 id="compilazione-ottimizzata">Compilazione ottimizzata</h2>
<p>We won’t cover exaustively all compilation options of the cross-compilers under our consideration. We refer to their respective manuals for the derails.<br>
Here we give a list of options to produced optimized code on our compilers.</p>
<h3 id="aggressive-compilation">“Aggressive” compilation</h3>
<p>The following options will apply the highest optimizations to produce faster and above all more compact code:</p>

<table>
<thead>
<tr>
<th>Architecture</th>
<th>Compiler</th>
<th>Options</th>
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
<td><code>-SO3</code> <code>--max-alloc-node20000</code></td>
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
</table><h4 id="speed-vs-memory">Speed vs Memory</h4>
<p>The most common problem for many 8-bit systems is the presence of little memory for code and data. Usually optimizing for speed also improves memory usage but this is not always the case.<br>
In some other cases, our goal is speed even at the cost of extra memory.<br>
Some compilers provide options to specify our preference with respect to speed and memory:</p>

<table>
<thead>
<tr>
<th>Architecture</th>
<th>Compiler</th>
<th>Options</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><code>--opt-code-size</code></td>
<td>Optimize memory</td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>--opt-code-speed</code></td>
<td>Optimize speed</td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-Oi</code>, <code>-Os</code></td>
<td>Optimize speed</td>
</tr>
</tbody>
</table><p><strong>Known problems</strong></p>
<ul>
<li>CC65: <code>-Cl</code> prevents recursive functions</li>
<li>CMOC: <code>-O2</code> has bugs</li>
<li>ZSDCC: has bugs that do not depend on the options and has specific bugs that are triggered by <code>-SO3</code> when no <code>--max-alloc-node20000</code> option is provided.</li>
</ul>
<h3 id="safer-optimization">Safer optimization</h3>
<p>In order to avoid these problems and reduce compilation time (above all for the Z80 architecture) we suggest:</p>

<table>
<thead>
<tr>
<th>Architecture</th>
<th>Compiler</th>
<th>Options</th>
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
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>-O1</code></td>
</tr>
</tbody>
</table><h2 id="avoid-linking-useless-code">Avoid linking useless code</h2>
<p>Our compilers will not always be able to detect and remove unused and useless code from the binary. Therefore we must avoid to include it in the first place.</p>
<p>We can do even better with some of the compilers by instructing them to not include some standard libraries or even portions of the libraries that we are sure not to use.</p>
<h3 id="avoid-the-standard-library">Avoid the standard library</h3>
<p>In some meaningful cases, we could avoid using the standard library in order to reduce significantly the size of the produced binary.</p>
<h4 id="cpm-80-use-only-getchar-and-putcharc">[cp/m-80] Use only <code>getchar()</code> and <code>putchar(c)</code></h4>
<p>Replacing functions such as <code>printf</code> and <code>scanf</code> with just <code>getchar()</code> and <code>putchar(c)</code> is very useful when using ACK to produce <code>CP/M-80</code> binaries because otherwise ACK links a huge implementation of the standard C library.</p>
<h4 id="z88dk-special-pragmas-to-remove-code">[z88dk] Special <code>pragma</code>'s to remove code</h4>
<p>Z88DK provides several <em>pragma</em> commands to instruct the compiler and linker to not include some useless code.</p>
<p>For example:</p>
<pre><code>#pragma printf = "%c %u"
</code></pre>
<p>includes only <code>%c</code> and <code>%u</code> converts and excludes all the others.</p>
<pre><code>#pragma-define:CRT_INITIALIZE_BSS=0
</code></pre>
<p>does not generate code to inizialize the BSS memory area.</p>
<pre><code>#pragma output CRT_ON_EXIT = 0x10001
</code></pre>
<p>the program does not when it exists (e.g., to BASIC).</p>
<pre><code>#pragma output CLIB_MALLOC_HEAP_SIZE = 0
</code></pre>
<p>no <code>heap</code> memory (i.e., no <code>malloc</code> are possible).</p>
<pre><code>#pragma output CLIB_STDIO_HEAP_SIZE = 0
</code></pre>
<p>removes <code>stdio heap</code> (no file can be opened).</p>
<p>More examples are in:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk</a></p>
<h2 id="use-rom-routines">Use ROM routines</h2>
<p>Most of the 8-bit systems (almost all computers), have plenty of routines in ROM. It is important to know about them and use them when they are need. In order to use them explicitly in our code, we may have to write some <em>in line</em> Assembly in our C code (or use separate Assembly routines). How to do this is different in every dev-kit and we refer to their respective manuals for more details.</p>
<p>This is very important for systems that are not natively supported by the compilers and for which all input/output routines have to be written.</p>
<p>Example (taken from <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c</a>)</p>
<p>In order to display characters on the screen for the Thomson Mo5, Mo6 and Olivetti Prodest PC128 (which are not supported by CMOC), we can use the ROM routine by using a little in line Assembly code:</p>
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
<h4 id="libraries-may-already-use-rom-routines">Libraries may already use ROM routines</h4>
<p>Luckily we use ROM routines implicitly by just using the libraries that are provided by the dev-kit. This saves us a lot of RAM memory because the code is already stored in ROM.<br>
Nevertheless we must take into consideration that when we use a ROM routine may add some constraints in our code because we cannot modify them and they may use some auxiliary RAM locations (e.g., buffers) that we won’t be allowed to use.</p>
<h2 id="use-the-specific-hardware">Use the specific hardware</h2>
<p>As seen in the previous section, even if we could in C we should not forget the specific hardware. In some cases the hardware can help us write more compact and faster code.</p>
<h3 id="redefine-characters-only-when-needed">Redefine characters only when needed</h3>
<p>When we need very simple low level graphics, we could avoid redefining all our characters. For example we could exploit the extended ASCII character set  (ATASCII, PETSCII, SHARPSCII, etc.). Even if some redefined characters are necessary we could just redefine the needed ones and in same cases, we could use characters in ROM and in RAM at the same time  (see the Vic 20 example in the next section).</p>
<h3 id="exploit-the-graphics-chips">Exploit the graphics chips</h3>
<p>In some cases we can save some significant amount of memory if we know the graphics chips.</p>
<p>Example (Texas VDP TMS9918A chip such as the one on MSX, Spectravideo, Memotech MTX, Sord M5, etc.)<br>
Systems that use this chip have a special text color mode (<em>Mode 1</em>) where each character has as preassigned color. When using this text mode setting a character and its color is done by a single byte.</p>
<p>Example (Commodore Vic 20)<br>
The Commodore Vic 20 is a very special case because of its very limited RAM memory (total RAM: 5k, RAM available for the code: 3,5K) but it also comes with tricky ways to mitigate these limits:</p>
<ul>
<li>It also has color ram (1024 nibbles).</li>
<li>A large part of the non-code RAM can be used by the code (buffers, auxiliary locations, etc.)</li>
<li>The most surprising feature is that each chip can map some characters in RAM and some in ROM.</li>
</ul>
<p>Therefore, we do not use RAM for colors. We can map some variables in some non-code RAM areas.<br>
If we need n (n&lt;=64) redefined characters, we can just map 64 onto RAM by <code>POKE(0x9005,0xFF);</code>.<br>
We can then redefine up to 64 characters and keep 64 standard characters (defined in ROM).</p>

