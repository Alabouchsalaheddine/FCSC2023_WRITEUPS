# Original challenge 🇫🇷
L'ENISA a décidé de mettre en place un nouveau service en ligne disponible à l'année pour les équipes qui participent à l'ECSC. Ce service permet aux joueurs des différentes équipes nationales de se créer des comptes individuels en utilisant des tokens secrets par pays. Une fois le compte créé, les joueurs peuvent voir les flags capturés dans différents CTF.

Les données personnelles des utilisateurs (mot de passe et pays) sont protégées dans la base de données, seuls les noms d'utilisateurs sont stockés en clair.

Le token pour la Team France est ohnah7bairahPh5oon7naqu1caib8euh.

Pour cette première épreuve, on vous met au défi d'aller voler un flag FCSC{...} à l'équipe suisse :-)

https://enisa-flag-store.france-cybersecurity-challenge.fr/

Notes :

Les flags au format FAKE{...} que vous pourrez trouver ne sont pas à soumettre.
Les comptes utilisateurs sont réinitialisés toutes les heures.

# Translated challenge 🇺🇸

ENISA has decided to set up a new online service available all year round for teams participating in the ECSC. This service allows players from different national teams to create individual accounts using secret tokens per country. Once the account is created, players can see flags captured in different CTFs.

Personal user data (password and country) are protected in the database, only user names are stored in clear text.

The token for Team France is ohnah7bairahPh5oon7naqu1caib8euh.

For this first event, we challenge you to steal an FCSC{...} flag from the Swiss team :-)

https://enisa-flag-store.france-cybersecurity-challenge.fr/

Notes:

Flags in FAKE{...} format that you can find are not to be submitted.
User accounts are reset every hour.

# Solution

The website starts with a basic authentication portal. With access to a Sign Up link.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/4f66e3ec-9d2c-45b6-a4b0-8515d809d2cf)
![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/05e87c06-b024-40c3-b263-a8cdc637164c)

Any combinition of Country Token and Country different than France isn't accepted and an error is generated. Using the provided token with France as country give us access to this portal once logged in.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/7fec1ba5-41a9-4b33-ba6b-d32436fefd0b)
![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/789bc801-68da-4eb8-8174-d2228f142cb3)

None of those lines contains the researched flag.
We must find a way to connect as a french competitor and at the same time find all flags.

The provided server script is the following:

```go
package main

import (
    "os"
    "log"
    "bytes"
    "errors"
    "net/http"
    "strings"
    "html/template"
    "database/sql"
    "fmt"
    "time"
    _ "github.com/lib/pq"
    "github.com/gorilla/sessions"

    "io"
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/hex"
)

var DB_HOST = os.Getenv("POSTGRES_HOST")
var DB_USER = os.Getenv("POSTGRES_USER")
var DB_PWD  = os.Getenv("POSTGRES_PASSWORD")
var DB_NAME = os.Getenv("POSTGRES_DB")
var DB_PORT = 5432

var ENCRYPTION_KEY = []byte(os.Getenv("ENCRYPTION_KEY"))

var db *sql.DB

/*
CREATE TABLE public.users (
    id SERIAL NOT NULL,
    username VARCHAR(192) NOT NULL,
    password VARCHAR(192) NOT NULL,
    country VARCHAR(192) NOT NULL,
    ts_created   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
*/
type User struct {
    Id                    int `json:"id"`
    Username           string `json:"username"`
    Password           string `json:"password"`
    Country            string `json:"country"`
    Created         time.Time `json:"ts_created"`
}

/*
CREATE TABLE public.flags (
    id SERIAL NOT NULL,
    country VARCHAR(192) NOT NULL,
    ctf VARCHAR(192) NOT NULL,
    challenge VARCHAR(192) NOT NULL,
    category VARCHAR(192) NOT NULL,
    flag VARCHAR(192) NOT NULL,
    points INTEGER NOT NULL
);
*/
type Flag struct {
    Id                    int `json:"id"`
    Country            string `json:"country"`
    CTF                string `json:"ctf"`
    Challenge          string `json:"challenge"`
    Category           string `json:"category"`
    Flag               string `json:"flag"`
    Points                int `json:"points"`
}

/*
CREATE TABLE public.country_tokens (
    id SERIAL NOT NULL,
    country VARCHAR(192) NOT NULL,
    token VARCHAR(192) NOT NULL
);
*/
type Token struct {
    Id                    int `json:"id"`
    Country            string `json:"country"`
    Token              string `json:"token"`
}

var (
    key = []byte(os.Getenv("SESSION_KEY"))
    store = sessions.NewCookieStore(key)
)

var (
    ErrUserAlreadyExist = errors.New("User already exist")
    ErrFieldTooLong     = errors.New("Text fields are limited to 192 characters.")
)

func main() {

    // Connect to the postgresql 14 database
    psqlInfo := fmt.Sprintf(`host=%s port=%d user=%s password=%s
                             dbname=%s sslmode=disable`,
                    DB_HOST, DB_PORT, DB_USER, DB_PWD, DB_NAME)

    var err error
    db, err = sql.Open("postgres", psqlInfo)
    if err != nil {
        panic(err)
    }
    defer db.Close()

    err = db.Ping()
    if err != nil {
        panic(err)
    }

    fmt.Println("Successfully connected to the database!")

    store.Options.HttpOnly = true

    mux := http.NewServeMux()

    fileServer := http.FileServer(http.Dir("./static"))
    mux.Handle("/static/", http.StripPrefix("/static", neuter(fileServer)))
    mux.HandleFunc("/", endpoint_index)
    mux.HandleFunc("/login", endpoint_login)
    mux.HandleFunc("/signup", endpoint_signup)
    mux.HandleFunc("/logout", endpoint_logout)
    mux.HandleFunc("/flags", endpoint_flags)

    err = http.ListenAndServe(":8000", mux)
    log.Fatal(err)
}

func neuter(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if strings.HasSuffix(r.URL.Path, "/") {
            http.NotFound(w, r)
            return
        }

        next.ServeHTTP(w, r)
    })
}

/*******************************************************************
 * ENDPOINTS
 ******************************************************************/

func endpoint_index(w http.ResponseWriter, r *http.Request) {

    if r.URL.Path != "/" {
        errorHandler(w, r, http.StatusNotFound)
        return
    }

    if checkAuth(w, r) {
        var user User
        session, _ := store.Get(r, "auth")
        user.Username = session.Values["username"].(string)
        user.Country = session.Values["country"].(string)
        show_profile(w, r, user)

    } else {
        error := r.URL.Query().Get("error")
        show_index(w, r, error)
    }
}

func endpoint_login(w http.ResponseWriter, r *http.Request) {

    if checkAuth(w, r) {
        http.Redirect(w, r, "/", http.StatusSeeOther)
    }

    if err := r.ParseForm(); err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    username := r.FormValue("username")
    password := r.FormValue("password")

    u, err := CheckLoginPassword(username, password)
    if err != nil {
        if err == sql.ErrNoRows {
            http.Redirect(w, r, "/?error=1", http.StatusSeeOther)
        } else {
            http.Redirect(w, r, "/?error=2", http.StatusSeeOther)
        }
        return
    }

    session, _ := store.Get(r, "auth")
    session.Values["authenticated"] = true
    session.Values["username"] = u.Username
    session.Values["country"] = u.Country
    session.Save(r, w)

    http.Redirect(w, r, "/", http.StatusSeeOther)
}

func endpoint_flags(w http.ResponseWriter, r *http.Request) {

    if checkAuth(w, r) {
        var user User
        session, _ := store.Get(r, "auth")
        user.Username = session.Values["username"].(string)
        user.Country   = session.Values["country"].(string)
        show_flags(w, r, user)

    } else {
        error := r.URL.Query().Get("error")
        show_index(w, r, error)
    }
}

func endpoint_signup(w http.ResponseWriter, r *http.Request) {

    if checkAuth(w, r) {
        http.Redirect(w, r, "/", http.StatusSeeOther)
    }

    if r.Method == "GET" {
        error := r.URL.Query().Get("error")
        show_signup(w, r, error)

    } else if r.Method == "POST" {

        if err := r.ParseForm(); err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        username := r.FormValue("username")
        password := r.FormValue("password")
        token    := r.FormValue("token")
        country  := r.FormValue("country")

        if country == "" {
            http.Redirect(w, r, "/signup?error=1", http.StatusSeeOther)
            return
        }

        if CheckToken(country, token) == false {
            http.Redirect(w, r, "/signup?error=2", http.StatusSeeOther)
            return
        }

        err := RegisterLoginPassword(username, password, country)
        if err != nil {
            if errors.Is(err, ErrUserAlreadyExist) {
                http.Redirect(w, r, "/signup?error=3", http.StatusSeeOther)
            } else if errors.Is(err, ErrFieldTooLong) {
                http.Redirect(w, r, "/signup?error=4", http.StatusSeeOther)
            } else {
                http.Redirect(w, r, "/signup?error=5", http.StatusSeeOther)
            }
            return
        }


        http.Redirect(w, r, "/", http.StatusSeeOther)
    }
}

func endpoint_logout(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "auth")

    session.Values["authenticated"] = false
    session.Values["username"] = ""
    session.Values["country"] = ""
    session.Save(r, w)

    http.Redirect(w, r, "/", http.StatusSeeOther)
}

/*******************************************************************
 * TEMPLATE RENDERING
 ******************************************************************/

func errorHandler(w http.ResponseWriter, r *http.Request, status int) {
    w.WriteHeader(status)
    if status == http.StatusNotFound {

        tmpl, err := template.New("").ParseFiles("views/base.html",
                                                 "views/error.html")
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }

        err = tmpl.ExecuteTemplate(w, "base", nil)
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
    }
}

func show_index(w http.ResponseWriter, r *http.Request, errno string) {
    tmpl, err := template.New("").ParseFiles("views/base.html",
                                             "views/login.html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    message := map[string]interface{}{}
    if errno == "1" {
        message["Message"] = "Inexisting username or invalid password"
    } else if errno == "2" {
        message["Message"] = "Text fields are limited to 192 characters."
    }

    var buf bytes.Buffer

    err = tmpl.ExecuteTemplate(&buf, "base", message)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/html; charset=UTF-8")
    buf.WriteTo(w)
}

func show_flags(w http.ResponseWriter, r *http.Request, user User) {

    tmpl, err := template.New("").ParseFiles("views/base.html",
                                              "views/flags.html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    data, err := getData(user)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    content := map[string]interface{}{}
    content["User"] = user
    content["Data"] = data

    var buf bytes.Buffer
    err = tmpl.ExecuteTemplate(&buf, "base", content)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/html; charset=UTF-8")
    buf.WriteTo(w)
}

func show_signup(w http.ResponseWriter, r *http.Request, errno string) {
    tmpl, err := template.New("").ParseFiles("views/base.html",
                                             "views/signup.html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    message := map[string]interface{}{}
    if errno == "1" {
        message["Message"] = "You must choose a country."
    } else if errno == "2" {
        message["Message"] = "Your token is invalid for the selected country."
    } else if errno == "3" {
        message["Message"] = "This username is already in use."
    } else if errno == "4" {
        message["Message"] = "Text field are limited to 192 characters."
    } else if errno == "5" {
        message["Message"] = "Undefined error."
    }

    var buf bytes.Buffer
    err = tmpl.ExecuteTemplate(&buf, "base", message)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/html; charset=UTF-8")
    buf.WriteTo(w)
}

func show_profile(w http.ResponseWriter, r *http.Request, user User) {

    tmpl, err := template.New("").ParseFiles("views/base.html",
                                             "views/profile.html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    var buf bytes.Buffer

    content := map[string]interface{}{}
    content["User"] = user

    err = tmpl.ExecuteTemplate(&buf, "base", content)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/html; charset=UTF-8")
    buf.WriteTo(w)
}

/*******************************************************************
 * HELPER FUNCTIONS
 ******************************************************************/

func checkAuth(w http.ResponseWriter, r *http.Request) (
            bool,
) {
    session, _ := store.Get(r, "auth")
    if auth, ok := session.Values["authenticated"].(bool); !ok || !auth {
        session.Values["authenticated"] = false
        session.Save(r, w)
        return false
    }
    return true
}

func getData(user User) (
    []Flag,
    error,
) {
    var flags []Flag

    req := fmt.Sprintf(`SELECT ctf, challenge, flag, points
                        FROM flags WHERE country = '%s';`, user.Country);  // SQL injection here to select %s as ---> fr' or 1=1 --
    rows, err := db.Query(req);
    if err != nil {
        return flags, err
    }
    defer rows.Close()

    for rows.Next() {
        var flag Flag
        err = rows.Scan(&flag.CTF, &flag.Challenge, &flag.Flag, &flag.Points)
        if err != nil {
            return flags, err
        }
        flags = append(flags, flag)
    }
    if err = rows.Err(); err != nil {
        return flags, err
    }

    return flags, nil
}

func CheckLoginPassword(username string, password string) (
    *User,
    error,
) {

    stmt, err := db.Prepare(`SELECT id, username, country FROM users
                             WHERE username = $1
                             AND password = encode(digest($2, 'sha1'), 'hex')`)
    if err != nil {
        log.Fatal(err)
    }

    u := &User{}
    err = stmt.QueryRow(username, password).Scan(&u.Id, &u.Username, &u.Country)

    valid, country_dec := decrypt(u.Country)
    if !valid {
        return nil, err
    }

    u.Country = country_dec

    if err != nil {
        return nil, err
    }
    return u, nil
}


func RegisterLoginPassword(username string, password string, country string) (
    error,
) {

    /* First check that user does not alreayd exist */
    stmt, err := db.Prepare(`SELECT username FROM users WHERE username = $1`)
    if err != nil {
        return err
    }

    var u string
    err = stmt.QueryRow(username).Scan(&u)
    if u == username {
        return ErrUserAlreadyExist
    }

    /* If not, then insert a new user */
    country_enc := encrypt(country)
    stmt, err = db.Prepare(`INSERT INTO users (username, password, country)
                            VALUES (
                                $1,
                                encode(digest($2, 'sha1'), 'hex'),
                                $3
                            )`)
    if err != nil {
        log.Fatal(err)
    }

    _, err = stmt.Exec(username, password, country_enc)
    if err != nil {
        return ErrFieldTooLong
    }
    return nil
}

func CheckToken(country string, token string) (
    bool,
) {

    stmt, err := db.Prepare(`SELECT id FROM country_tokens
                             WHERE country = SUBSTR($1, 1, 2) // Checking only the 2 first chars of the country allows us to add chars after the fr value of the select dropdown 
                             AND token = encode(digest($2, 'sha1'), 'hex')`)
    if err != nil {
        log.Fatal(err)
    }

    t := &Token{}
    err = stmt.QueryRow(country, token).Scan(&t.Id)
    if err != nil {
        return false
    }
    return true
}

func encrypt(plaintext string) (
    string,
) {

    c, err := aes.NewCipher(ENCRYPTION_KEY)
    if err != nil {
        fmt.Println(err)
    }

    gcm, err := cipher.NewGCM(c)
    if err != nil {
        fmt.Println(err)
    }

    nonce := make([]byte, gcm.NonceSize())
    if _, err = io.ReadFull(rand.Reader, nonce); err != nil {
        fmt.Println(err)
    }

    ciphertext := gcm.Seal(nonce, nonce, []byte(plaintext), nil)
    return hex.EncodeToString(ciphertext)
}

func decrypt(ciphertext_hex string) (
    bool,
    string,
) {

    ciphertext, err := hex.DecodeString(ciphertext_hex)
    if err != nil {
        fmt.Println(err)
        return false, ""
    }

    c, err := aes.NewCipher(ENCRYPTION_KEY)
    if err != nil {
        fmt.Println(err)
        return false, ""
    }

    gcm, err := cipher.NewGCM(c)
    if err != nil {
        fmt.Println(err)
        return false, ""
    }

    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        fmt.Println(err)
        return false, ""
    }

    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        fmt.Println(err)
        return false, ""
    }
    return true, string(plaintext)
}

```

As described in the comments added here (could be searched in this page):

- Checking only the 2 first chars of the country allows us to add chars after the fr value of the select dropdown 
- SQL injection here to be done

The value of country should be **fr' or 1=1 --** to inject sql code to see all countries flags.

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/1791b953-8a03-4eb2-be5d-1405bc368528)

Once logged in, listing the flags will show you first the target flag 😉

![image](https://github.com/Alabouchsalaheddine/FCSC2023_WRITEUPS/assets/13770404/7bea3b34-039f-49d2-a2e5-74d61bfa0609)

