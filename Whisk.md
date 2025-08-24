---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Crypto]]"
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** rvsms
> 
> Someone tried sabotaging our operation by "whisking” away the secret ingredient for the perfect brunsviger. All that's left on the workbench is this sticky note full of pastry-themed symbols and random letters.
> 
> Can you help us recover the secret ingredient?
> 
> File download: `crypto_whisk.zip`

# Walk Through

## Poking around

The `zip` archive contains a single file, `whisk.txt`. It appears to be a file containing UTF-8 text. It's contains 4 lines, 40 bytes, and 463 words.

```
❯ ls
whisk.txt
❯ file whisk.txt
whisk.txt: Unicode text, UTF-8 text
❯ wc whisk.txt
  4  40 463 whisk.txt
```

### Contents of the file

```
❯ cat whisk.txt
DR🥐 C🥐TZ🥐D 🧁SXZ🥐A🧁🥐SD 🧁C 🍰KE🍰FC K🍩M🥐. D🍩 O🍰Q🥐 🍰 Y🥐ZP🥐TD
OZ🥖SCM🧁X🥐Z, H🥐KD O🥖DD🥐Z E🧁DR OZ🍩ES C🥖X🍰Z, Y🍩🥖Z 🧁D 🍩M🥐Z DR🥐 E🍰ZH
A🍩🥖XR, 🍰SA K🥐D DR🥐 CFZ🥖Y C🥐🥐Y 🧁SD🍩 🥐M🥐ZF T🍩ZS🥐Z. D🍰CD🥐, CH🧁K🥐,
🍰SA Z🥐H🥐HO🥐Z: CR🍰Z🧁SX Y🍰CDZF N🍩F 🧁C H🍰SA🍰D🍩ZF.
OZ🥖SS🥐Z{S0_H0Z3_K🥖HYF_T1YR3Z}
```

### Substitution cipher

Looking at the cipher text, I notice a familiar pattern in the final word, `OZ🥖SS🥐Z{S0_H0Z3_K🥖HYF_T1YR3Z}`. This looks like the format of the flags.

#### Translation table

I'm using the following table to keep track of the translation table.

*Note: this is the final translation table. It was filled in bit by bit throughout the [[#Translation process]].*

| cipher | CLEAR | cipher | CLEAR | cipher | CLEAR | cipher | CLEAR |
| ------ | ----- | ------ | ----- | ------ | ----- | ------ | ----- |
| A      | D     | I      |       | Q      | K     | Y      | P     |
| B      |       | J      |       | R      | H     | Z      | R     |
| C      | S     | K      | L     | S      | N     | 🥖     | U     |
| D      | T     | L      |       | T      | C     | 🥐     | E     |
| E      | W     | M      | V     | U      |       | 🧁     | I     |
| F      | Y     | N      | J     | V      |       | 🍰     | A     |
| G      | G     | O      | B     | W      |       | 🍩     | O     |
| H      | M     | P      | F     | X      |       |        |       |

### Translation process

I put together a small Python script to translate the message as I go along, adding translations to the `cipher_map` as I find them.

```python
#! /bin/python3

cipher_map = {
		# initial letters from the "brunner" flag prefix.
        'O': 'B',
        'S': 'N',
        '🥖': 'U',
        '🥐': 'E',
        'Z': 'R'
        # ...
        # append to this list as I find new translations.
}

table = str.maketrans(cipher_map)

message = 'DR🥐 C🥐TZ🥐D 🧁SXZ🥐A🧁🥐SD 🧁C 🍰KE🍰FC K🍩M🥐. D🍩 O🍰Q🥐 🍰 Y🥐ZP🥐TDOZ🥖SCM🧁X🥐Z, H🥐KD O🥖DD🥐Z E🧁DR OZ🍩ES C🥖X🍰Z, Y🍩🥖Z 🧁D 🍩M🥐Z DR🥐 E🍰ZHA🍩🥖XR, 🍰SA K🥐D DR🥐 CFZ🥖Y C🥐🥐Y 🧁SD🍩 🥐M🥐ZF T🍩ZS🥐Z. D🍰CD🥐, CH🧁K🥐,🍰SA Z🥐H
🥐HO 🥐Z: CR🍰Z🧁SX Y🍰CDZF N🍩F 🧁C H🍰SA🍰D🍩ZF. OZ🥖SS🥐Z{S0_H0Z3_K🥖HYF_T1YR3Z}'

print(message.translate(table))
```

#### BRUNNER translation verification

At this stage of the code, I run it to be sure the "BRUNNER" translation is successful. We can see that it is.

```
❯ python3 whisktrans.py
DRE CETRED 🧁NXREA🧁END 🧁C 🍰KE🍰FC K🍩ME. D🍩 B🍰QE 🍰 YERPETDBRUNCM🧁XER, HEKD BUDDER E🧁DR BR🍩EN CUX🍰R, Y🍩UR 🧁D 🍩MER DRE E🍰RHA🍩UXR, 🍰NA KED DRE CFRUY CEEY 🧁ND🍩 EMERF T🍩RNER. D🍰CDE, CH🧁KE,🍰NA REHEHB ER: CR🍰R🧁NX Y🍰CDRF N🍩F 🧁C H🍰NA🍰D🍩RF. BRUNNER{N0_H0R3_KUHYF_T1YR3R}
```

#### Ten-letter keywords (*BRUNSVIGER*)

Next, I'll see if I can find some one-, two-, or three-letter words which may be a good starting place to guess. These are words like: I, am, you, are, a, ... Alternatively, by referencing the prompt, we know that the clear text message is a recipe for the perfect *brunsviger* (the length of which is 10). So, I look for words of the same length in the cipher text (aka `message`).

``` python
for word in message.split():
     if len(word) == 10:
             print(word)
```

The previous snippet yields the following results:

```
🧁SXZ🥐A🧁🥐SD
E🍰ZHA🍩🥖XR
```

Translating these words using the existing table of known characters reveals:

```
🧁NXREA🧁END
E🍰RHA🍩UXR
```

Notice the first and last letters of each word remained the same. Presumably, each letter in the cipher's alphabet has an associated English clear text letter. Therefore the second ten-letter cipher-word is not our *brunsgiver*, as it ends with the cipher-letter 'R'.

> Aside - I didn't need to translate the ten-letter cipher-words to notice this, but that's how it went down.

So, looking at the first ten-letter cipher-word, '🧁NXREA🧁END,' I'll adjust the [[#Translation table]] in my notes.

**Strikeout**. I already found 'B' earlier, from "BRUNNER". Ugh... I suck at crypto shit. :~D

#### Five-letter keywords (*SUGAR*)

I liked the idea I had, so didn't want to give up on it. I tailored my method by switching words. *sugar*  is stable ingredient in *brunsviger*, right?

#### Common keywords

Looking at the first word in the semi-translated cipher text at this point, "DRE," we know

- it ends in an 'E'
- it's three letters long
- it's at the beginning of a sentence
- 'D' is not the first letter and 'R' is not the second letter

Let's ask ChatGPT for a hand:

> [!quote] ChatGPT
> A common three-letter English word that can start a sentence and ends with “e” is:
> 
> **“The”**
> 
> - It’s extremely frequent.
> - Fits naturally at the beginning of a sentence.
> - Ends with `e`.
> 
> Other possibilities are rare or archaic; “the” is by far the most standard.

Add `'D': 'T'` and `'R': 'H'` to the [[#Translation table]], then add them to the translation table in the script. We're now left with the following semi-translated message. Progress!

```
❯ python3 whisktrans.py 
THE SETRET 🧁NGREA🧁ENT 🧁S AKEAFS K🍩ME. T🍩 BAQE A YERPETTBRUNSM🧁GER, HEKT BUTTER E🧁TH BR🍩EN SUGAR, Y🍩UR 🧁T 🍩MER THE EARHA🍩UGH, ANA KET THE SFRUY SEEY 🧁NT🍩 EMERF T🍩RNER. TASTE, SH🧁KE,ANA REHEHB ER: SHAR🧁NG YASTRF N🍩F 🧁S HANAAT🍩RF. BRUNNER{N0_H0R3_KUHYF_T1YH3R}
```

#### 1337

It's worth mentioning the distinction of numbers within the flag brackets. I'd assume these are direct translations to their common alphabetical counterparts in 1337-speak. 

This will be next point of focus.

## Much time

I've spent quite a lot of time on this challenge and these notes. I've pretty much got it solved from here. I'll just try to find the flag without recording every detail here.

### Got it

```
❯ python3 whisktrans.py
cipher message:
 DR🥐 C🥐TZ🥐D 🧁SXZ🥐A🧁🥐SD 🧁C 🍰KE🍰FC K🍩M🥐. D🍩 O🍰Q🥐 🍰 Y🥐ZP🥐TDOZ🥖SCM🧁X🥐Z, H🥐KD O🥖DD🥐Z E🧁DR OZ🍩ES C🥖X🍰Z, Y
🍩🥖Z 🧁D 🍩M🥐Z DR🥐 E🍰ZHA🍩🥖XR, 🍰SA K🥐D DR🥐 CFZ🥖Y C🥐🥐Y 🧁SD🍩 🥐M🥐ZF T🍩ZS🥐Z. D🍰CD🥐, CH🧁K🥐,🍰SA Z🥐H🥐HO 🥐Z: CR🍰Z🧁SX Y🍰CDZF N🍩F 🧁C H🍰SA🍰D🍩ZF. OZ🥖SS🥐Z{S0_H0Z3_K🥖HYF_T1YR3Z}

translation:
 THE SECRET INGREDIENT IS ALWAYS LOVE. TO BAKE A PERFECTBRUNSVIGER, MELT BUTTER WITH BROWN SUGAR, POUR IT OVER THE WARMDOUGH, AND LET THE SYRUP SEEP INTO EVERY CORNER. TASTE, SMILE,AND REMEMB ER: SHARING PASTRY JOY IS MANDATORY. BRUNNER{N0_M0R3_LUMPY_C1PH3R}
```

### Final script

``` python
#! /bin/python3

cipher_map = {
        'O': 'B',
        'S': 'N',
        '🥖': 'U',
        '🥐': 'E',
        'Z': 'R', 
        'C': 'S',
        'X': 'G',
        '🍰': 'A',
        'D': 'T',
        'R': 'H',
        '🧁': 'I',
        'A': 'D',
        'T': 'C',
        'Y': 'P',
        '🍩': 'O',
        'Q': 'K',
        'P': 'F',
        'K': 'L',
        # line of certainty
        'M': 'V',
        'F': 'Y',
        'E': 'W',
        'H': 'M',
        'N': 'J'
}

table = str.maketrans(cipher_map)

message = 'DR🥐 C🥐TZ🥐D 🧁SXZ🥐A🧁🥐SD 🧁C 🍰KE🍰FC K🍩M🥐. D🍩 O🍰Q🥐 🍰 Y🥐ZP🥐TDOZ🥖SCM🧁X🥐Z, H🥐KD O🥖DD🥐Z E🧁DR OZ🍩ES C🥖X🍰Z, Y🍩🥖Z 🧁D 🍩M🥐Z DR🥐 E🍰ZHA🍩🥖XR, 🍰SA K🥐D DR🥐 CFZ🥖Y C🥐🥐Y 🧁SD🍩 🥐M🥐ZF T🍩ZS🥐Z. D🍰CD🥐, CH🧁K🥐,🍰SA Z🥐H
🥐HO 🥐Z: CR🍰Z🧁SX Y🍰CDZF N🍩F 🧁C H🍰SA🍰D🍩ZF. OZ🥖SS🥐Z{S0_H0Z3_K🥖HYF_T1YR3Z}'

print("cipher message:\n", message)
print()
print("translation:\n", message.translate(table))
```

## Flag found

**BRUNNER{N0_M0R3_LUMPY_C1PH3R}**
