## Name

    zmakebas - convert text file into Spectrum Basic program

## Synopsis

```shell
zmakebas [-hlp3rv] [-a startline] [-i incr] [-n speccy_filename] [-o output_file] [-s line] [input_file]
```

## Description

zmakebas converts a Spectrum Basic program written as a text file into an actual speccy
Basic file (as a .TAP file, or optionally a raw headerless file). By default, input comes
from stdin, and output goes to 'out.tap'.

Using zmakebas rather than (say) writing the Basic in an emulator means you can write
using a nicer editor, and can use tools which work on text files, etc. Also, with the '-l'
option you can write without line numbers, using labels in their place where necessary.

The program was originally intended to be used simply to make little loader programs, so
they wouldn't have to be sourceless binaries. However, I went to a fair amount of effort
to make sure it'd work for bigger, more serious programs too, so you can also use it for
that kind of thing.

## Options

|option|comments|
|------|--------|
|`-a`|Make the generated file auto-start from line startline. If '`-l`' was specified, this can be a label, but don't forget to include the initial '@' to point this out.|
|`-h`|Give help on command line options.|
|`-i`|In labels mode, set line number increment (default 2).|
|`-l`|Use labels rather than line numbers.|
|`-n`|Specify filename to use in .TAP file (up to 10 chars), i.e. the filename the speccy will see. Default is a blank filename (10 spaces).|
|`-o`|Send output to `output_file` rather than the default '`out.tap`'. Use '-' as the filename to output on stdout.|
|`-r`|Write a raw headerless Basic file, rather than the default .TAP file.|
|`-s`|In labels mode, set starting line number (default 10).|
|`-p`|Output .p instead (set ZX81 mode).|
|`-3`|Write a +3DOS compatible Basic file, rather than the default .TAP file.|


## Input format

The input should be much as you would type into a speccy (a 128, to be precise), with the
following exceptions:

Lines starting with '#' are ignored. This allows you to insert comments which are not
copied into the output Basic file.

Blank lines are ignored.

Case is ignored in keywords - 'print', 'PRINT', and 'pRiNt' are equivalent.

You can optionally use 'randomise' as an alternative to 'randomize'.

You can get hex numbers by using 'bin' with a C-style hex number, e.g. to get 1234h you'd
use `bin 0x1234`. (It appears in exactly that form in the speccy listing, though, so don't
use it if you want to be able to edit the output program on a speccy.)

You can get a pound sign (character 96 on a speccy) by using a backquote ( \` ).

One input line normally equals one line of Basic, but you can use backslash as the last character of a line to continue the statement(s) on the next input line.

### User Defined Graphics (UDG)

Rather than literally inserting block graphics characters and UDGs as you would on a speccy, you should use an escape sequence. These begin with a backslash ( `\` ). To get a UDG, follow this backslash with the UDG's letter, in the range 'a' to 'u' ('t' and 'u' will only have the desired effect if the program is run on a 48k speccy or in 48k mode, though); both upper and lowercase work. To get the copyright symbol, follow it with '*'. To get a block graphics character, follow it with a two-character 'drawing' of it using spaces, dots, apostrophes and/or colons. (For example, you'd get character 135 with '`\':`', and character 142 with '`\:.`'). To get a literal '@', follow it with '@'. (This is needed only if the '-l' option was given, but works whether it was or not.) To specify a literal eight-bit character code to dump into the Basic output file directly (to use for embedded colour control codes and the like), use braces and a C-syntax number e.g. '`\{42}`' for decimal, and '`\{0x42}`' for hex. Finally, as usual with such things, you can get a literal backslash by following the first backslash with another.

### Automatic line numbers
If the '-l' option was given, line numbers must be omitted. Instead these are automatically generated in the output, and you can use labels where necessary as substitute line numbers for 'goto' commands etc. A label is defined with the text `@label:` at the beginning of a line (possibly preceded by whitespace). It can be referred to (before or after) with `@label`. Any printable ASCII character other than colon and space can be used in a label name. Here's an example of how labels work, showing both the input and (listing of) the output - first, the input:

```basic
goto @foo
print "not seen"
@foo: print "hello world"
```

Now the output:

```basic
10 GO TO 14
12 PRINT "not seen"
14 PRINT "hello world"
```

Note that case is significant for labels; 'foo' and 'FOO' are different.

You can specify a line number in labels mode. This can be useful when defining subroutines that we want to be at specific locations. You can also write:

```
9000+2 DATA "Blah"
DATA "Foo"
DATA "Bat"
```
to tell it to start at 9000 and increment by 2 for subsequent lines.


## Bugs

There's almost no syntax checking. To do this would require a complete parser, which would be overkill I think. What's wrong with `'C Nonsense in BASIC'` as a syntax check, anyway? :-)

Excess spaces are removed everywhere other than in strings and rem statements. I think this is generally what you'd want, but it could be seen as a bad thing I s'pose.

Labels are substituted even in string literals. That's arguably a feature not a bug - the problem is, the label name has to be followed by whitespace or a colon or EOL when referenced, which is fine for more normal references but is less than ideal for references in strings.

In the label-using mode, two passes are made over the input, which usually means the input must be from a file. If you like making one-liner Basic programs with 'echo' and the like, I'm afraid you'll have to use line numbers. :-)

The inline floating-point numbers which have to be generated are not always exactly the same as the speccy would generate - but they usually are, and even when they're not the difference is extremely small and due to rounding error on the speccy's part. For example, 0.5 is encoded by the speccy as `7F 7F FF FF FF` (exponent -1, mantissa approx. `0.9999999997672`) and by zmakebas as `80 00 00 00 00` (exponent 0, mantissa 0.5).

zmakebas has most of the same (parsing) problems, relative to the original basic editor, that the 128 editor has. Specifically, you can't use variable names which clash with reserved words, so e.g. 'ink ink' doesn't work; and certain tightly-packed constructions you might expect to work, like `chr$a`, don't (you need a space or bracket after `CHR$`). These can be more of a problem with zmakebas though, due to the lack of syntax checking.

The way tokenisation is done is sub-optimal, to say the least. If you ran this code on a Z80, even the 128 editor's tokenisation would seem quick in comparison. (Here's a hint of the full horror of it - program lines take exponentially longer to tokenise the longer they are.) However, since I never had a conversion take more than about a second on my old 486 (it took a second for a 10k program), it hardly seems worth the effort of fixing.

zmakebas has no problem with translating BIN numbers of more than 16 bits, unlike the speccy, though numbers with more than 32 significant bits can only be approximated, and on machines where 'unsigned long' is no more than 32 bits they'll be very approximate. :-) (If this sounds confusing, you should note that BIN numbers are translated when entered, and only the 5-byte inline form is dealt with at runtime. This also explains why the speccy tolerates the 'bin 0x...' construction.)

On machines without FP hardware, zmakebas will be rather slow (this is due to the need to generate inline FP numbers). 

Since Basic is an acronym, pedants will doubtless insist I should write it as 'BASIC'. But we live in a world with 'laser' etc., and at least I can be bothered to capitalise the thing, right? :-)


## AUTHOR

Russell Marks (russell.marks@ntlworld.com).

See [ABOUT](./ABOUT) for more info on forks and versions. 
