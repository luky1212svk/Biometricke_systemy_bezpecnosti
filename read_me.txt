===============================================================================
  POST-KVANTOVÁ KRYPTOGRAFIA V BIOMETRICKÝCH AUTENTIFIKAČNÝCH SYSTÉMOCH
  Semestrálne zadanie – Biometrické systémy bezpečnosti (KEMT FEI TUKE)
  Autori: Bc. Lukáš Zachar, Bc. Leonard Silvester Ujlaki
===============================================================================


1. ČO PROGRAM ROBÍ
-------------------------------------------------------------------------------
Program simuluje vzdialenú biometrickú autentifikáciu (prihlásenie tvárou)
medzi KLIENTOM (zariadenie so snímačom) a SERVEROM (autentifikačná služba
s databázou šablón). Komunikácia prebieha cez sieť (TCP).

Každé prihlásenie je možné vykonať v dvoch režimoch:

  Režim       Výmena kľúča   Digitálny podpis   Šifrovanie šablóny
  ---------   ------------   ----------------   ------------------
  klasicky    X25519         Ed25519            AES-256-GCM
  pq          ML-KEM-768     ML-DSA-65          AES-256-GCM

  klasicky = súčasné algoritmy (prelomiteľné kvantovým počítačom)
  pq       = post-kvantové algoritmy NIST (FIPS 203, FIPS 204)

Program meria veľkosť prenesených dát, počet TCP paketov, čas
kryptografických operácií a overuje odolnosť voči základným útokom.
Biometrické šablóny (tvár = vektor 128 čísel) sú simulované.


2. POŽIADAVKY
-------------------------------------------------------------------------------
  - Operačný systém: Windows 10/11 (funguje aj Linux a macOS)
  - Python 3.10 alebo novší (testované na Python 3.14)
  - Knižnice (inštalujú sa v kroku 3.3):
      cryptography >= 50   (obsahuje ML-KEM a ML-DSA)
      matplotlib   >= 3.8  (kreslenie grafov)
  - Pripojenie na internet iba počas inštalácie knižníc


3. INŠTALÁCIA (Windows)
-------------------------------------------------------------------------------
3.1 Inštalácia Pythonu (ak ho ešte nemáte)
    1. Stiahnite Python z https://www.python.org/downloads/
    2. Pri inštalácii ZAŠKRTNITE možnosť "Add python.exe to PATH".
    3. Dokončite inštaláciu.

3.2 Otvorenie terminálu v priečinku projektu
    Možnosť A (Visual Studio Code):
      File -> Open Folder -> vyberte priečinok "pq_biometria"
      Terminal -> New Terminal
    Možnosť B (Prieskumník):
      Otvorte priečinok "pq_biometria", kliknite pravým tlačidlom
      do prázdneho miesta -> "Otvoriť v termináli".

    Overte, že Python funguje:
      py --version
    (Vypíše napr. "Python 3.14.0". Ak príkaz "py" nefunguje,
     používajte vo všetkých príkazoch namiesto "py" slovo "python".)

3.3 Inštalácia knižníc
      py -m pip install -r requirements.txt

    Overenie (nesmie vypísať chybu):
      py -c "from cryptography.hazmat.primitives.asymmetric import mlkem, mldsa; print('OK')"


4. SPUSTENIE – AUTOMATICKÉ MERANIE (hlavná časť zadania)
-------------------------------------------------------------------------------
      py meranie.py

Skript sám spustí server na pozadí (port 5051) a pre oba režimy vykoná:
  1. meranie času každej kryptografickej operácie (300 opakovaní),
  2. registráciu 10 osôb,
  3. 200 prihlásení správnych osôb            -> očakávané PRIJATÝ,
  4. 90 pokusov cudzej osoby na cudzom zariadení -> očakávané ZAMIETNUTÝ,
  5. 50 útokov so zmenenou správou            -> očakávané ZAMIETNUTÝ.

Trvanie: približne 10 sekúnd.

Ukážka výstupu v termináli (časy sa líšia podľa počítača):

  === VEĽKOSTI (bajty) ===
                                    klasický   post-kvantový
  verejny kluc vymeny (B)                 32            1184
  zabalene tajomstvo (B)                  32            1088
  verejny podpisovy kluc (B)              32            1952
  podpis (B)                              64            3309
  ...
  === JEDNO PRIHLÁSENIE CEZ SIEŤ ===
  prenesené bajty                      846.1          9538.1
  TCP pakety (MTU 1500)                  4.0            10.0
  Post-kvantový režim prenesie 11.3x viac dát.

  === BEZPEČNOSTNÉ SCENÁRE ===
  klasicky   spravna osoba    správne vyhodnotené 200/200
  klasicky   ina osoba        správne vyhodnotené 90/90
  klasicky   zmenena sprava   správne vyhodnotené 50/50
  pq         spravna osoba    správne vyhodnotené 200/200
  ...

Výsledky sa uložia do priečinka "vysledky":
  prihlasenia.csv          – každé prihlásenie (výsledok, bajty, pakety, čas)
  velkosti.csv             – veľkosti kľúčov a podpisov
  cas_operacii_ms.csv      – priemerné časy operácií
  suhrn.csv                – súhrn za jedno prihlásenie
  graf_prenesene_bajty.png – graf prenesených dát
  graf_cas_operacii.png    – graf času operácií
  graf_rychlost_siete.png  – čas prenosu podľa rýchlosti siete
(CSV súbory používajú oddeľovač ";" a dajú sa otvoriť v Exceli.)


5. SPUSTENIE – RUČNÉ VYSKÚŠANIE (dva terminály)
-------------------------------------------------------------------------------
Je potrebné mať otvorené DVA terminály v priečinku "pq_biometria".

TERMINÁL 1 – spustenie servera (nechajte ho bežať):
      py server.py

  Výpis: "Server beží na 127.0.0.1:5050 (ukončenie: Ctrl+C)"

TERMINÁL 2 – klient (snímač):

  a) Registrácia osoby 1 pod menom "jano" v post-kvantovom režime:
      py klient.py registracia --meno jano --osoba 1 --rezim pq

  b) Prihlásenie tej istej osoby (očakávané PRIJATY):
      py klient.py prihlasenie --meno jano --osoba 1 --rezim pq

  c) Pokus inej osoby (osoba 2) na Janovom zariadení (očakávané ZAMIETNUTY):
      py klient.py prihlasenie --meno jano --osoba 2 --rezim pq

  d) To isté v klasickom režime:
      py klient.py registracia --meno fero --osoba 3 --rezim klasicky
      py klient.py prihlasenie --meno fero --osoba 3 --rezim klasicky

  Ukážka výstupu:
      PRIJATY  (vzdialenost 0.479)
      Prenesené: 4990 B odoslané + 4541 B prijaté = 9531 B,
      približne 10 TCP paketov, čas 52.8 ms

Parametre klienta:
  registracia | prihlasenie   – operácia
  --meno MENO                 – meno používateľa (povinné)
  --osoba N                   – čia tvár sa sníma (1, 2, 3, ...), predvolene 1
  --rezim klasicky | pq       – kryptografický režim, predvolene pq
  --server IP                 – IP adresa servera, predvolene 127.0.0.1
  --port PORT                 – port servera, predvolene 5050

Poznámky:
  - Prihlásenie funguje len v rovnakom režime, v akom bola osoba
    zaregistrovaná.
  - Opätovná registrácia rovnakého mena prepíše pôvodný záznam.
  - Čas pri ručnom spustení zahŕňa aj nadviazanie spojenia a načítanie
    knižníc, preto je vyšší ako v automatickom meraní (kap. 4).
  - Server sa ukončí v termináli 1 klávesmi Ctrl+C.


6. SPUSTENIE NA DVOCH POČÍTAČOCH V SIETI (voliteľné)
-------------------------------------------------------------------------------
Na oboch počítačoch musí byť projekt a nainštalované knižnice (kap. 3).

Počítač SERVER:
  1. Spustite:  py server.py --adresa 0.0.0.0
  2. Zistite IP adresu príkazom:  ipconfig   (napr. 192.168.1.20)
  3. Vo Windows Firewall povoľte prichádzajúce spojenia na TCP port 5050
     (pri prvom spustení sa Windows zvyčajne opýta sám – zvoľte "Povoliť").

Počítač KLIENT:
  1. Skopírujte zo servera súbory
        data\server_klasicky_verejny.bin
        data\server_pq_verejny.bin
     do priečinka "data" na klientovi (ak neexistuje, vytvorte ho).
     Sú to verejné kľúče servera – klient nimi overuje, že komunikuje
     so skutočným serverom.
  2. Spustite napr.:
        py klient.py registracia --meno jano --osoba 1 --rezim pq --server 192.168.1.20
        py klient.py prihlasenie --meno jano --osoba 1 --rezim pq --server 192.168.1.20

Komunikáciu možno zachytiť nástrojom Wireshark s filtrom:
        tcp.port == 5050


7. ŠTRUKTÚRA PROJEKTU
-------------------------------------------------------------------------------
  krypto.py         výmena kľúča, podpis, HKDF a AES-GCM pre oba režimy
  biometria.py      simulácia šablón tváre, snímania a porovnania
  siet.py           posielanie správ cez TCP, počítanie bajtov a paketov
  server.py         autentifikačný server s databázou šablón
  klient.py         klient (snímač) – registrácia a prihlásenie
  meranie.py        automatické meranie, export CSV a grafov
  requirements.txt  zoznam potrebných knižníc
  README.md         stručný popis projektu
  read_me.txt       tento návod

  data\             vytvorí sa automaticky pri prvom spustení servera:
    server_*.pem            tajné podpisové kľúče servera
    server_*_verejny.bin    verejné kľúče servera (pre klienta)
    kluc_databazy.bin       AES kľúč na šifrovanie databázy
    databaza.json           databáza (šablóny sú uložené zašifrované)
    zariadenie_*.pem        tajné kľúče zaregistrovaných zariadení
  vysledky\         výstupy automatického merania (kap. 4)


8. PRIEBEH PRIHLÁSENIA (protokol)
-------------------------------------------------------------------------------
  1. klient -> server : "PRIHLASENIE", režim, meno
  2. server -> klient : jednorazový verejný kľúč + podpis servera
  3. klient           : overí podpis servera,
                        vytvorí spoločné tajomstvo -> AES kľúč (HKDF),
                        zašifruje šablónu (AES-256-GCM),
                        podpíše správu kľúčom zariadenia
     klient -> server : zabalené tajomstvo, zašifrovaná šablóna, podpis
  4. server           : overí podpis zariadenia, dešifruje šablónu,
                        porovná ju s uloženou (vzdialenosť < 0.8)
     server -> klient : PRIJATY / ZAMIETNUTY


9. RIEŠENIE PROBLÉMOV
-------------------------------------------------------------------------------
Problém: "py" nie je rozpoznaný ako príkaz
  -> Python nie je nainštalovaný alebo nie je v PATH. Použite "python"
     namiesto "py", prípadne preinštalujte Python so zaškrtnutým
     "Add python.exe to PATH" (kap. 3.1).

Problém: ImportError: cannot import name 'mlkem'
  -> Stará verzia knižnice cryptography. Aktualizujte ju:
        py -m pip install --upgrade cryptography

Problém: ModuleNotFoundError: No module named 'matplotlib'
  -> Knižnice nie sú nainštalované:  py -m pip install -r requirements.txt

Problém: ConnectionRefusedError (pri klient.py)
  -> Server nebeží. Spustite v druhom termináli "py server.py".
     Pri dvoch počítačoch skontrolujte IP adresu a firewall.

Problém: OSError: [WinError 10048] / Address already in use
  -> Port je obsadený (server už beží v inom termináli). Ukončite ho
     klávesmi Ctrl+C alebo použite iný port:
        py server.py --port 5060
        py klient.py ... --port 5060

Problém: FileNotFoundError: ...server_pq_verejny.bin
  -> Klient nemá verejné kľúče servera. Najprv aspoň raz spustite
     server (vytvorí ich), pri dvoch počítačoch ich skopírujte (kap. 6).

Problém: "Toto zariadenie nie je zaregistrované"
  -> Pre dané meno a režim ešte neprebehla registrácia. Spustite najprv
     "py klient.py registracia ..." s rovnakým menom a režimom.

Problém: ZAMIETNUTY (neznamy pouzivatel)
  -> Meno nie je v databáze servera alebo bolo zaregistrované v inom
     režime (klasicky/pq).

Problém: "Podpis servera nesedí - možný útok (falošný server)!"
  -> Klient má verejné kľúče z iného servera (napr. po vymazaní priečinka
     data na serveri). Znova skopírujte data\server_*_verejny.bin.

Problém: chyba pri kopírovaní príkazu
  -> Kopírujte iba samotný príkaz, bez textu za ním (napr. bez
     poznámok "očakávané PRIJATY").

Úplný reset projektu:
  Ukončite server a vymažte priečinky "data" a "vysledky".
  Pri ďalšom spustení sa vytvoria nové kľúče a prázdna databáza.

===============================================================================
