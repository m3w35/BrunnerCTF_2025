---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** H4N5
> 
> It's 1974. Deep inside the crusty vaults of CrumbTrust Bank, a covert team of COBOL developers and pastry chefs created a failsafe for their most prized possession: The Marzipan Reverse Recipe.
> 
> This sacred document was too valuable to be stored on paper. So, they did the unthinkable - they encoded it and buried it in the production mainframe under a fake batch job titled IEBCAKED. Only those with knowledge of both banking ops and baking science could ever retrieve it.
> 
> Years later, the IEBCAKED job has mysteriously reappeared in the job output spool of an IBM z/OS LPAR. But what's left is a string of mysterious byte codes. It's clearly not hex, not ASCII... maybe something older... something only the graybeards of computing would recognize.
> 
> Can you decipher the original recipe and unlock the secret to the perfect butterbyte tart?
> 
> `d0-85-97-89-83-85-d9-6d-85-a2-99-85-a5-85-d9-6d-95-81-97-89-a9-99-81-d4-6d-85-88-e3-6d-a2-c9-6d-a2-89-88-e3-6d-84-95-c1-6d-a2-85-94-81-99-c6-95-89-81-d4-6d-95-d6-6d-f0-f7-f9-f1-6d-85-88-e3-6d-95-c9-6d-84-85-a2-e4-6d-a2-81-e6-6d-c3-c9-c4-c3-c2-c5-c0-99-85-95-95-a4-99-82`

# Walk Through

At first glance, this appears to be a string of bytes in hexidecimal notation, separated by a `-` character.

However, we read the prompt carefully so we know this is not the case.

## What the hell is an IBM z/OS LPAR?

Alright, so I tried for about a minute to research this stuff without using an LLM. But I have zero exposure to this old-school mainframe stuff. So, ChatGPT to the rescue!

According to Chat Gippity:

> If this came from **COBOL / IBM z/OS**, then the “hex” you’re looking at is almost certainly not plain **ASCII UTF-8** text. On IBM mainframes:
> - **Character data** is usually stored in **EBCDIC** (Extended Binary Coded Decimal Interchange Code), not ASCII.
> - A hex dump of such data will show bytes like `C1`, `C2`, etc., which represent letters in **EBCDIC**, not the ASCII you’re used to.
> - COBOL `COMP-3` (packed decimal) and binary fields will also appear in raw hex, but they aren’t meant to be decoded as text at all — they represent numeric fields.
> - The presence of `-` separators makes me think this came from a `DISPLAY` dump, a hex utility, or maybe even a `REXX` tool that prints data in grouped bytes.

## Translate this thing

Using `bash`, with the help of ChatGPT, I stumbled upon the following.

### Remove the `-` delimiters and revert from Hex to ASCII

	❯ echo 'd0-85-97-89-83-85-d9-6d-85-a2-99-85-a5-85-d9-6d-95-81-97-89-a9-99-81-d4-6d-85-88-e3-6d-a2-c9-6d-a2-89-88-e3-6d-84-95-c1-6d-a2-85-94-81-99-c6-95-89-81-d4-6d-95-d6-6d-f0-f7-f9-f1-6d-85-88-e3-6d-95-c9-6d-84-85-a2-e4-6d-a2-81-e6-6d-c3-c9-c4-c3-c2-c5-c0-99-85-95-95-a4-99-82' | tr -d '-' | xxd -r -p > raw.bin

#### Check the contents of `raw.bin`

	❯ cat raw.bin        
	Ѕmmmmmmmƕmmmmmmm%  

### Use the `iconv` command to "convert text from one character encoding to another"

	❯ iconv -f IBM256 raw.bin      
	}epiceR_esreveR_napizraM_ehT_sI_sihT_dnA_semarFniaM_nO_0791_ehT_nI_desU_saW_CIDCBE{rennurb%    

#### Reverse that shit

	❯ iconv -f IBM256 raw.bin | rev
	brunner{EBCDIC_Was_Used_In_The_1970_On_MainFrames_And_This_Is_The_Marzipan_Reverse_Recipe}

## Flag found

**brunner{EBCDIC_Was_Used_In_The_1970_On_MainFrames_And_This_Is_The_Marzipan_Reverse_Recipe}**

