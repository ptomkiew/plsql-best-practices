# Najlepsze Praktyki Formatowania Kodu PL/SQL

[![License](https://img.shields.io/badge/license-Internal%20Use-blue.svg)](LICENSE)
[![Oracle PL/SQL](https://img.shields.io/badge/Oracle-PL%2FSQL-red.svg)](https://www.oracle.com/database/technologies/appdev/plsql.html)
[![Language](https://img.shields.io/badge/język-polski-green.svg)](README.md)

## 📖 O projekcie

Repozytorium zawiera szczegółowy przewodnik dotyczący najlepszych praktyk formatowania kodu PL/SQL w języku polskim. Dokument został stworzony w celu standaryzacji pisania kodu w projektach bazodanowych Oracle, zwiększenia jego czytelności i ułatwienia współpracy zespołowej.

## 📋 Zawartość

Dokument obejmuje następujące tematy:

1. **Konwencje nazewnictwa**
   - Nazwy tabel i kolumn
   - Nazwy zmiennych (i_, o_, io_, v_, c_, g_)
   - Prefiksy określające typ i zakres zmiennych

2. **Wcięcia i formatowanie**
   - Standardy wcięć (4 spacje)
   - Formatowanie bloków BEGIN-END
   - Struktura kodu

3. **Instrukcje SQL**
   - Formatowanie SELECT, INSERT, UPDATE, DELETE
   - Stosowanie aliasów tabel
   - Rozbijanie długich warunków WHERE

4. **Procedury i funkcje**
   - Deklaracja parametrów z prefiksami kierunkowymi
   - Struktura procedur i funkcji
   - Organizacja sekcji deklaracyjnych

5. **Obsługa wyjątków**
   - Prawidłowa struktura EXCEPTION
   - Logowanie błędów
   - Wyjątki własne vs. systemowe

6. **Komentarze i dokumentacja**
   - Komentarze nagłówkowe
   - Komentowanie logiki biznesowej
   - Nagłówki sekcji w dłuższych procedurach

7. **Najlepsze praktyki**
   - Bulk operations (BULK COLLECT, FORALL)
   - Zarządzanie transakcjami
   - Autonomous transactions
   - Obsługa wartości NULL
   - Używanie kursorów FOR LOOP

## 🎯 Dla kogo?

Dokument przeznaczony jest dla:
- Programistów PL/SQL
- Administratorów baz danych Oracle
- Architektów systemów
- Zespołów developerskich pracujących z bazami Oracle
- Osób uczących się PL/SQL

## 📚 Dokument główny

➡️ **[Najlepsze Praktyki Formatowania Kodu PL/SQL](najlepsze_praktyki_formatowania_plsql.md)**

## 🔑 Kluczowe konwencje

### Prefiksy parametrów:
- `i_` - parametry **IN** (wejściowe)
- `o_` - parametry **OUT** (wyjściowe)
- `io_` - parametry **IN OUT** (dwukierunkowe)

### Prefiksy zmiennych:
- `v_` - zmienne lokalne
- `c_` - kursory
- `g_` - zmienne globalne

### Przykład:
```sql
CREATE OR REPLACE PROCEDURE przetworz_dane (
    i_id_klienta     IN     NUMBER,
    o_wynik          OUT    VARCHAR2,
    io_licznik       IN OUT NUMBER
) IS
    v_temp           NUMBER;
    c_dane CURSOR IS SELECT * FROM klienci WHERE id = i_id_klienta;
BEGIN
    -- Implementacja
    NULL;
END przetworz_dane;
/
```

## 🛠️ Zalecane narzędzia

- **Oracle SQL Developer** - darmowe IDE z wbudowanym formatterem (Ctrl+F7)
- **PL/SQL Developer** - komercyjne narzędzie z Beautifier
- **SQLcl** - narzędzie command-line Oracle
- **PLDoc** - generator dokumentacji z kodu PL/SQL

## 📖 Dodatkowe zasoby

- [Oracle Database PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)
- [Oracle Database Development Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/adfns/)
- "Oracle PL/SQL Best Practices" - Steven Feuerstein
- [Oracle Live SQL](https://livesql.oracle.com/) - interaktywne przykłady

## 📄 Licencja

Dokument przeznaczony do użytku wewnętrznego.

## 👥 Autor

**GitHub Copilot**  
Data utworzenia: 7 listopada 2025  
Wersja dokumentu: 1.0

---

## 🤝 Wkład w projekt

Sugestie i poprawki są mile widziane! W przypadku pytań lub propozycji ulepszeń, proszę o kontakt.

## ⭐ Jeśli dokument Ci pomógł

Jeśli ten przewodnik okazał się przydatny, rozważ oznaczenie repozytorium gwiazdką ⭐ na GitHubie!