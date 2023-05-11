# Original challenge 🇫🇷

Vous avez très envie de rejoindre l'organisation du FCSC 2024 et vos skills d'OSINT vous ont permis de trouver ce site en construction. Prouvez à l'équipe organisatrice que vous êtes un crack en trouvant un flag !

https://tes-lent.france-cybersecurity-challenge.fr/

# Translated challenge 🇺🇸

You really want to join the FCSC 2024 organization and your OSINT skills allowed you to find this site under construction. Prove to the organizing team that you are a crack by finding a flag!

https://tes-lent.france-cybersecurity-challenge.fr/

# Solution

The website is showing some internship offers that the companiny is offering. Nothing really special when you see the website.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/19609597-6052-4c44-8059-d74f9ffad56d)

Clicking onto the only offer available redirect us to another page.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/14a4921f-eb44-4e16-9fe6-8c353f039e2a)

It's a web challenge, let's open the console and investigate.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/576ac581-c633-4393-a036-b2cb73cfde24)

We can see a comment on the index page, decommenting it add a new internship offer _Générateur de noms de challenges_. Opening this offer leads us into another page.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/b6e23b00-f80e-4c66-aa8b-44fff6de5b51)
![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/ef54d5cc-c369-4433-b70a-c7693524112b)

Trying to apply leads you into a todo page. It seems like the website is still WIP. 

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/f5f3e933-fa83-4975-aa7b-7d82298edff7)

Still nothing to see here. BUT, going back to the hidden offer html code shows us another comment:

> Do not forget to add this announcement on the secret administration interface: /admin-zithothiu8Yeng8iumeil5oFaeReezae.html

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/1d66aba7-3883-4bd1-939f-5fd3b388e02a)

Visiting this page shows us the flag :wink:

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/60848f28-1c32-4f31-8286-d89134245104)


