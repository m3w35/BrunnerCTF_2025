---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Web]]"
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** The Mikkel
> 
> Ahh, the Brunnerne company.  
> But they have a secret, hidden away from robot search.

# Walk Through

This is a "Personal Challenge Instance". Looks like they just spun up a web server for me to hack on. Let's check it out.

![[Pasted image 20250822200239.png]]

First, I clicked the **Check out our locations** button. I noticed a `#` was appended to the URL.

## Poking around the directory structure

Try to manually check different files on the web-server's backend that may be exposed.

	https://where-robots-cannot-search-15dbf7a25dd6d51c.challs.brunnerne.xyz/about

Nothing came from the 'about' page, besides an Internal Server Error.

	https://where-robots-cannot-search-15dbf7a25dd6d51c.challs.brunnerne.xyz/robots.txt

Bingo!

![[Pasted image 20250822200620.png]]

Navigate to the `/flag.txt` file to see if we can read it.

![[Pasted image 20250822200658.png]]

Nice.

## Flag found

**brunner{r0bot5_sh0u1d_nOt_637_h3re_b0t_You_g07_h3re}**

