---
layout: post
title: Babies First ez Python4 Writeup
date: 2025-04-14 16:16 -0400
description: A writeup for a python bytecode challege in TexSAW ctf 2025
category: 
 - Writeups
 - TexSAW 2025
tags: 
 - Reverse Engineering
 - Python Bytecode
---

# Babies first ez Python4
**Rev • 448 pts • chal author: elli**
*Attachments:* chal.txt

## Thoughts Ig
Looking at the attachment, its pretty clear that we are supposed to reverse engineer some python bytecode. The concept is similar to a challenge in picoCTF 2024 that I solved (WeirdSnake) so I do have some experience in reading and understanding python bytecode.

That challenge however was only ~400 lines of bytecode, compared to this challenge which had ***8435** lines of bytecode*, so we definitely need a script to solve this.

## The Bytecode
First just glancing at the bytecode to get a sense of it's structure, we can see that there are 2 main sections, the function declarations, and the function definitions. The first section, function declarations, mainly consists of sections similar to this:
```
  1           LOAD_CONST             149 (('',))
              LOAD_CONST               1 (<code object aaabaa... at 0x73aa883a29d0, file "chal.py", line 1>)
              MAKE_FUNCTION
              SET_FUNCTION_ATTRIBUTE   1 (defaults)
              STORE_NAME               0 (aaabaa...)
```
This goes on for about 800 lines. However this isn't particularly useful as this bytecode translates to the following python code: 
```py
def aaabaa...(string=''):
    ...
```
Notice that we don't have any information on the function's contents, so this section is effectively useless to our goals, other than just to note that the function has an argument.

There is one small thing of note however, near the end of this section, around line 879, a more unique looking function showed up, this time with no arguments, and being named `zzz___...`
```
293           LOAD_CONST             147 (<code object zzz___... at 0x63bbc3064890, file "chal.py", line 293>)
              MAKE_FUNCTION
              STORE_NAME             146 (zzz___...)

297           LOAD_NAME              146 (zzz___...)
              PUSH_NULL
              CALL                     0
              POP_TOP
              RETURN_CONST           148 (None)
```
We can also see that this function is immediately being called, which implies that this is the "main" function.

Ok now on to the actual function definitions, most of the a/b functions look very similar with only a couple of differences. To understand the a/b functions, take for example this function:
```

Disassembly of <code object aaabaa... at 0x73aa883a29d0, file "chal.py", line 1>:
  1           RESUME                   0

  2           LOAD_FAST                0 (s)
              LOAD_CONST               1 ('')
              LOAD_ATTR                1 (join + NULL|self)
              LOAD_CONST               2 (<code object <genexpr> at 0x73aa883a2af0, file "chal.py", line 2>)
              MAKE_FUNCTION
              LOAD_CONST               3 ('____...')
              LOAD_ATTR                3 (split + NULL|self)
              LOAD_CONST               4 (' ')
              CALL                     1
              GET_ITER
              CALL                     0
              CALL                     1
              BINARY_OP                0 (+)
              RETURN_VALUE
```
To step though this bytecode, we need to keep track of the stack, so first we load a variable: `s` and since this has not been defined anywhere else, we can assume that this is the function parameter, then we load the empty string and then the attribute `join` which is a function on that empty string, and then we load a generator which is likely an inline comprehension, and then a string containing some amount of underscores, and then the function split on that string, and then another string with a single space, so the stack looks like this: [`s`, `''.join`, `<genexpr>`, `'____'.split`, `' '`], then we have a bunch of call statements and then an addition, so decompiling this manually results in some code similar to this:
```py
return s + ''.join(<genexpr> for w in '___...'.split(' '))
```
However we still don't know what the `<genexpr>` does, so the next step is to investigate that. Looking specifically at the genexpr referenced by the first function:
```
Disassembly of <code object <genexpr> at 0x73aa883a2af0, file "chal.py", line 2>:
   2           RETURN_GENERATOR
               POP_TOP
       L1:     RESUME                   0
               LOAD_FAST                0 (.0)
               GET_ITER
       L2:     FOR_ITER                28 (to L3)
               STORE_FAST               1 (c)
               LOAD_GLOBAL              1 (chr + NULL)
               LOAD_GLOBAL              3 (len + NULL)
               LOAD_FAST                1 (c)
               CALL                     1
               LOAD_CONST               0 (27)
               BINARY_OP               10 (-)
               CALL                     1
               YIELD_VALUE              0
               RESUME                   5
               POP_TOP
               JUMP_BACKWARD           30 (to L2)
       L3:     END_FOR
               POP_TOP
               RETURN_CONST             1 (None)

  --   L4:     CALL_INTRINSIC_1         3 (INTRINSIC_STOPITERATION_ERROR)
               RERAISE                  1
ExceptionTable:
  L1 to L4 -> L4 [0] lasti
```
This again needs us to keep track of the stack, but I'm not going to walk through that, but instead just show what the decompiled code would look like:
```py
(chr(len(c) - 27) for c in '...')
```
Where `'...'` is the underscore string in the original function, so now we have all of the information we need to decompile the original function:
```py
def aaabaa...(s=''):
    return s + ''.join(chr(len(c) - 27) for c in '___...'.split(' '))
```
We can see that all the function does is append a character based on the length of the underscore string. Now looking at the other functions, they all look very similar pretty much only differing, as far as I noticed, in the length of the underscore string, furthermore, all of the genexpr functions are exactly the same, every single one of them, so whenever we see that we don't need to actually read the genexpr function, as we can just assume it's the same. Now the only function we haven't reversed, or at least have a strong suspicion on, is the `zzz___...` function, but before that, we can make a script to get the information we need out of the a/b functions.

## The script
So for this script all we need to do is get a list of the functions and their respective underscore strings. To do this I first separated the code into functions, and its code:
```py
with open('chall.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()

functions = {}
current_key = None
current_block = []

pattern = re.compile(r"Disassembly of <code object (.+) at 0x[0-9a-fA-F]+")

for line in lines:
    match = pattern.match(line)
    if match:
        if 'genexpr' in match.group(1):
            continue
        if current_key and current_block:
            functions[current_key] = current_block
        func_name = match.group(1)
        current_key = func_name
        current_block = []
    elif current_key:
        current_block.append(line.rstrip())

if current_key and current_block:
    functions[current_key] = current_block
```

Now that we have the functions, and its code we can search for the underscore string, which is done with this code:
```py
result = {}

for (name, addr), lines in functions.items():
    if name == "<genexpr>":
        continue
    # skip the "main" function
    if 'z' in name:
        continue

    # only the a/b functions should be analyzed

    underscore_string = None
    for line in lines:
        if underscore_string is None:
            underscore_match = re.search(r"LOAD_CONST\s+\d+\s+\('(_[ _]+)'\)", line)
            if underscore_match:
                underscore_string = underscore_match.group(1)

    if underscore_string:
        result[name] = underscore_string
```

So now we have a mapping between the function name and its underscore string. Now we can reverse the "main" function to put together what the whole program will do.

## The "main" Function
Ok now we need to reverse the main function, called `zzz___...` if we recall. There are 3 distinct parts to this function, idk how to name them, so let's start with the first part. This part is pretty long, and not too repetitive, however the code is not too complex to understand at a surface level. Essentially the code boils down to spamming the `getattr` builtin, and using the generator comprehension on long underscore strings. After some manual reversing, we can decompile into the following python code, replacing the generator comprehensions with the strings they result in:
```py
def zzz___...():
    z_zz... = getattr(__builtins__, 'input')()
    getattr(getattr(__builtins__, '__import__')('base64'), 'b85encode')(getattr(getattr(__builtins__, '__import__')('zlib'), 'compress')(getattr(z_zz..., 'encode')))
```
Now we have the next part, this one is pretty comical, as we just have a huge string of functions being called on the result of the previous looking something like this: `aaba...(babba...(...('c')))` and since we have the information to get the actual string we can do that using the script:
```py
result = {f: ''.join(chr(len(w) - 27) for w in m.split(' ')) for f, m in result.items() if 'z' not in f}
string = 'c'
with open('functions.txt') as f:
    functions = [l.strip() for l in f.readlines()]
for function in functions[::-1]:
    string += result[function]
```

Where `functions.txt` contains the list of functions as from top to bottom in the bottom. This is then reversed because the bytecode calls runs in a first-in-first-out order, so we need to reverse the list so we add the characters from bottom to top as listed in the bytecode.

Running this nets us the string:
```
c$^)LOAf;z3`8&0eUCCCjT0=09bprKs@}bX)kyPs<8)hS-?MvE!3&LZhR{U?bh9~@>fjCRSb2Tq;5|CBYC`5j@W_TE^o3?u6e@&I5DaJ20;`4QxQD^6Ho_QuKYgap{EbWlOh}^bc{Ba}0r&Mj*8
```

The last part is very simple, all it does is compare the result of the `getattr` spam to the string we got in the first part, and if they are equal, that is the correct flag. And so we can deduce that an obfuscated version of the original code could be:
```py
import base64
import zlib

def main():
    guess = input()
    if base64.b85encode(zlib.compress(guess.encode())) == "c$^)LOAf;z3`8&0eUCCCjT0=09bprKs@}bX)kyPs<8)hS-?MvE!3&LZhR{U?bh9~@>fjCRSb2Tq;5|CBYC`5j@W_TE^o3?u6e@&I5DaJ20;`4QxQD^6Ho_QuKYgap{EbWlOh}^bc{Ba}0r&Mj*8".encode():
        print('correct flag!')
```

To finally get the flag, we can decode the base 85 and then decompress it in the script:
```py
compressed = base64.b85decode(string.encode())
print(zlib.decompress(compressed))
```

This finally gets us the flag: `texsaw{python_4_will_never_exist_but_if_it_did_it_might_look_like_this_maybe_but_no_one_can_be_for_sure_did_yall_use_chatgpt_for_this?_let_me_know_if_so}` a long flag, and no, I'm proud to say, I did not use chatGPT for this challenge
