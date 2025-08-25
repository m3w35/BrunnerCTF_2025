---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Reverse Engineering]]"
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** Nissen
> 
> Baker Brian says he has a plan to make him super rich, but he refuses to share any details 😠 Can you access his Cake Vault where he keeps all his business secrets?
> 
> **Info:** The challenge has both a file download and a server to connect to. Please click "Start Challenge" below and wait a minute for your team's challenge instance to start, then connect to the server from a terminal with the command that appears.

# Walk Through

Alright! Let's download the file, `rev_baker-brian.zip`, and start the challenge server.

## Poking around

Looking around at the givens to get a lay of the land.

### Connect to Baker Brian's server

```
❯ ncat --ssl baker-brian-1521871a62e290b3.challs.brunnerne.xyz 443

		  🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂
		  🍰                            🍰
		  🍰  Baker Brian's Cake Vault  🍰
		  🍰                            🍰
		  🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂

Enter Username:

```
### Checkout the contents of the given .zip file

```
❯ unzip -l rev_baker-brian.zip
Archive:  rev_baker-brian.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
	0  2025-08-21 21:24   rev_baker-brian/
 1853  2025-08-21 06:32   rev_baker-brian/auth.py
---------                     -------
 1853                     2 files
```

Looks like we may be able to use the `auth.py` file to authenticate to Baker Brian's server and steal his secrets.

### Read the `auth.py` file

Alright, cool. This looks like the plaintext code for the authentication program running on Baker Brian's server.

#### The username is hard-coded in the program

```python
# <snip>

# Make sure nobody else tries to enter my vault
username = input("Enter Username:\n> ")
if username != "Br14n_th3_b3st_c4k3_b4k3r":
    print("❌ Go away, only Baker Brian has access!")
    exit()

# <snip>
```

Username: **Br14n_th3_b3st_c4k3_b4k3r**

### Determine the password

We can find the password by stepping through the code to deduce what the `auth.py` program will accept as the valid password.

```python
# <snip>

# Password check if anybody guesses my username
# Naturally complies with all modern standards, nothing weak like "Tr0ub4dor&3"
password = input("\nEnter password:\n> ")

# Check each word separately
words = password.split("-")

# <snip>
```

According to this first snippet, the password is a string of words separated by the `-` character.

#### Word 1

```python
# <snip>

# Word 1
if not (
    len(words) > 0 and
    words[0] == "red"
):
    print("❌ Word 1: Wrong - get out!")
    exit()
else:
    print("✅ Word 1: Correct!")

# <snip>
```

Word 1 is "**red**".

#### Word 2

```python
# <snip>

# Word 2
if not (
    len(words) > 1 and
    words[1][::-1] == "yromem"
):
    print("❌ Word 2: Wrong - get out!")
    exit()
else:
    print("✅ Word 2: Correct!")

# <snip>
```

Since I don't know python syntax off the top of my head, I'll just open an interpretive python session to help me understand what's going on with the code.

```
>>> 'yromem'[::-1]
'memory'
>>> 'yromem'[::-1] == 'memory'
True
>>> 'memory'[::-1] == 'yromem'
True
```

Word 2 is "**memory**"

#### Word 3

```python
# <snip>

# Word 3
if not (
    len(words) > 2 and
    len(words[2]) == 5 and
    words[2][0] == "b" and
    words[2][1] == "e" and
    words[2][2:4] == "r" * 2 and
    words[2][-1] == words[1][-1]
):

# <snip>
```

```
>>> word3 = '12345'
>>> len(word3)
5
>>> word3[0]
'1'
>>> word3[1]
'2'
>>> word3[2:4]
'34'
>>> word3[-1]
'5'
>>> words
['red', 'memory']
>>> words[1][-1]
'y'
```

Word 3 is "**berry**"

#### Word 4

``` python
# Word 4
if not (
    len(words) > 3 and
    words[3] == words[0][:2] + words[1][:3] + words[2][:3]
):
```

```
>>> words
['red', 'memory', 'berry']
>>> words[0][:2] + words[1][:3] + words[2][:3]
'remember'
```

Word 4 is "**remember**"

## Sign into the server with the discovered credentials

```
❯ ncat --ssl baker-brian-1521871a62e290b3.challs.brunnerne.xyz 443

              🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂
              🍰                            🍰
              🍰  Baker Brian's Cake Vault  🍰
              🍰                            🍰
              🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂

Enter Username:
> Br14n_th3_b3st_c4k3_b4k3r

Enter password:
> red-memory-berry-remember
✅ Word 1: Correct!
✅ Word 2: Correct!
✅ Word 3: Correct!
✅ Word 4: Correct!

Welcome back, Brian! Your vault has been opened:

    🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂
    🍰                                                  🍰
    🍰  Plan: Based on serious, cutting-edge research:  🍰
    🍰     https://pubmed.ncbi.nlm.nih.gov/29956364/    🍰
    🍰  start selling cookies to university professors  🍰
    🍰    they can give students for better ratings     🍰
    🍰                                                  🍰
    🍰         Daily XKCD: https://xkcd.com/936/        🍰
    🍰                                                  🍰
    🍰     Flag: brunner{b4k3r_br14n_w1ll_b3_r1ch!}     🍰
    🍰                                                  🍰
    🎂🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🍰🎂

```

## Flag found

**brunner{b4k3r_br14n_w1ll_b3_r1ch!}**