# Original challenge 🇫🇷

Vous avez très envie de rejoindre l'organisation du FCSC 2024 et vos skills d'OSINT vous ont permis de trouver ce site en construction. Prouvez à l'équipe organisatrice que vous êtes un crack en trouvant un flag !

https://tes-lent.france-cybersecurity-challenge.fr/

# Translated challenge 🇺🇸

You really want to join the FCSC 2024 organization and your OSINT skills allowed you to find this site under construction. Prove to the organizing team that you are a crack by finding a flag!

https://tes-lent.france-cybersecurity-challenge.fr/

# Solution

The website is showing some internship offers that the companiny is offering. Nothing really special when you see the website.

Clicking into the only offer available redirect us to another page.

It's a web challenge, let's open the console and investigate.
```html
<!--
      <div class="col-md-6">
        <div class="h-100 p-5 text-white bg-dark rounded-3">
          <h2>Générateur de noms de challenges</h2>
          <p>Vous serez en charge de trouver le noms de toutes les épreuves du FCSC 2024.</p>
          <a class="btn btn-outline-light" href="/stage-generateur-de-nom-de-challenges.html">Plus d'infos</a>
        </div>
      </div>
```
We can see a comment on the index page, decommenting it add a new internship offer _Générateur de noms de challenges_. Opening this offer leads us into another page.

Trying to apply leads you into a todo page. It seems like the website is still WIP. 

Still nothing to see here. BUT, going back to the hidden offer html code shows us another comment:

```html
 Do not forget to add this announcement on the secret administration interface: /admin-zithothiu8Yeng8iumeil5oFaeReezae.html
```

Visiting this page shows us the flag :wink:


