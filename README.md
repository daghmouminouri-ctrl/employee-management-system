# employee-management-system
employee-management-system

Via een menu voert het programma verschillende acties uit : loon aanpassen, verlofdagen toekennen, verlof aftrekken, werknemers verwijderen en een overzicht tonen. Er wordt telkens een keuze gemaakt, waarna het programma de gekozen functie uitvoert en de wijzigingen in de database opslaat. Het is opgebouwd uit losse functies die elk één taak uitvoeren. Eerst maakt het programma verbinding met de database. Daarna blijft het menu herhalen tot dat de keuze wordt gemaakt om te stoppen. Bij bijna elke actie controleert de code eerst of de invoer geldig is. Wanneer de gebruiker een getal moet ingeven, gebruikt de code ‘try’ en ‘except ValueError ‘ zodat het programma niet crasht als er tekst wordt ingevoerd in plaats van een getal. Voor werknemers die met een id gezocht worden, betekent ‘fetchone()’ met ‘None’ dat er geen werknemer gevonden is. Dan stopt de functie meteen en wordt er geen wijziging uitgevoerd voor die werknemer.

