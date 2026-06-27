# employee-management-system
employee-management-system

Via een menu voert het programma verschillende acties uit : loon aanpassen, verlofdagen toekennen, verlof aftrekken, werknemers verwijderen en een overzicht tonen. Er wordt telkens een keuze gemaakt, waarna het programma de gekozen functie uitvoert en de wijzigingen in de database opslaat. Het is opgebouwd uit losse functies die elk één taak uitvoeren. Eerst maakt het programma verbinding met de database. Daarna blijft het menu herhalen tot dat de keuze wordt gemaakt om te stoppen. Bij bijna elke actie controleert de code eerst of de invoer geldig is. Wanneer de gebruiker een getal moet ingeven, gebruikt de code ‘try’ en ‘except ValueError ‘ zodat het programma niet crasht als er tekst wordt ingevoerd in plaats van een getal. Voor werknemers die met een id gezocht worden, betekent ‘fetchone()’ met ‘None’ dat er geen werknemer gevonden is. Dan stopt de functie meteen en wordt er geen wijziging uitgevoerd voor die werknemer.

```python
import mysql.connector


def connect_db():
    return mysql.connector.connect(
        host="name",
        user="user",
        password="password",
        database="name"
    )


def loonsverhoging(conn):
    cursor = conn.cursor()

    werknemer_id = input("Geef de id van de werknemer: ")
    bedrag = input("Welk bedrag moet het loon stijgen? ")

    try:
        bedrag = float(bedrag)

        cursor.execute(
            "SELECT first_name, last_name FROM employees WHERE id = %s",
            (werknemer_id,)
        )
        werknemer = cursor.fetchone()

        if werknemer is None:
            print("Werknemer niet gevonden.")
            return

        cursor.execute(
            "UPDATE employees SET salary = salary + %s WHERE id = %s",
            (bedrag, werknemer_id)
        )
        conn.commit()

        print("Loon succesvol aangepast.")
    except ValueError:
        print("Geef een geldig bedrag in.")


def verlofdagen_toekennen(conn):
    cursor = conn.cursor()

    aantal = input("Hoeveel verlofdagen krijgt elke werknemer? ")

    try:
        aantal = int(aantal)

        cursor.execute(
            "UPDATE employees SET vacation_days = vacation_days + %s",
            (aantal,)
        )
        conn.commit()

        print("Verlofdagen succesvol toegekend.")
    except ValueError:
        print("Geef een geldig geheel getal in.")


def verlofdagen_opnemen(conn):
    cursor = conn.cursor()

    werknemer_id = input("Geef de id van de werknemer: ")
    aantal = input("Hoeveel verlofdagen wil je opnemen? ")

    try:
        aantal = int(aantal)

        cursor.execute(
            "SELECT vacation_days FROM employees WHERE id = %s",
            (werknemer_id,)
        )
        resultaat = cursor.fetchone()

        if resultaat is None:
            print("Werknemer niet gevonden.")
            return

        huidig_saldo = resultaat[0]

        if huidig_saldo < aantal:
            print("Onvoldoende verlofsaldo. Actie geannuleerd.")
            return

        cursor.execute(
            "UPDATE employees SET vacation_days = vacation_days - %s WHERE id = %s",
            (aantal, werknemer_id)
        )
        conn.commit()

        print("Verlofdagen succesvol afgetrokken.")
    except ValueError:
        print("Geef een geldig geheel getal in.")


def werknemer_ontslaan(conn):
    cursor = conn.cursor()

    werknemer_id = input("Geef de id van de werknemer: ")

    cursor.execute(
        "SELECT first_name, last_name FROM employees WHERE id = %s",
        (werknemer_id,)
    )
    werknemer = cursor.fetchone()

    if werknemer is None:
        print("Werknemer niet gevonden.")
        return

    naam = f"{werknemer[0]} {werknemer[1]}"
    bevestiging = input(f"Weet je zeker dat je werknemer {naam} wil ontslaan? (ja/nee): ")

    if bevestiging.lower() != "ja":
        print("Actie geannuleerd.")
        return

    cursor.execute(
        "DELETE FROM employees WHERE id = %s",
        (werknemer_id,)
    )
    conn.commit()

    print("Werknemer succesvol ontslagen.")


def toon_overzicht(conn):
    cursor = conn.cursor()

    cursor.execute("SHOW COLUMNS FROM employees")
    kolommen = [rij[0] for rij in cursor.fetchall()]

    velden = []
    if "id" in kolommen:
        velden.append("id")
    if "first_name" in kolommen:
        velden.append("first_name")
    if "last_name" in kolommen:
        velden.append("last_name")
    if "job_title" in kolommen:
        velden.append("job_title")
    if "salary" in kolommen:
        velden.append("salary")
    if "vacation_days" in kolommen:
        velden.append("vacation_days")

    if len(velden) < 3:
        print("De tabel bevat niet genoeg kolommen voor een overzicht.")
        return

    query = f"SELECT {', '.join(velden)} FROM employees ORDER BY id"
    cursor.execute(query)
    werknemers = cursor.fetchall()

    if not werknemers:
        print("Geen werknemers gevonden.")
        return

    print("\nOVERZICHT WERKNEMERS")
    print("-" * 90)

    heeft_functie = "job_title" in velden
    heeft_loon = "salary" in velden
    heeft_verlof = "vacation_days" in velden

    if heeft_functie:
        print(f"{'ID':5} {'Naam':25} {'Functie':20} {'Loon':10} {'Verlof':10}")
        print("-" * 90)
    else:
        print(f"{'ID':5} {'Naam':30} {'Loon':10} {'Verlof':10}")
        print("-" * 90)

    for werknemer in werknemers:
        i = 0
        werknemer_id = werknemer[i]
        i += 1
        voornaam = werknemer[i]
        i += 1
        achternaam = werknemer[i]
        i += 1
        naam = f"{voornaam} {achternaam}"

        if heeft_functie:
            functie = werknemer[i] if heeft_functie else "-"
            if heeft_functie:
                i += 1
            loon = werknemer[i] if heeft_loon else "-"
            if heeft_loon:
                i += 1
            verlof = werknemer[i] if heeft_verlof else "-"
            print(f"{werknemer_id:<5} {naam:25} {functie:20} {loon:<10} {verlof:<10}")
        else:
            loon = werknemer[i] if heeft_loon else "-"
            if heeft_loon:
                i += 1
            verlof = werknemer[i] if heeft_verlof else "-"
            print(f"{werknemer_id:<5} {naam:30} {loon:<10} {verlof:<10}")

    print("-" * 90)


def main():
    conn = connect_db()

    while True:
        print("\nKies een optie:")
        print("1. Loonsverhoging toewijzen")
        print("2. Verlofdagen toekennen bij nieuw jaar")
        print("3. Verlofdagen opnemen")
        print("4. Werknemer ontslaan")
        print("5. Overzicht tonen van alle werknemers")
        print("6. Stoppen")

        keuze = input("Maak je keuze: ")

        if keuze == "1":
            loonsverhoging(conn)
        elif keuze == "2":
            verlofdagen_toekennen(conn)
        elif keuze == "3":
            verlofdagen_opnemen(conn)
        elif keuze == "4":
            werknemer_ontslaan(conn)
        elif keuze == "5":
            toon_overzicht(conn)
        elif keuze == "6":
            break
        else:
            print("Ongeldige keuze.")

    conn.close()


if __name__ == "__main__":
    main()
```
  
