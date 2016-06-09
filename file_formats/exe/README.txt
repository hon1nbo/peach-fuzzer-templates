/********************************************************
** EXE Template README
** Author: Hon1nbo
** Template for 32bit EXE Portable Executable Fuzzing
** Created using Microsoft specification (included)
	and analyzing of files
** Current Status: TEMPORARILY FROZEN.
	Template is mostly complete for general use,
	but run times are too long for current needs.
*********************************************************/
# Reading Tip: this readme is laid out for syntax highlighters
# I use notepad++ with C++ highlighting enabled.

/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\
 #Table of Contents:           *
                               *
 * Info                        *
 * TODOs                       *
 * Template Statistics         *
 * Usage                       *
 * PE EXE Resources            *
 * sampleProgram.exe details.  *
	#Building                  *
	#Customizing               *
/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\


**************
#### Info ####
**************
{
* This template was designed for fuzzing 32bit 'PE EXE' files.
	It covers a significant amount, but not quite all, of the specification.
	I was authoried to release this outside of Cigital as we do not have a reason to keep it internal.
	Being only 32bit, it does not serve as much use. However, it can be adapted for 64bit and makes a good academic study.
	It was created in the Fall of 2012, and only recently got dug off an old repo server that is being purged.


* This template, while valid, will not give a "full" validation in the Peach Validator.	
	This is due to the FIFO/LIFO placement workaround involving the "dummyB" Block.
	See the Cigital Peach Wiki for more information on the FIFO/LIFO placement problem.
	
* For the time being, with regard to major sections directories: Raw data sizes, Raw data offsets and RVAs are set with mutable="false" 
	since I still need to work out the chicken-egg scenario in handling them. For smaller information within sections it is still mutable.
	Relevant areas that may or may not be affected are marked with '<!-- mutable? -->'

* Not all EXE files use the specification as Microsoft lists it, so some parts
	may vary from file to file.

* A sample EXE, labled "sampleProgram.exe", is included. I custom made the program for use with this fuzzer.
	it naturally included a lot of sections, though I manually added .edata from gedit.exe and .reloc sections from firefox.exe
	Since these additional sections are not native to the sampleProgram, I do not know how effective they will be.
	The sampleProgram may need some features added to force them in natively.
}
***************
#### TODOs ####
***************
{
* Implement calculation of RVAs and Virtual Sizes as needed. This is a tricky conundrum, as the RVAs need te data to be calculated,
	as well if the RVAs are mutated the data is useless and won't be used by the loader.
	So there is a need to balance proper calculation Vs. mutation.

* Code Signing. Currently the Certificate section of the program does not exist. This is because if the certificate mutates the exe simply won't run.
	If the program mutates and the certificate isn't recalculated, it won't run. Implementing proper certificate signing will need significant work.

* Padding. There are numerous locations where padding or alignment exists. Peach is incapable of performing alignment or true padding in current release. 
	and calculating these is difficult with peach or a python fixup as it is relative to the whole program.
	Right now there are a couple of fixed values in the template for the given sampleProgram.exe 
	(the template has these marked, as well as comments for some other samples)
	
}
*****************************
#### Template Statistics ####
*****************************
{
A dump of "-1 --debug" is included for sampleProgram.exe

sampleProgram.exe run through "peach.bat -c" results in the following:

[*] Total time to crack data: 238.97
[*] Building relation cache
-- Counting total test cases...
-- Count completed, found 25671219 tests
Test TheTest has 25671219 test cases

Total test cases for run DefaultRun is 25671219
}
***************
#### Usage ####
***************
{
To use the template a couple things have to be adjusted.

* First, if something other than sampleProgram.exe is used, the Blob "SectionPadding" needs the length adjusted acccordingly.
	it is the length from the end of the list item for Section_Table to the beginning of the first section of data.
 
* Second, if something other than sampleProgram.exe is used, read the comments in the exe.xml by searching for:
	"README Note 1" and "README Note 2"
 
* Unless you are testing each type of Section that is predefined, rename the dummy equivelents at the top of the template so they get dumped
	into a blob to save time.

* If you are using sample files larger than any included, 
	then check to make sure that the maxOccurs in sections such as idata/edata/rsrc/reloc are large enough
}
**************************
#### PE EXE Resources ####
**************************
{
* Official Microsoft documentation for pecoff (included in distribution)
	NOTE: there are some minor typos that affect things, as well as some missing information.
	First, there is the occasional skipped/duplicated address for some flags. This has been corrected in template.
	Second, the specification does not list the format of the x86 .pdata. I have yet to find documentation of it, or an example of use.
	
* Microsoft MSDN articles: These have been very good at explaining some of the format as well as filling in some details.
	part one: http://msdn.microsoft.com/en-us/magazine/bb985992.aspx
	part two: http://msdn.microsoft.com/en-us/magazine/cc301808.aspx

* PEDUMP: A program for analyzing PE EXE files made by the author of the above two articles on MSDN.
	The zip of the program is in the distribution since the microsoft link is broken.
}
***********************************
#### sampleProgram.exe details ####
***********************************
{
*sampleProgram.exe was compiled and modified just for this template.
*Icon and version information were included to give a wider attack surface, and it adds valuable template cracking data
without adding much time.
*If you want more realistic .edata and .reloc sections, re-write and compile the sampleProgram.c and ignore the second step.

//////////
#Building
//////////
{
First, we need to configure the sampleProgram.rc file. A sample one is included. It will run if you just set proper paths.
	NOTE: if you change the label of the icon, make sure [label length] % 8 = 1. This is due to a hack
	needed to make the sample program work. There are some variations from the specification that compilers
	tend to make use of, this is to make things easier for us right now. This is due to needing alignment on a certain boundry
	Also See 'README Note 1' and 'README Note 2' in the template.

using MinGW in command prompt on Windows 7 (what I used, your mileage may vary):

* "C:\MinGW\bin>windres 'path'/sampleProgram.rc -O coff -o 'path'/sampleProgram.res"
* "C:\MinGW\bin>gcc -o 'path'/sampleProgram.exe 'path'/sampleProgram.c 'path'/sampleProgram.res"
}
/////////////
#Customizing
/////////////
{
* You can now insert other sections from other EXEs to make the program more useful.
	edata from gedit and the reloc section from firefox are included in the distribution for this purpose.

(NOTE: since LordPE is of dubious origins, at least how I found out about and obtained it, I ran it in a VM)

* Using LordPE, open sampleFile.exe in the PE Editor

* Under 'Sections', right click on a section and select to load a section from disk.
* load the _edata file included in the source.
* right click _edata in the sections window and modify Section Headers.
* change "_edata" to ".edata"
* In the main "Sections" window, mark down the values for "VOffset" and "VSize" for use in later steps.
* repeat the above for "f_reloc" in the source distribution, renaming to ".reloc".

* In the main window of the PE Editor, click on "Directories"
* In the fields for "Exports", insert the values marked down earlier for the ".edata" section.
* Repeat above for "Relocations" using values marked down earlier for the ".reloc" section.
* Click on "Save" then "OK" to close.

* Click the "?" next to the "Checksum" field in the main PE Editor window. The value should change automatically.
*Click "Save" and then "Ok"

The sampleProgram.exe has now been finished and is ready for use.
}
}
