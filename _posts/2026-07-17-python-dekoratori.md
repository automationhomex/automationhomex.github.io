---
layout: post
title: "Python dekoratori — magija koja čini kod čišćim i moćnijim"
date: 2026-07-17 21:00:00 +0200
categories: python automatizacija
---

Zamislite da možete da dodate novu funkcionalnost funkciji, a da ne menjate nijedan red njenog koda. Zvuči kao magija? U Pythonu to se zove **dekorator**.

Dekoratori su jedan od onih koncepata koji na početku deluju misteriozno, ali kada ih jednom savladate — postaju neizostavni alat. Pogotovo u **automatizaciji**, gde vam treba ponovljiv, čist i efikasan kod.

## Šta je zapravo dekorator?

Dekorator je **funkcija koja uzima drugu funkciju, dodaje joj neku funkcionalnost i vraća je nazad**. U Pythonu se koristi sa `@` sintaksom:

```python
@moj_dekorator
def moja_funkcija():
    pass
```

Ovo je samo skraćenica za:

```python
def moja_funkcija():
    pass
moja_funkcija = moj_dekorator(moja_funkcija)
```

## Prvi dekorator — merenje vremena

Krenimo od korisnog primera. Zamišljamo da u automatizaciji često treba da znamo koliko traje izvršavanje nekog zadatka:

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        pocetak = time.time()
        rezultat = func(*args, **kwargs)
        kraj = time.time()
        print(f"{func.__name__} je izvršena za {kraj - pocetak:.2f} sekundi")
        return rezultat
    return wrapper

@timer
def obradi_podatke():
    # simulacija obrade
    time.sleep(2)
    return "Obrada završena"

print(obradi_podatke())
# Izlaz:
# obradi_podatke je izvršena za 2.00 sekundi
# Obrada završena
```

## Logger dekorator — znaj šta se dešava

U automatizaciji je ključno imati uvid u to šta se dešava u pozadini:

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"[LOG] Pokrećem {func.__name__} sa argumentima: {args} {kwargs}")
        try:
            rezultat = func(*args, **kwargs)
            print(f"[LOG] {func.__name__} uspešno završena: {rezultat}")
            return rezultat
        except Exception as e:
            print(f"[LOG] {func.__name__} je pala sa greškom: {e}")
            raise
    return wrapper

@logger
def download_fajl(url):
    if "error" in url:
        raise ValueError("URL sadrži grešku")
    return f"Preuzeto sa {url}"

download_fajl("https://primer.com/fajl.csv")   # uspešno
# download_fajl("https://primer.com/error.csv") # greška
```

## Retry dekorator — nikad ne odustaj

Ovo je jedan od najkorisnijih dekoratora u automatizaciji. Mrežni pozivi, API pozivi i čitanje fajlova često padnu — treba nam automatsko ponavljanje:

```python
import time
import random

def retry(max_attempts=3, delay=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for pokusaj in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if pokusaj == max_attempts:
                        raise
                    print(f"Pokušaj {pokusaj} nije uspeo: {e}. Ponavljam za {delay}s...")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def pozovi_api():
    if random.random() < 0.7:  # 70% šansa da padne
        raise ConnectionError("Mrežna greška")
    return "API odgovor uspešan"

print(pozovi_api())
```

> **Primetite**: `@retry(max_attempts=5, delay=0.5)` je **dekorator sa argumentima** — zapravo je funkcija koja vraća dekorator. Moćno, zar ne?

## Kombinovanje više dekoratora

Dekoratori se mogu slagati jedan na drugi — kao Lego kockice:

```python
@logger
@timer
def slozeni_zadatak():
    time.sleep(1)
    return "Gotovo"

# Redosled: prvo se primenjuje @timer, zatim @logger
# => slozeni_zadatak = logger(timer(slozeni_zadatak))
```

## Pametni dekorator — čuvanje metapodataka

Kada koristimo dekorator, originalno ime i dokumentacija funkcije se gube. Zato postoji `functools.wraps`:

```python
from functools import wraps

def pametni_timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """Ovo je wrapper funkcija"""
        # ... isto kao gore
        pass
    return wrapper

@pametni_timer
def pozdrav():
    """Pozdravlja svet"""
    print("Zdravo svete!")

print(pozdrav.__name__)   # "pozdrav" (a ne "wrapper")
print(pozdrav.__doc__)    # "Pozdravlja svet" (a ne "Ovo je wrapper funkcija")
```

## Zašto su dekoratori bitni za automatizaciju?

Evo nekoliko primera gde dekoratori sijaju u automatizaciji:

1. **Rate limiting** — ograničavanje broja API poziva u minuti
2. **Caching** — keširanje rezultata skupih funkcija
3. **Validacija ulaza** — automatska provera argumenata
4. **Autorizacija** — provera da li korisnik ima dozvolu
5. **Transakcije** — automatsko otvaranje/zatvaranje baze
6. **Notifikacije** — slanje mejla ili poruke kada zadatak padne

## Mini projekat: dekorator za notifikacije

```python
def notifikuj_na_gresku(channel="email"):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                poruka = f"❌ Zadatak {func.__name__} je pao: {e}"
                if channel == "email":
                    print(f"[EMAIL] Šaljem notifikaciju: {poruka}")
                elif channel == "slack":
                    print(f"[SLACK] Šaljem poruku: {poruka}")
                elif channel == "sms":
                    print(f"[SMS] Šaljem SMS: {poruka}")
                raise
        return wrapper
    return decorator

@notifikuj_na_gresku(channel="slack")
def obradi_fajl(putanja):
    with open(putanja) as f:
        return f.read()

obradi_fajl("nepostojeci_fajl.txt")
# [SLACK] Šaljem poruku: ❌ Zadatak obradi_fajl je pao: [Errno 2] No such file...
```

## Zaključak

Dekoratori su kao **funkcijske nadogradnje** — pišete ih jednom, a koristite svuda. U automatizaciji, gde često imamo ponavljajuće obrasce (logovanje, merenje, ponavljanje, notifikacije), dekoratori su savršeno rešenje.

Kad sledeći put napišete isti `try/except` blok treći put — setite se dekoratora. 🐍

---

*Imaš omiljeni dekorator koji koristiš u svom kodu? Piši u komentar!*