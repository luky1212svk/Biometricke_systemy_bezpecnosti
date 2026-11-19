POST-KVANTOVÁ KRYPTOGRAFIA V BIOMETRICKÝCH AUTENTIFIKAČNÝCH SYSTÉMOCH
Semestrálne zadanie – Biometrické systémy bezpečnosti (KEMT FEI TUKE)
Autori: Bc. Lukáš Zachar, Bc. Leonard Silvester Ujlaki


Program simuluje prihlásenie tvárou medzi klientom (snímač) a serverom cez TCP
v dvoch režimoch: "klasicky" (X25519 + Ed25519) a "pq" (ML-KEM-768 + ML-DSA-65).
Šablóna sa v oboch režimoch šifruje AES-256-GCM. Biometria je simulovaná.

1. INŠTALÁCIA
  Potrebný je Python 3.10 alebo novší (testované na 3.14, Windows):
  https://www.python.org/downloads/  (pri inštalácii zaškrtnite "Add python.exe
  to PATH"; návod: https://docs.python.org/3/using/windows.html)

  V priečinku projektu otvorte terminál a spustite:
      py -m pip install -r requirements.txt

  (Ak príkaz "py" nefunguje, použite všade "python".)

2. AUTOMATICKÉ MERANIE (hlavná časť)
      py meranie.py

  Pre oba režimy zmeria časy operácií, vykoná 10 registrácií, 200 prihlásení,
  90 pokusov cudzej osoby a 50 útokov zmenou správy (trvá asi 10 s).
  Výsledky (CSV a grafy) sa uložia do priečinka "vysledky".

3. RUČNÉ VYSKÚŠANIE (dva terminály)
  Terminál 1:  py server.py
  Terminál 2:  py klient.py registracia --meno jano --osoba 1 --rezim pq
               py klient.py prihlasenie --meno jano --osoba 1 --rezim pq   -> PRIJATY
               py klient.py prihlasenie --meno jano --osoba 2 --rezim pq   -> ZAMIETNUTY

  --rezim klasicky | pq     prihlásenie funguje len v režime registrácie
  --server IP, --port N     pre spustenie na dvoch počítačoch (server spustite
                            s --adresa 0.0.0.0 a skopírujte klientovi súbory
                            data\server_*_verejny.bin)

4. ČASTÉ PROBLÉMY
  cannot import name 'mlkem'   -> py -m pip install --upgrade cryptography
  ConnectionRefusedError       -> server nebeží (terminál 1)
  Address already in use       -> server už beží, alebo použite --port 5060
  Úplný reset                  -> vymažte priečinky "data" a "vysledky"
