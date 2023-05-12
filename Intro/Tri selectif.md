# Tri sélectif

# Original challenge 🇫🇷
Vous devez trier un tableau dont vous ne voyez pas les valuers

# Translated challenge 🇺🇸
You need to sort an array whose values you don't see

```python
#!/usr/bin/env python3

# python3 -m pip install pwntools
from pwn import *

# Paramètres de connexion
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2051

def comparer(x, y):
	io.sendlineafter(b">>> ", f"comparer {x} {y}".encode())
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

N = longueur()
trier(N)
# Verification
verifier()

# Fermeture de la connexion
io.close()

```

# Solution
We should complete with one of the known sorting algorithms

```python
#!/usr/bin/env python3

# python3 -m pip install pwntools
from pwn import *

# Paramètres de connexion
HOST, PORT = "challenges.france-cybersecurity-challenge.fr", 2051

def comparer(x, y):
	io.sendlineafter(b">>> ", f"comparer {x} {y}".encode())
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
	for i in range(N):
		min = i
		for j in range(i + 1, N):
			if comparer(min, j) == 0:
				min = j
		echanger(min, i)
  #############################
	pass

# Ouvre la connexion au serveur
io = remote(HOST, PORT)

N = longueur()
trier(N)
# Verification
verifier()

# Fermeture de la connexion
io.close()
```
Then, we can run the Python script
# Result
```
[!] Pwntools does not support 32-bit Python.  Use a 64-bit release.
[x] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2051
[x] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2051: Trying 51.254.115.216
[+] Opening connection to challenges.france-cybersecurity-challenge.fr on port 2051: Done
Le flag est : FCSC{e687c4749f175489512777c26c06f40801f66b8cf9da3d97bfaff4261f121459}
[*] Closed connection to challenges.france-cybersecurity-challenge.fr port 2051
```