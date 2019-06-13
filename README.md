## Unicode encoding


### A bit of history

Initially, the majority of computer systems used the Latin alphabet which was quite suitable for communication with the user. For all the numbers and conventional signs max of 7 bits are quite enough.

Over time, there was a need for support for additional characters from other languages and it initiated the use of 8 bit for their encoding. Every operation system at that time defined its own encodings for different languages. 

Conversions between languages became a real problem as each language has its own encodings. Russian letters, for example, can be encoded in three ways at once (866 in DOS, 1251 in Windows, and sometimes you can still find KOI8). At the same time, the Russian letters in the still common version of the DOS866 encoding are located “randomly”. For instance, the code of the letter “п” is not followed by the code of the letter “р”.

More problems were added with appearance of fonts. Now if the user worked with several languages, he must install options for each language. Another headache was the Asian languages ​​with their thousands of characters.

To solve this problem in 1991, it was proposed to introduce a single standard for encoding characters. To begin with, the ISO 10646 standard, a defining character set, was created and, in parallel, a consortium of software manufacturers called Unicode was formed. Both projects have agreed to maximally combine their ideas about symbols. The character set defined by the standards was named UCS (Universal Character Set). Initially, a set of 65535 (0xFFFF) (hereinafter, C will be used for notation of the numbers) characters was defined and used 2 bytes. As the number of characters increased it became 4.


### General principles



*   The first 127 characters codes match the ASCII character set.
*   Characters with codes 0x0000-0xFFFF contain the characters of the most common scripts, including Asian hieroglyphs.
*   If the character code exceeds 0xFFFF, then the high byte specifies the so-called plane. Accordingly, the main part of the characters is in the plane 0, also defined as Basic Multilingual Plane (BMP).
*   There are a total of 17 planes and, accordingly, the maximum character code 0x10FFFF.
*   Some characters can be formed by combining several Unicode characters using so-called control characters:

    **Example.**


    You can obtain German letter “ä”  in two ways:


    -The first one is to use direct code 0xE4;


    -The second one implies the use of  two characters 0x61 (a) and 0x308 (̈);


     You also can combine more than two characters.


To maintain compatibility with early versions of the encoding of such characters a similar character education method has been introduced, with ISO 8859 in particular.

Different versions of the Unicode encoding can be used for storage in the memory of computing systems: UTF-16, UTF-32, UTF-8.

More details can be found on the page of the Unicode Consortium [www.unicode.org](http://www.unicode.org).


### UTF-16

Historically, the first proposed format was UTF-16, in which, initially, 2 bytes were used for each character.

In the first version of the Unicode standard, there were only 65535 (0xFFFF) characters and 2 bytes were allocated for each character. The ability to append more bytes was added with the introduction of additional character planes. 

The principle is quite simple:



*   If the code is not in the range 0xD800-0xDFFF, then this is just a character code from the zero plane, i.e. from 0x0000-0xFFFF code range. 
*   If the character code is in range 0xD800-0xDFFF, then it is a character from the “extended plane” and consists of 4 bytes, you must subtract the value of the first 16-bit word 0xD800. From the second 16-bit word, deduct 0xDC00. The first number forms the high 10 bit of the character code, the second low 10 bit.

And there is a "small" problem.  Historically in Intel architecture and the rest (Motorola and others) the order of bytes in a word is different. Accordingly, the encodings were divided into: UTF-16BE and UTF-16LE.

How to determine the byte order of a specific file? At the beginning of the file two bytes0xFEFF are written.  If the first word read from file is 0xFEFF, the bytes have “big-endian” order, otherwise, if 0xFFFE, we will have “little-endian” byte order.


## UTF-8

UTF-16 encoding provides Unicode encoding, but is completely incompatible with single-byte encodings. UTF-8 was developed In order to minimize the difference with standard ASCII encoding.

In UTF-8, a character can be encoded from 1 to 4 bytes. The first 127 characters are fully ASCII-encoded. The upper bits specify the number of bytes required to encode a single character.

For files containing only ASCII UTF-8 characters, the encoding is identical to ASCII.


### The principle of encoding characters



*   If the high bit is 0, this is an ASCII character and it takes 1 byte.
*   If the three high bits are 110, this is one of the characters in the range 0080-07FF and they take 2 bytes. For the character code is given 11 bits.
*   If the four high-order bits are 1110, this is a character in the range 0800-FFFF and it takes 3 bytes. For a character code 16 bits are allocated.
*   If the five most significant bytes are 11110, this is a character in the range 10000-10FFFF and it takes 4 bytes. 21 bits are allocated for the character code.
*   The upper bits of a character code are placed in low bits of first byte: for 1 byte character code, this is 7 bits, for 2 - 5 bits, for 3 - 4 bits, for 4 - 3 bits. The remaining bits are in the 6 lower bits of the subsequent data bytes. The upper 2 bits of the subsequent bytes are always 10.

**Examples:**



*   English letter Z. Unicode character 0x005A

0x5A -  0101 1010



*   Russian letter И. Unicode character 0x042F

The binary code of the symbol 010000101111 is divided into two groups of bits: 10000 and 101111. 

Next, 110(an indicator) is added to the first group of bit, two bytes are used, bits 10 are added to the second.

So it turns out in UTF-8.

0xD0 AF - 1101 0000 1010 1111

The infinity symbol ∞. Code 0x221E.

The binary code is 00100010 00011110. We break it  into three groups 0010, 001000, 011110. Add the high-order bit, respectively, 1110 (three bytes), 10 and 10.

That we get is

11100010 10001000 10011110 – 0xE2 88 9E.

UTF-8 encoding gives a noticeable decrease in file size for typical European languages. For Russian text, the file size will be almost twice as large as, for instance, Windows1251 (but spaces, line breaks, numbers and punctuation will still be single-byte). For HTML pages, which are almost entirely composed of tags, a very significant decrease is achieved compared to UTF-16.

In addition, UTF-8 is insensitive to the byte order of the system and has protection against   failures. If one byte is lost in transmission in UTF-16, the entire further stream of characters will turn into a “mess”. In UTF-8, in case of loss of the first byte specifying a multibyte character, the correct decoder will simply discard 1 or several bytes with high bits 10 and continue to read characters correctly. Even the “wrong” decoder will make 1-2 incorrect characters and continue to read them correctly.

The complexity of UTF-8 consists in calculating the length of the string. To insert a substring after 105 characters you will have to decode the string first, to determine which character will have the number 105. 


## UTF-32

In UTF-32, 4 bytes are provided for each character, and each character is encoded simply by a 32-bit number. Despite the memory overrun (high byte will always be 0, but some software used this byte for internal purposes), UTF-32 is used in software internal for higher text processing speed. No decoding / encoding of multibyte sequences is required, indexing is simplified.

However, in reality, due to the presence of multi-character combinations, it will still be necessary, as in UTF-8, to count the number of characters from the first line.


## Implementation levels of Unicode support in applications

The standard defines three levels of implementation in applications:

**Level 1.**

Combining characters and Korean writing is not supported.

**Level 2**

Part of combined characters is supported.

**Level 3**

The entire Unicode character set is supported.


## Normalization

The Unicode standard provides for such a thing as normalization of character strings.

The essence of normalization is to convert character code sequences into a unified sequence of codes for correct string comparison.

As mentioned above, the same symbols can be represented in different ways. When on screen/ print they look the same. In order to be able to properly compare such strings normalization is applied.

According to the standard, four forms of normalization are defined:

- Form D - all characters that may be compiled of several character codes are divided into such sequences in a specific order;

- Form C - first, a breakdown is made as in form D. Then all sequences of character codes are converted, if possible, into the minimum number of character codes;

- Form KD - similar to form D, but a series of characters are divided into several characters, some specific characters are converted into ordinary. Asian characters are converted into sequences of Latin characters;

- The KC form is similar to C, but the transformation is performed as in the form of KD.


## Unicode implementation in operating systems


### Windows

Starting from Windows NT, WinApi already had three options for calling API functions: normal (the encoding depends on the compiler), with single-byte strings (such functions have suffix A) and using double-byte Unicode (suffix W).

Windows CE initially uses UTF-16.


### Unix

Since the early 2000s, most Unix systems have adopted UTF-8 as the standard encoding.


## Implementation in languages



*   In C and C ++ everything is very democratic - there is a single-byte char and two-byte char16_t and 32-bit char32_t;
*   In Java and C#, char has a size of 16 bits and uses UTF-16;
*   Until 2009, Delphi had separate char types of 8 bits and widechar 16 bits. Since 2009, the char has become 16-bit;
*   Javascript isn`t implementing one char type and is always supporting Unicode in string.

    At this time almost all platforms support as minimum level 1 Unicode implementation.  



## Summation



*   if you need to process very large texts, and the memory consumption is critical, you need to use UTF-8;
*   If processing speed is important for typical European languages, you need to use UTF-16. In addition, support for UTF-16 is already well implemented in most programming languages;
*   If you need to support characters from the range of more than 0xFFFF and at the same time provide maximum speed, you need UTF-32.

<!-- Docs to Markdown version 1.0β17 -->
