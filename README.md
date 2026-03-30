# Report z penetračního testu: SQL Injection

## Průzkum (Reconnaissance)
Testování začalo ověřením, zda je vyhledávací pole aplikace zranitelné vůči SQL Injection. K ověření byl použit znak jednoduché uvozovky (`'`), který slouží k předčasnému ukončení textového řetězce v SQL dotazu. Následným přidáním klauzule `ORDER BY` a zakomentováním zbytku dotazu (`-- -`) bylo možné sledovat reakci databáze. Aplikace vracela detailní chybová hlášení (např. *SQL Chyba: 1st ORDER BY term out of range*), což potvrdilo, že vstupy nejsou parametrizovány a aplikace je zranitelná vůči Error-based a UNION-based SQL Injection.

## Zjištění struktury (Enumeration)
Pro úspěšný UNION útok bylo nutné zjistit přesný počet sloupců, které vrací původní `SELECT` dotaz. 
Pomocí techniky `ORDER BY X` bylo zjištěno, že dotaz `ORDER BY 3-- -` vrací chybu (sloupec neexistuje), zatímco `ORDER BY 2-- -` proběhne v pořádku. Původní dotaz tedy vrací přesně **2 sloupce**.

Následně bylo nutné zjistit strukturu databáze. Vzhledem k tomu, že dotazy do `information_schema` končily chybou, bylo zřejmé, že se jedná o databázi **SQLite**. K výpisu existujících tabulek byl použit dotaz do systémové tabulky `sqlite_master`:
`' UNION SELECT name, sql FROM sqlite_master WHERE type='table'-- -`

Tento dotaz vypsal strukturu databáze přímo do uživatelského rozhraní. Byla nalezena zajímavá tabulka s názvem `config`, která byla vytvořena se sloupci `key` a `value` (dle vypsaného příkazu `CREATE TABLE config (key TEXT, value TEXT)`).

## Exfiltrace dat (Data Extraction)
Jelikož zadání indikovalo, že se vlajka nachází v konfiguračních údajích, byla pro finální extrakci dat zacílena tabulka `config`. Do vyhledávače byl vložen následující payload, který propojil původní (prázdný/existující) výsledek s obsahem sloupců `key` a `value` z cílové tabulky:

`' UNION SELECT key, value FROM config-- -`

Aplikace úspěšně vypsala obsah konfigurační tabulky, včetně hledané vlajky `FLAG{Sql_Un1on_M4st3r_S0SE}`.
