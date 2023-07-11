# Aaarg
# Original challenge 🇫🇷
Vous devez afficher le flag quelque soit le moyen utilisé !
# Translated challenge 🇺🇸
You must display the flag whatever the means used!
# Solution
We are dealing with an .elf file.

By reading the file with IDA and jumping to pseudo code,  we can easily notice the begining of the flag.
![Capture d'écran 2023-05-13 175808](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/blob/main/screenshots/Screenshot_2023-05-13_175808.png)
![Capture d'écran 2023-05-13 175447](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/blob/main/screenshots/Screenshot_2023-05-13_175447.png)


By copying the hexadecimal values and converting them to text with the following script we find the flag.
```python
myList = [
    'F', '\xE2', '\x80', '\x8D', 'C', '\xE2', '\x80', '\x8D', 'S', '\xE2', '\x80', '\x8D', 'C',
    '\xE2', '\x80', '\x8D', '{', '\xE2', '\x80', '\x8D', 'f', '\xE2', '\x80', '\x8D', '9', '\xE2',
    '\x80', '\x8D', 'a', '\xE2', '\x80', '\x8D', '3', '\xE2', '\x80', '\x8D', '8', '\xE2', '\x80',
    '\x8D', 'a', '\xE2', '\x80', '\x8D', 'd', '\xE2', '\x80', '\x8D', 'a', '\xE2', '\x80', '\x8D',
    'c', '\xE2', '\x80', '\x8D', 'e', '\xE2', '\x80', '\x8D', '9', '\xE2', '\x80', '\x8D', 'd',
    '\xE2', '\x80', '\x8D', 'd', '\xE2', '\x80', '\x8D', 'a', '\xE2', '\x80', '\x8D', '3', '\xE2',
    '\x80', '\x8D', 'a', '\xE2', '\x80', '\x8D', '9', '\xE2', '\x80', '\x8D', 'a', '\xE2', '\x80',
    '\x8D', 'e', '\xE2', '\x80', '\x8D', '5', '\xE2', '\x80', '\x8D', '3', '\xE2', '\x80', '\x8D',
    'e', '\xE2', '\x80', '\x8D', '7', '\xE2', '\x80', '\x8D', 'a', '\xE2', '\x80', '\x8D', 'e',
    '\xE2', '\x80', '\x8D', 'c', '\xE2', '\x80', '\x8D', '1', '\xE2', '\x80', '\x8D', '8', '\xE2',
    '\x80', '\x8D', '0', '\xE2', '\x80', '\x8D', 'c', '\xE2', '\x80', '\x8D', '5', '\xE2', '\x80',
    '\x8D', 'a', '\xE2', '\x80', '\x8D', '7', '\xE2', '\x80', '\x8D', '3', '\xE2', '\x80', '\x8D',
    'd', '\xE2', '\x80', '\x8D', 'b', '\xE2', '\x80', '\x8D', 'b', '\xE2', '\x80', '\x8D', '7',
    '\xE2', '\x80', '\x8D', 'c', '\xE2', '\x80', '\x8D', '3', '\xE2', '\x80', '\x8D', '6', '\xE2',
    '\x80', '\x8D', '4', '\xE2', '\x80', '\x8D', 'f', '\xE2', '\x80', '\x8D', 'e', '\xE2', '\x80',
    '\x8D', '1', '\xE2', '\x80', '\x8D', '3', '\xE2', '\x80', '\x8D', '7', '\xE2', '\x80', '\x8D',
    'f', '\xE2', '\x80', '\x8D', 'c', '\xE2', '\x80', '\x8D', '6', '\xE2', '\x80', '\x8D', '7',
    '\xE2', '\x80', '\x8D', '2', '\xE2', '\x80', '\x8D', '1', '\xE2', '\x80', '\x8D', 'd', '\xE2',
    '\x80', '\x8D', '7', '\xE2', '\x80', '\x8D', '9', '\xE2', '\x80', '\x8D', '9', '\xE2', '\x80',
    '\x8D', '7', '\xE2', '\x80', '\x8D', 'c', '\xE2', '\x80', '\x8D', '5', '\xE2', '\x80', '\x8D',
    '4', '\xE2', '\x80', '\x8D', 'e', '\xE2', '\x80', '\x8D', '8', '\xE2', '\x80', '\x8D', 'd',
    '\xE2', '\x80', '\x8D', '}', '\0'
]
flag = ""
for i in range(0,len(myList), 4) :
    flag += myList[i]
print(flag)
```
The flag is : 

FCSC{f9a38adace9dda3a9ae53e7aec180c5a73dbb7c364fe137fc6721d7997c54e8d}
