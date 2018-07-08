Three Letter Combo in String
===

March 7, 2016

Original question from [>>/g/53346630](https://archive.rebeccablacktech.com/g/thread/S53346630).

> Find all 3-letter-strings that can be constructed by combining letter from the input string in sequence. No duplication allowed, unless it occurred in the input.
> 
> \>Example
> ```
> input: richardstallman
> valid combo: rll
> invalid combo: llr # letters don't occur in this sequence in the input
> ```
> 
> Question before you start:
> 
> How many results will you find in the input string "richardstallman"?

I could roughly remember doing something similar in Data Structures, but I couldn't recall how I did it. After some hammering around with Python, and a tip (well, more like answer) from fellow Anons, I more or less got it.

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# input_str = raw_input('Input: ')
input_str = 'richardstallman'
input_len = len(input_str)

output_set = set()

for i in range(input_len - 2):
    for j in range(i + 1, input_len - 1):
        for k in range(j + 1, input_len):
            output_set.add('{i}{j}{k}'.format(
                i=input_str[i], j=input_str[j], k=input_str[k]))

print output_set
print len(output_set)
```

Originally I was doing something like `if combo not in output_list`, but a small Google search revealed that Python had a [set data type](https://docs.python.org/2/library/stdtypes.html#set-types-set-frozenset), which effectively prevents duplicate entries.

My final output was **278** unique combos with the string `richardstallman`, but I am rather unsure if it was right. Subsequent discussions in the thread are still arguing about the right Math algorithm. Nevertheless, good refresher.
