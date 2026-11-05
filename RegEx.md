# L1: letters and digits
- everything is essentially a character
- most patterns use normal ASCII, which includes letters, digits, punctuation and other symbols on the keyboard like %#$@!
- the character `\d` can be used in any place of **any digit from 0 to 9**
# L2: the dot
- there is the concept of a **wildcard**, which is represented by the `.` metacharacter and can **match any single character** (even the whitespaces)
- ex: `...\.` -> 3 characters and a final dot
# L3: matching specific characters
- we can use the square brackets to select all the words containing at least one of the characters inside the brackets in that specific position
- ex: `[abc][abc]` -> selects all the words where the first 2 characters are a, b or c
# L4: excluding specific characters
- same before but we use `^` to exclude the characters in the squared brackets
- ex: `[^abc]` -> selects all the words where the first character is different from a, b and c
# L5: character ranges
- matching a character in list of **sequential characters** by using the **dash** to indicate a character range
- `\w`: equivalent to any alphanumeric character (alias `[A-Za-z0-9_]`)
- `\W`: equivalent to any non-alphanumeric character
- ex: `[A-Z]` -> select all the words where the first character is a uppercase letter
# L6: catching some zzz's
- how do we have to do for selecting the consecutive repetition of some character of a specific number of times?
- `\d\d\d`: 3 repetitions of the same digit
- ex: `[wxy]{5}` -> 5 characters, each of which can be a w, x or y
- ex: `.{1,3}` -> range from 1 to 3 with the repetition of any character
# L7: Mr. Kleene
- Kleene Star `*` (0 or more repetitions of the character/group that it follows) and Kleene Plus `+` (1 or more repetitions of the character/group that it follows)
- ex: `[abc]+` -> 1 or more of any a, b, or character
- ex: `.*` -> 0 or more of any character
# L8: characters optional
- `?` denotes **optionality**, it allows to match either 0 or 1 of the preceding character/group
- ex: `ab?c` -> it matches *abc* or *ac*
# L9: all this whitespace
- most common forms of whitespace are: `_` space, `\t` tab, `\n` new line, `\r` carriage return and `\s` any of the previous whitespaces
# L10: starting and ending
- there are even metacharacters to match a line starting or ending with a specific word
	- round parentheses are necessary only for terms with more the one word
- **starting**: ^(prime parole)
- **ending**: (ultime parole)$
# L11: match groups
- we can extract information for further processing by defining groups of characters
- ex: `^(IMG\d+)\.png$` -> matches all "files" of the following format: *IMG + 1 or more digits + .png*
# L12-13: captures sub-groups and multiple groups
- same concept as before but with nested and sequential groups
- ex. `^(IMG(\d+))\.png$`
- ex. `(\d{4})x(\d{3}(\d)?)`
# L14: it's all conditional
- when we use groups, we can use the `|` (logical OR, aka. the pipe) to denote different possible sets of characters
---
# Other special characters
- `\w`: alphanumeric character 
- `\W`: non-alphanumeric character
- `\d`: digit character
- `\D` non-digit character
- `\s`: whitespace character
- `\S`: non-whitespace character
- `\b`: boundary between a word and a non-word character