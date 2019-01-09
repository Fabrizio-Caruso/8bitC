---


---

<h1 id="portable-and-optimized-c-for-8-bit-systems">Portable and Optimized C for 8-bit Systems</h1>
<p>This article describes some optimizing techniques for ANSI C code for <strong>all</strong> 8-bit <em>vintage</em> systems, i.e., computers, consoles, hand-helds, scientific calculators from the end of the '70s until mid '90s and in particular for the systems based on the following <em>architectures</em> (and the derived and compatible architectures):</p>
<ul>
<li>Intel 8080 (*)</li>
<li>MOS 6502</li>
<li>Motorola 6809</li>
<li>Zilog Z80 (*)</li>
</ul>
<p>(*) The Zilog Z80 is an extension of the Intel 8080. Therefore an Intel 8080 binary is compatible with a Z80-based system but not the other way round.</p>
<p>Most of the techniques presented here remain valid on other 8-bit architectures such as the ones found in microcontrollers, e.g., on the Intel 8051.</p>
<p>The goal of this article is two-fold:</p>
<ol>
<li>present general techniques to <strong>optimize</strong> C code for <strong>all</strong> 8-bit systems;</li>
<li>present general techniques to write <strong>portable</strong> C code, i.e., valid and compatible code for <strong>all</strong> 8-bit systems, including systems that are not natively and explicitly supported by C compilers</li>
</ol>
<h2 id="preconditions">Preconditions</h2>
<p>This article is <strong>not</strong> a introduction nor a manual for the <em>C</em> language and has the following preconditions:</p>
<ul>
<li>knowledge of the <em>C</em> langiage;</li>
<li>knowledge of structured and object-oriented programming;</li>
<li>knowledge of basic use of compilers and linkers.</li>
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
<li>GCC-6809 (<a href="https://github.com/bcd/gcc">https://github.com/bcd/gcc</a>) for the Motorola 6809 (GCC adaptation);</li>
<li>GCC-6502 (<a href="https://github.com/itszor/gcc-6502-bits">https://github.com/itszor/gcc-6502-bits</a>) for the MOS 6502 (GCC adaptation);</li>
<li>SmallC-85 (<a href="https://github.com/ncb85/SmallC-85">https://github.com/ncb85/SmallC-85</a>) for the Intel 8080/8085 ;</li>
<li>devkitSMS (<a href="https://github.com/sverx/devkitSMS">https://github.com/sverx/devkitSMS</a>) for Sega consoles (Sega Master System, Sega Game Gear, Sega SG-1000).</li>
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
<p><strong><em>Example</em></strong> (from da <a href="https://www.cc65.org/doc/coding.html">https://www.cc65.org/doc/coding.html</a>):<br>
A <em>single pass</em> compiler may evaluate the following expression from left to right and miss the fact that <code>OFFS+3</code> is a constant:</p>
<pre><code>	#define OFFS   4
	int  i;
	i = i + OFFS + 3;
</code></pre>
<p>In this case it would be better to re-write <code>i = i + OFFS+3</code> as <code>i = OFFS+3+i</code> or <code>i = i + (OFFS+3)</code>.</p>
<h2 id="bit-specific-code-optimization">8-bit specific code optimization</h2>
<p>The C language has both high level constructs (such as <code>struct</code>, functions as parameters, etc.) and low level constructs (such as pointers, bitsise operators, etc.).<br>
This is not enough to make C a programming language well-suited for programming 8-bit systems.</p>
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
<h3 id="optimal-binary-structured">Optimal binary structured</h3>
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
<h3 id="codice-su-file-diversi">Codice su file diversi?</h3>
<p>In generale è bene separare in più file il proprio codice se il progetto è di grosse dimensioni.<br>
Questa buona pratica può però avere degli effetti deleteri per gli ottimizzatori dei compilatori 8-bit perché in generale non eseguono <em>link-time optimization</em>, cioè non ottimizzeranno codice tra più file ma si limitano ad ottimizzare ogni file singolarmente.<br>
Quindi se per esempio abbiamo una funzione che chiamiamo una sola volta e la funzione è definita nello stesso file in cui viene usata, l’ottimizzatore potre metterla <em>in line</em> ma non lo farà se la funzione è definita in un altro file.<br>
Il mio consiglio <strong>non</strong> quello di creare file enormi con tutto ma è quello di tenere comunque conto di questo aspetto quando si decide di separare il codice su più file e di non abusare di questa buona pratica.</p>
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
<td>Commodore Vic 20</td>
<td>tape buffer</td>
<td>$033C-03FB</td>
</tr>
<tr>
<td>Commodore Vic 20</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
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
</table><p>(*): Diversi tipi di buffer e memoria ausiliare. Per maggiori dettagli fare riferimento a: <a href="http://www.trs-80.com/trs80-zaps-internals.htm">http://www.trs-80.com/trs80-zaps-internals.htm</a></p>
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
<h2 id="la-programmazione-ad-oggetti">La programmazione ad oggetti</h2>
<p>Contrariamente a quello che si possa credere, la programmazione ad oggetti è possibile in ANSI C e può aiutarci a produrre codice più compatto in alcune situazioni. Esistono interi framework ad oggetti che usano ANSI C (es. Gnome è scritto usando <em>GObject</em> che è uno di questi framework).</p>
<p>Nel caso delle macchine 8-bit con vincoli di memoria molto forti, possiamo comunque implementare <em>classi</em>, <em>polimorfismo</em> ed <em>ereditarietà</em> in maniera molto efficiente.<br>
Una trattazione dettagliata non è possibile in questo articolo e qui ci limitiamo a citare i due strumenti fondamentali:</p>
<ul>
<li>usare <em>puntatori a funzioni</em> per ottenere metodi <em>polimorfici</em>, cioè il cui <em>binding</em> (e quindi comportamento) è dinamicamente definito a <em>run-time</em>. Si può evitare l’implementazione di una <em>vtable</em> se ci si limita a classi con un solo metodo polimorfico.</li>
<li>usare <em>puntatori a</em> <code>struct</code> e <em>composizione</em> per implementare sotto-classi: dato uno <code>struct</code> A, si implementa una sua sotto-classe con uno <code>struct</code> B definito come uno <code>struct</code> il cui <strong>primo</strong> campo è A. Usando puntatori a tali <code>struct</code>, il C garantisce che gli <em>offset</em> di B siano gli stessi degli offset di A.</li>
</ul>
<p>Esempio (preso da<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase</a>)<br>
Definiamo <code>Item</code> come un sotto-classe di <code>Character</code> a cui aggiungiamo delle variabili ed il metodo polimorfico <code>_effect()</code>:</p>
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
<p>e poi potremo passare un puntatore a <code>Item</code> come se fosse un puntatore a <code>Character</code> (facendo un semplice <em>cast</em>):</p>
<pre><code>	Item *myItem;
	void foo(Character * aCharacter);
	...
	foo((Character *)myItem);
</code></pre>
<p>Perché ci guadagniamo in termine di memoria?<br>
Perché sarà possibile trattare più oggetti con lo stesso codice e quindi risparmiamo memoria.</p>
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
<td><code>--opt-code-size</code></td>
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
<li>CMOC: <code>-O2</code> ha dei bug</li>
<li>ZSDCC: ha dei bug a prescindere dalle opzioni e ne ha altri presenti con <code>-SO3</code> in assenza di <code>--max-alloc-node20000</code>.</li>
</ul>
<h3 id="ottimizzazione-più-sicura">Ottimizzazione più sicura</h3>
<p>Per ovviare a i problemi sopramenzionati e ridurre i tempi di compilazione (soprattutto per l’architettura Z80) si consiglia:</p>

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
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>-O1</code></td>
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

