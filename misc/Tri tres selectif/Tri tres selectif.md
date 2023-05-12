# Tri sélectif

# Original challenge 🇫🇷
Comme l'épreuve Tri Sélectif, vous devez trier le tableau, mais cette fois vous devez être efficace !

# Translated challenge 🇺🇸
Like the Selective Sorting test, you have to sort the board, but this time you have to be efficient!

```python
#!/usr/bin/env python3

# python3 -m pip install pwntools
from pwn import *

# Paramètres de connexion
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2052

def comparer(x, y):
	c = f"comparer {x} {y}".encode()
	io.sendlineafter(b">>> ", c)
	return int(io.recvline().strip().decode())

def echanger(x, y):
	io.sendlineafter(b">>> ", f"echanger {x} {y}".encode())

def longueur():
	io.sendlineafter(b">>> ", b"longueur")
	return int(io.recvline().strip().decode())

def verifier():
	io.sendlineafter(b">>> ", b"verifier")
	r = io.recvline().strip().decode()
	if "flag" in r:
		print(r)
	else:
		print(io.recvline().strip().decode())
		print(io.recvline().strip().decode())

def trier(N):
	#############################
	#   ... Complétez ici ...   #
	# Ajoutez votre code Python #

	#############################
	pass

# Ouvre la connexion au serveur
io = remote(HOST, PORT)

# Récupère la longueur du tableau
N = longueur()

# Appel de la fonction de tri que vous devez écrire
trier(N)

# Verification
verifier()

# Fermeture de la connexion
io.close()
```

# Solution
```python

#!/usr/bin/env python3

# python3 -m pip install pwntools
from pwn import *

# Paramètres de connexion
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2052

def comparer(x, y):
	c = f"comparer {x} {y}".encode()
	io.sendlineafter(b">>> ", c)
	return int(io.recvline().strip().decode())

def echanger(x, y):
	io.sendlineafter(b">>> ", f"echanger {x} {y}".encode())

def longueur():
	io.sendlineafter(b">>> ", b"longueur")
	return int(io.recvline().strip().decode())

def verifier():
	io.sendlineafter(b">>> ", b"verifier")
	r = io.recvline().strip().decode()
	if "flag" in r:
		print(r)
	else:
		print(io.recvline().strip().decode())
		print(io.recvline().strip().decode())
def quicksort(low, high):
    if low < high:
        pivot_index = partition(low, high)
        quicksort(low, pivot_index - 1)
        quicksort(pivot_index + 1, high)

def partition(low, high):
    pivot_index = low
    i = low + 1
    j = high

    while True:
        while i <= j and comparer(i, pivot_index):
            i += 1
        while j >= i and comparer(pivot_index, j):
            j -= 1
        if i >= j:
            break
        echanger(i, j)

    echanger(pivot_index, j)
    return j

def trier(N):
    quicksort(0,N-1)

# Ouvre la connexion au serveur
io = remote(HOST, PORT)

# Récupère la longueur du tableau
N = longueur()
print("N : ",N)

# Appel de la fonction de tri que vous devez écrire
trier(N)

# Verification
verifier()

# Fermeture de la connexion
io.close()

```
# Result
```
[!] Pwntools does not support 32-bit Python.  Use a 64-bit release.
[x] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2052
[x] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2052: Trying 51.254.115.216
[+] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2052: Done
N :  32
Le flag est : FCSC{6d275607ccfba86daddaa2df6115af5f5623f1f8f2dbb62606e543fc3244e33a}
[*] Closed connection to challenges.france-cybersecurity-challenge.fr port 2052
```
