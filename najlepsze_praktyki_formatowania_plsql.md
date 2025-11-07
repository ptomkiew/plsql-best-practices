# Najlepsze Praktyki Formatowania Kodu PL/SQL

## Spis treści
1. [Wprowadzenie](#wprowadzenie)
2. [Konwencje nazewnictwa](#konwencje-nazewnictwa)
3. [Wcięcia i formatowanie](#wcięcia-i-formatowanie)
4. [Instrukcje SQL](#instrukcje-sql)
5. [Procedury i funkcje](#procedury-i-funkcje)
6. [Obsługa wyjątków](#obsługa-wyjątków)
7. [Komentarze i dokumentacja](#komentarze-i-dokumentacja)
8. [Najlepsze praktyki](#najlepsze-praktyki)

---

## Wprowadzenie

Dokument ten przedstawia standardy i najlepsze praktyki formatowania kodu PL/SQL, których celem jest zwiększenie czytelności, utrzymywalności i spójności kodu w projektach bazodanowych Oracle.

---

## Konwencje nazewnictwa

### 1.1 Nazwy tabel i kolumn

**Założenie:** Używaj wielkich liter dla słów kluczowych SQL i małych liter dla nazw obiektów użytkownika.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE TABLE pracownicy (
  id_pracownika  NUMBER(10) NOT NULL,
  imie       VARCHAR2(50),
  nazwisko     VARCHAR2(50),
  data_zatrudnienia DATE,
  CONSTRAINT pk_pracownicy PRIMARY KEY (id_pracownika)
);
```

**Wyjaśnienie:** Stosowanie spójnej konwencji wielkości liter pozwala szybko rozróżnić słowa kluczowe SQL od nazw obiektów utworzonych przez użytkownika. Ułatwia to czytanie i rozumienie kodu.

### 1.2 Nazwy obiektów bazodanowych

**Założenie:** Stosuj standardową konwencję nazewniczą dla wszystkich obiektów bazodanowych.

#### 1.2.1 Tabele i podstawowe ograniczenia

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Tabela** | Nazwa w liczbie mnogiej (ostatni człon) | `POLICIES` |
| | | `POLICY_ROLES` |
| | | `POLICY_ROLE_TYPES` |
| **Primary Key** | `<table>_PK` | `POLICIES_PK` |
| **Foreign Key** | `<table>_<ref_table>_<optional_name>_FK` | `POLICIES_CUSTOMERS_FK` |
| | ⚠️ **Zawsze tworzyć indeks na kolumnie FK!** | `CUSTOMERS_CUSTOMERS_FK` |
| **Unique Key** | `<table>_<optional_name>_UK` | `POLICIES_UK` |
| **Check Constraint** | `<table>_<optional_name>_CHK` | `POLICIES_STATUS_CHK` |
| | | `POLICIES_FLAG_CHK` |

#### 1.2.2 Indeksy

| Typ indeksu | Konwencja nazewnicza | Przykład |
|------------|---------------------|----------|
| **Indeks dla PK** | `<PK_name>_IDX` | `POLICIES_PK_IDX` |
| **Indeks dla UK** | `<UK_name>_IDX` | `POLICIES_UK_IDX` |
| **Indeks dla FK** | `<FK_name>_IDX` | `POLICIES_CUSTOMERS_FK_IDX` |
| **Indeks pozostałe** | `<table>_<optional_name>_IDX` | `POLICIES_CUSTOMER_NAME_IDX` |

#### 1.2.3 Triggery

| Element | Opis | Przykład |
|---------|------|----------|
| **Konwencja** | `<table>_<BA_flag><IUD><RS_flag>_<HIST>_TRG` | |
| **BA_flag** | B (before) lub A (after) | `POLICIES_BIUR_TRG` |
| **IUD** | I (insert), U (update), D (delete) | `POLICIES_AIUDR_HIST_TRG` |
| **RS_flag** | R (row) lub S (statement) | |
| **HIST** | Opcjonalny element dla triggerów historycznych | |

**Przykłady triggerów:**
- `POLICIES_BIUR_TRG` - before insert/update row
- `POLICIES_AIUDR_HIST_TRG` - after insert/update/delete row (historyzujący)

#### 1.2.4 Sekwencje

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Sequence** | `<optional_table>_<optional_column>_<optional_name>_SEQ` | `POLICIES_ID_SEQ` |

#### 1.2.5 Widoki i tabele specjalne

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Widoki** | `<name>_V` | `ACTIVE_POLICIES_V` |
| **Widoki zmaterializowane** | `<name>_MV` | `POLICY_SUMMARY_MV` |
| **External Table** | `<name>_EXT` | `CUSTOMER_DATA_EXT` |
| **Global Temporary Table** | `<name>_TMP` | `CALCULATION_RESULTS_TMP` |

#### 1.2.6 Typy danych

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Typy** | `<name>_T` | `EMPLOYEE_RECORD_T` |
| **Typy - table of** | `<name>_TT` | `EMPLOYEE_LIST_TT` |

#### 1.2.7 Obiekty systemowe

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Directory** | `<name>_DIR` | `DATA_EXPORT_DIR` |

#### 1.2.8 Obiekty PL/SQL

| Typ obiektu | Konwencja nazewnicza | Przykład |
|-------------|---------------------|----------|
| **Pakiet PL/SQL (INSIS events)** | `<name>_SRV` | `POLICY_HANDLER_SRV` |
| **Procedura PL/SQL** | `<name>_PRC` | `CALCULATE_PREMIUM_PRC` |
| **Funkcja PL/SQL** | `<name>_FUN` | `GET_CUSTOMER_RATING_FUN` |

**Przykład zastosowania:**
```sql
-- ✓ POPRAWNIE
CREATE TABLE policies (
  policy_id          NUMBER(10) NOT NULL,
  customer_id        NUMBER(10) NOT NULL,
  policy_number      VARCHAR2(50) NOT NULL,
  status             VARCHAR2(20) DEFAULT 'ACTIVE',
  created_date       DATE DEFAULT SYSDATE,
  
  CONSTRAINT policies_pk PRIMARY KEY (policy_id),
  CONSTRAINT policies_customers_fk FOREIGN KEY (customer_id) 
    REFERENCES customers (customer_id),
  CONSTRAINT policies_status_chk CHECK (status IN ('ACTIVE', 'INACTIVE', 'SUSPENDED'))
);

-- Indeksy (OBOWIĄZKOWE dla FK!)
CREATE INDEX policies_pk_idx ON policies (policy_id);
CREATE INDEX policies_customers_fk_idx ON policies (customer_id);
CREATE INDEX policies_policy_number_idx ON policies (policy_number);

-- Sequence
CREATE SEQUENCE policies_id_seq
  START WITH 1
  INCREMENT BY 1
  NOCACHE;

-- Trigger historyzujący
CREATE OR REPLACE TRIGGER policies_aiudr_hist_trg
  AFTER INSERT OR UPDATE OR DELETE ON policies
  FOR EACH ROW
BEGIN
  -- Logika historyzacji
  NULL;
END;
/
```

### 1.3 Nazwy zmiennych PL/SQL

**Założenie:** Używaj standardowych prefiksów dla wszystkich zmiennych, parametrów i obiektów PL/SQL.

#### Prefiksy zmiennych PL/SQL

**Zmienne podstawowe:**

| Typ zmiennej | Prefiks | Przykład |
|-------------|---------|----------|
| **Zmienna globalna** | `g_` | `g_session_user` |
| **Zmienna lokalna** | `l_` | `l_counter` |
| **Stała** | `c_` | `c_max_attempts` |

**Struktury danych:**

| Typ struktury | Prefiks | Przykład |
|---------------|---------|----------|
| **Typ danych** | `t_` | `t_employee_rec` |
| **Kursor** | `cur_` | `cur_employees` |
| **Zmienna lokalna rekordowa** | `lr_` | `lr_employee_data` |
| **Zmienna globalna rekordowa** | `gr_` | `gr_session_info` |
| **Zmienna lokalna typu kolekcja** | `lt_` | `lt_employee_list` |
| **Zmienna globalna typu kolekcja** | `gt_` | `gt_lookup_cache` |

**Wyjątki i parametry:**

| Typ elementu | Prefiks | Przykład |
|-------------|---------|----------|
| **Wyjątek** | `e_` | `e_invalid_data` |
| **Parametr IN** | `i_` | `i_employee_id` |
| **Parametr OUT** | `o_` | `o_result_count` |
| **Parametr IN OUT** | `io_` | `io_error_message` |

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE oblicz_wynagrodzenie (
  i_id_pracownika    IN     NUMBER,
  i_uwzglednij_bonus IN     BOOLEAN DEFAULT TRUE,
  o_wynagrodzenie    OUT    NUMBER,
  io_komunikat       IN OUT VARCHAR2
) IS
  -- Stałe
  c_bonus_multiplier  CONSTANT NUMBER := 1.2;
  c_max_salary       CONSTANT NUMBER := 50000;
  
  -- Zmienne lokalne
  l_podstawa         NUMBER;
  l_bonus           NUMBER := 0;
  
  -- Kursor
  cur_dane_pracownika CURSOR IS
    SELECT p.wynagrodzenie_podstawowe, 
           p.premia_roczna
      FROM pracownicy p
     WHERE p.id_pracownika = i_id_pracownika;
  
  -- Zmienna rekordowa
  lr_pracownik       cur_dane_pracownika%ROWTYPE;
  
  -- Wyjątek własny
  e_nieprawidlowy_id EXCEPTION;
  
BEGIN
  OPEN cur_dane_pracownika;
  FETCH cur_dane_pracownika INTO lr_pracownik;
  
  IF cur_dane_pracownika%NOTFOUND THEN
    CLOSE cur_dane_pracownika;
    RAISE e_nieprawidlowy_id;
  END IF;
  CLOSE cur_dane_pracownika;
  
  l_podstawa := lr_pracownik.wynagrodzenie_podstawowe;
  
  IF i_uwzglednij_bonus THEN
    l_bonus := lr_pracownik.premia_roczna * c_bonus_multiplier;
  END IF;
  
  o_wynagrodzenie := l_podstawa + l_bonus;
  io_komunikat := io_komunikat || ' - Przeliczono z bonusem: ' || l_bonus;
  
EXCEPTION
  WHEN e_nieprawidlowy_id THEN
    RAISE_APPLICATION_ERROR(-20001, 'Nie znaleziono pracownika o ID: ' || i_id_pracownika);
END oblicz_wynagrodzenie;
/
```

**Wyjaśnienie:** Standardowe prefiksy pozwalają natychmiast zidentyfikować typ i zakres zmiennej. Szczególnie ważne jest rozróżnienie między zmiennymi lokalnymi (`l_`) a globalnymi (`g_`), oraz między różnymi typami kolekcji i rekordów. Stosowanie wyjątków z prefiksem `e_` ułatwia identyfikację własnych wyjątków biznesowych.

---

## Wcięcia i formatowanie

### 2.1 Wcięcia

**Założenie:** Używaj 2 spacji dla każdego poziomu wcięcia. Nie używaj tabulatorów.

**Przykład:**
```sql
-- ✓ POPRAWNIE
BEGIN
  IF l_wartosc > 0 THEN
    FOR i IN 1..10 LOOP
      IF i MOD 2 = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Parzysta: ' || i);
      ELSE
        DBMS_OUTPUT.PUT_LINE('Nieparzysta: ' || i);
      END IF;
    END LOOP;
  END IF;
END;
/
```

**Wyjaśnienie:** Konsekwentne wcięcia tworzą wizualną hierarchię struktury kodu, ułatwiając identyfikację bloków BEGIN-END, pętli i instrukcji warunkowych. 2 spacje oferują dobry kompromis między czytelnością a oszczędnością miejsca, jednocześnie zwiększając czytelność w długich blokach kodu z wieloma poziomami zagnieżdżenia.

### 2.2 Formatowanie bloków BEGIN-END

**Założenie:** Słowa kluczowe BEGIN i END umieszczaj na osobnych liniach, na tym samym poziomie wcięcia.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE proces_zamowienia (
  i_id_zamowienia IN NUMBER
) IS
  l_status VARCHAR2(20);
BEGIN
  SELECT z.status 
    INTO l_status
    FROM zamowienia z
   WHERE z.id_zamowienia = i_id_zamowienia;
  
  IF l_status = 'NOWE' THEN
    BEGIN
      -- Zagnieżdżony blok
      UPDATE zamowienia z
         SET z.status = 'W_REALIZACJI'
       WHERE z.id_zamowienia = i_id_zamowienia;
      
      COMMIT;
    EXCEPTION
      WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
    END;
  END IF;
END proces_zamowienia;
/
```

**Wyjaśnienie:** Umieszczanie BEGIN i END na osobnych liniach z odpowiednim wcięciem tworzy czytelną strukturę blokową. Każdy blok jest wyraźnie odgraniczony, co ułatwia debugowanie i zrozumienie logiki programu.

---

## Instrukcje SQL

### 3.1 Formatowanie SELECT

**Założenie:** Umieszczaj każdą kolumnę w SELECT na osobnej linii. Klauzule (FROM, WHERE, ORDER BY) rozpoczynaj od nowej linii.

**Przykład:**
```sql
-- ✓ POPRAWNIE
SELECT 
  p.id_pracownika,
  p.imie,
  p.nazwisko,
  p.data_zatrudnienia,
  d.nazwa AS nazwa_dzialu,
  d.lokalizacja,
  s.stanowisko,
  s.wynagrodzenie
FROM 
  pracownicy p
  INNER JOIN dzialy d 
    ON p.id_dzialu = d.id_dzialu
  LEFT JOIN stanowiska s 
    ON p.id_stanowiska = s.id_stanowiska
WHERE 
  p.aktywny = 'T'
  AND p.data_zatrudnienia >= ADD_MONTHS(SYSDATE, -12)
  AND d.lokalizacja IN ('Warszawa', 'Kraków', 'Wrocław')
ORDER BY 
  d.nazwa,
  p.nazwisko,
  p.imie;
```

**Wyjaśnienie:** Umieszczanie każdej kolumny na osobnej linii pozwala łatwo zobaczyć wszystkie wybierane pola. W JOIN-ach klauzula `ON` powinna być w nowej linii z wcięciem dla lepszej czytelności warunków łączenia. Formatowanie klauzul WHERE z warunkami na oddzielnych liniach ułatwia modyfikację zapytań i szybkie lokalizowanie błędów logicznych.

### 3.2 Formatowanie INSERT

**Założenie:** Wymień wszystkie kolumny jawnie, umieszczając je na osobnych liniach lub grupując logicznie.

**Przykład:**
```sql
-- ✓ POPRAWNIE - dla małej liczby kolumn
INSERT INTO pracownicy (
  id_pracownika,
  imie,
  nazwisko,
  email,
  data_zatrudnienia
) VALUES (
  pracownicy_id_seq.NEXTVAL,
  'Jan',
  'Kowalski',
  'jan.kowalski@firma.pl',
  SYSDATE
);

-- ✓ POPRAWNIE - dla większej liczby kolumn
INSERT INTO pracownicy (
  id_pracownika, 
  imie, 
  nazwisko, 
  email,
  telefon, 
  adres, 
  kod_pocztowy, 
  miasto,
  data_zatrudnienia, 
  id_dzialu, 
  id_stanowiska
) VALUES (
  pracownicy_id_seq.NEXTVAL, 
  'Jan', 
  'Kowalski', 
  'jan.kowalski@firma.pl',
  '123456789', 
  'ul. Główna 1', 
  '00-001', 
  'Warszawa',
  SYSDATE, 
  10, 
  5
);
```

**Wyjaśnienie:** Jawne wymienianie kolumn zapobiega błędom przy zmianach struktury tabeli (dodanie/usunięcie kolumn). Grupowanie wartości w logiczne zestawy (dane osobowe, adresowe, służbowe) zwiększa czytelność dla tabel z wieloma kolumnami.

### 3.3 Formatowanie UPDATE

**Założenie:** Każde przypisanie SET na osobnej linii. Warunki WHERE z odpowiednim wcięciem.

**Przykład:**
```sql
-- ✓ POPRAWNIE
UPDATE pracownicy p
   SET p.wynagrodzenie = p.wynagrodzenie * 1.1,
       p.data_ostatniej_podwyzki = SYSDATE,
       p.zaktualizowal = USER,
       p.data_aktualizacji = SYSTIMESTAMP
 WHERE p.id_dzialu = 10
   AND p.ocena_roczna >= 4
   AND p.data_ostatniej_podwyzki < ADD_MONTHS(SYSDATE, -12);
```

**Wyjaśnienie:** Rozdzielenie każdego przypisania na osobną linię pozwala łatwo śledzić, które pola są aktualizowane. Używanie aliasu tabeli (`p`) zwiększa czytelność i jest spójne z praktykami stosowanymi w SELECT i JOIN. Jest to szczególnie ważne podczas code review i debugowania, gdy trzeba szybko zidentyfikować, co dokładnie jest zmieniane.

---

## Procedury i funkcje

### 4.1 Deklaracja parametrów

**Założenie:** Każdy parametr na osobnej linii z wyrównaniem typów danych i trybów (IN/OUT/IN OUT). Używaj prefiksów: i_ dla IN, o_ dla OUT, io_ dla IN OUT.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE przetworz_zamowienie (
  i_id_zamowienia    IN     NUMBER,
  i_id_klienta       IN     NUMBER,
  i_tryb_platnosci   IN     VARCHAR2,
  i_kod_rabatowy     IN     VARCHAR2 DEFAULT NULL,
  o_status_wynikowy  OUT    VARCHAR2,
  o_kwota_koncowa    OUT    NUMBER,
  io_komunikat_bledu IN OUT VARCHAR2
) IS
  l_rabat        NUMBER := 0;
  l_kwota_brutto NUMBER;
  l_kwota_netto  NUMBER;
BEGIN
  -- Implementacja
  NULL;
END przetworz_zamowienie;
/

**Wyjaśnienie:** Wyrównanie parametrów w kolumnach (nazwa, tryb, typ) tworzy czytelną "tabelę" parametrów. Prefiksy i_, o_, io_ natychmiast informują o kierunku przepływu danych. Łatwo zauważyć, które parametry są wejściowe, wyjściowe czy dwukierunkowe. Wyrównanie typów danych pomaga szybko zidentyfikować niezgodności typów.

### 4.2 Struktura procedury/funkcji

**Założenie:** Zachowaj spójną strukturę: nagłówek → deklaracje → ciało → obsługa wyjątków.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE FUNCTION oblicz_podatek (
  i_kwota_brutto   IN NUMBER,
  i_stawka_vat     IN NUMBER DEFAULT 23
) RETURN NUMBER
IS
  -- Sekcja deklaracji stałych
  c_max_kwota    CONSTANT NUMBER := 1000000;
  c_min_stawka   CONSTANT NUMBER := 0;
  
  -- Sekcja deklaracji zmiennych
  l_kwota_podatku  NUMBER;
  l_kwota_netto    NUMBER;
  
  -- Sekcja deklaracji wyjątków własnych
  e_nieprawidlowa_kwota  EXCEPTION;
  e_nieprawidlowa_stawka EXCEPTION;
  
BEGIN
  -- Walidacja danych wejściowych
  IF i_kwota_brutto <= 0 
    OR i_kwota_brutto > c_max_kwota 
  THEN
    RAISE e_nieprawidlowa_kwota;
  END IF;
  
  IF i_stawka_vat < c_min_stawka 
    OR i_stawka_vat > 100 
  THEN
    RAISE e_nieprawidlowa_stawka;
  END IF;
  
  -- Logika biznesowa
  l_kwota_netto := i_kwota_brutto / (1 + i_stawka_vat / 100);
  l_kwota_podatku := i_kwota_brutto - l_kwota_netto;
  
  -- Logowanie
  INSERT INTO logi_podatkowe (
    data_obliczenia,
    kwota_brutto,
    stawka_vat,
    kwota_podatku
  ) VALUES (
    SYSTIMESTAMP,
    i_kwota_brutto,
    i_stawka_vat,
    l_kwota_podatku
  );
  
  RETURN ROUND(l_kwota_podatku, 2);
  
EXCEPTION
  WHEN e_nieprawidlowa_kwota THEN
    RAISE_APPLICATION_ERROR(-20001, 
      'Kwota brutto musi być między 0 a ' || c_max_kwota);
  
  WHEN e_nieprawidlowa_stawka THEN
    RAISE_APPLICATION_ERROR(-20002, 
      'Stawka VAT musi być między 0 a 100');
  
  WHEN OTHERS THEN
    RAISE_APPLICATION_ERROR(-20999, 
      'Błąd podczas obliczania podatku: ' || SQLERRM);
END oblicz_podatek;
/
```

**Wyjaśnienie:** Uporządkowana struktura z wyraźnie oddzielonymi sekcjami (stałe, zmienne, wyjątki, walidacja, logika, obsługa błędów) sprawia, że kod jest przewidywalny i łatwy w nawigacji. Programista wie, gdzie szukać konkretnych elementów. Grupowanie deklaracji według typu ułatwia zarządzanie zakresem zmiennych.

---

## Obsługa wyjątków

### 5.1 Struktura EXCEPTION

**Założenie:** Obsługuj wyjątki specyficzne przed ogólnym WHEN OTHERS. Zawsze loguj błędy.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE aktualizuj_zamowienie (
  i_id_zamowienia IN NUMBER,
  i_nowy_status   IN VARCHAR2
) IS
  l_aktualny_status VARCHAR2(20);
  
  e_nieprawidlowy_status EXCEPTION;
  e_zamowienie_nieznalezione EXCEPTION;
  
BEGIN
  -- Sprawdzenie istnienia zamówienia
  BEGIN
    SELECT z.status 
      INTO l_aktualny_status
      FROM zamowienia z
     WHERE z.id_zamowienia = i_id_zamowienia;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      RAISE e_zamowienie_nieznalezione;
  END;
  
  -- Walidacja statusu
  IF i_nowy_status NOT IN ('NOWE', 'W_REALIZACJI', 'WYSLANE', 'ZAKONCZONE') THEN
    RAISE e_nieprawidlowy_status;
  END IF;
  
  -- Aktualizacja
  UPDATE zamowienia z
     SET z.status = i_nowy_status,
         z.data_modyfikacji = SYSTIMESTAMP,
         z.uzytkownik_modyfikujacy = USER
   WHERE z.id_zamowienia = i_id_zamowienia;
  
  COMMIT;
  
  -- Logowanie sukcesu
  INSERT INTO logi_zmian (
    tabela, 
    id_rekordu, 
    akcja, 
    uzytkownik, 
    data_operacji
  ) VALUES (
    'ZAMOWIENIA', 
    i_id_zamowienia, 
    'ZMIANA_STATUSU', 
    USER, 
    SYSTIMESTAMP
  );
  
EXCEPTION
  WHEN e_zamowienie_nieznalezione THEN
    ROLLBACK;
    -- Logowanie błędu
    INSERT INTO logi_bledow (
      procedura, 
      komunikat, 
      uzytkownik, 
      data_bledu
    ) VALUES (
      'AKTUALIZUJ_ZAMOWIENIE',
      'Nie znaleziono zamówienia o ID: ' || i_id_zamowienia,
      USER,
      SYSTIMESTAMP
    );
    COMMIT;
    RAISE_APPLICATION_ERROR(-20001, 
      'Zamówienie o podanym ID nie istnieje');
  
  WHEN e_nieprawidlowy_status THEN
    ROLLBACK;
    INSERT INTO logi_bledow (
      procedura, 
      komunikat, 
      uzytkownik, 
      data_bledu
    ) VALUES (
      'AKTUALIZUJ_ZAMOWIENIE',
      'Nieprawidłowy status: ' || i_nowy_status,
      USER,
      SYSTIMESTAMP
    );
    COMMIT;
    RAISE_APPLICATION_ERROR(-20002, 
      'Nieprawidłowy status zamówienia');
  
  WHEN OTHERS THEN
    ROLLBACK;
    INSERT INTO logi_bledow (
      procedura, 
      komunikat, 
      kod_bledu, 
      uzytkownik, 
      data_bledu
    ) VALUES (
      'AKTUALIZUJ_ZAMOWIENIE',
      SQLERRM,
      SQLCODE,
      USER,
      SYSTIMESTAMP
    );
    COMMIT;
    RAISE;
    
END aktualizuj_zamowienie;
/
```

**Wyjaśnienie:** Prawidłowa obsługa wyjątków jest kluczowa dla stabilności aplikacji. Obsługa specyficznych wyjątków przed WHEN OTHERS pozwala na precyzyjną reakcję na różne sytuacje błędne. Logowanie błędów jest niezbędne do debugowania i monitorowania. Nigdy nie używaj pustego WHEN OTHERS, który ukrywa błędy.

---

## Komentarze i dokumentacja

### 6.1 Komentarze nagłówkowe

**Założenie:** Każda procedura/funkcja powinna mieć komentarz nagłówkowy opisujący cel, parametry, wartość zwracaną i autora.

**Przykład:**
```sql
-- ✓ POPRAWNIE
/*******************************************************************************
 * Nazwa: oblicz_wynagrodzenie_roczne
 * Cel: Oblicza roczne wynagrodzenie pracownika z uwzględnieniem premii i bonusów
 * 
 * Parametry:
 *   i_id_pracownika  - ID pracownika (klucz główny tabeli PRACOWNICY)
 *   i_rok      - Rok, dla którego obliczamy wynagrodzenie
 *   i_uwzglednij_bonus - Czy uwzględnić bonus roczny (domyślnie TAK)
 * 
 * Zwraca:
 *   NUMBER - Całkowite wynagrodzenie roczne w PLN
 * 
 * Wyjątki:
 *   -20001: Nie znaleziono pracownika o podanym ID
 *   -20002: Nieprawidłowy rok (mniejszy niż rok zatrudnienia)
 * 
 * Historia zmian:
 *   2025-11-07  Jan Kowalski  Utworzenie
 *   2025-10-15  Anna Nowak  Dodanie obsługi bonusów kwartalnych
 * 
 * Przykład użycia:
 *   DECLARE
 *   l_wynagrodzenie NUMBER;
 *   BEGIN
 *   l_wynagrodzenie := oblicz_wynagrodzenie_roczne(
 *     i_id_pracownika => 1001,
 *     i_rok => 2025,
 *     i_uwzglednij_bonus => TRUE
 *   );
 *   DBMS_OUTPUT.PUT_LINE('Wynagrodzenie: ' || l_wynagrodzenie);
 *   END;
 ******************************************************************************/
CREATE OR REPLACE FUNCTION oblicz_wynagrodzenie_roczne (
  i_id_pracownika  IN NUMBER,
  i_rok      IN NUMBER,
  i_uwzglednij_bonus   IN BOOLEAN DEFAULT TRUE
) RETURN NUMBER
IS
  l_wynagrodzenie_podstawowe NUMBER;
  l_premie         NUMBER := 0;
  l_bonusy         NUMBER := 0;
BEGIN
  -- Implementacja
  RETURN l_wynagrodzenie_podstawowe + l_premie + l_bonusy;
END oblicz_wynagrodzenie_roczne;
/
```

**Wyjaśnienie:** Szczegółowy komentarz nagłówkowy służy jako dokumentacja API dla innych programistów. Zawiera wszystkie informacje niezbędne do prawidłowego użycia procedury/funkcji bez konieczności czytania implementacji. Historia zmian pozwala śledzić ewolucję kodu.

### 6.2 Komentarze w kodzie

**Założenie:** Komentuj "dlaczego", nie "co". Używaj komentarzy do wyjaśnienia logiki biznesowej i nietypowych rozwiązań.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE przelicz_zamowienia IS
  l_kurs_waluty NUMBER;
BEGIN
  -- Pobieramy kurs z NBP dla daty bieżącej, ale jeśli nie ma kursu
  -- (np. weekend), cofamy się do ostatniego dnia roboczego
  BEGIN
    SELECT k.kurs 
      INTO l_kurs_waluty
      FROM kursy_nbp k
     WHERE k.waluta = 'EUR'
       AND k.data_kursu = TRUNC(SYSDATE);
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      -- Szukamy ostatniego dostępnego kursu (max 7 dni wstecz)
      SELECT k.kurs
        INTO l_kurs_waluty
        FROM kursy_nbp k
       WHERE k.waluta = 'EUR'
         AND k.data_kursu = (
        SELECT MAX(k2.data_kursu)
          FROM kursy_nbp k2
         WHERE k2.waluta = 'EUR'
           AND k2.data_kursu <= TRUNC(SYSDATE)
           AND data_kursu >= TRUNC(SYSDATE) - 7
      );
  END;
  
  -- UWAGA: Celowo nie używamy MERGE, ponieważ w naszej wersji Oracle (11g)
  -- występuje znany bug przy MERGE z funkcjami deterministycznymi (Bug #14719814)
  FOR rec IN (SELECT z.id_zamowienia, 
                     z.kwota_eur 
                FROM zamowienia z 
               WHERE z.status = 'AKTYWNE') LOOP
    UPDATE zamowienia z
       SET z.kwota_pln = rec.kwota_eur * l_kurs_waluty
     WHERE z.id_zamowienia = rec.id_zamowienia;
  END LOOP;
  
  COMMIT;
END przelicz_zamowienia;
/

**Wyjaśnienie:** Dobre komentarze wyjaśniają intencje i kontekst biznesowy, nie oczywiste operacje. Komentarz "pobieramy kurs" jest zbędny, bo kod jest czytelny. Natomiast wyjaśnienie, dlaczego cofamy się do ostatniego dnia roboczego lub dlaczego nie używamy MERGE, dostarcza cennej informacji, której nie da się wywnioskować z samego kodu.

### 6.3 Nagłówki sekcji

**Założenie:** Używaj komentarzy do oznaczenia głównych sekcji w dłuższych procedurach.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE PROCEDURE kompleksowe_przetwarzanie_zamowien IS
BEGIN
  /*==========================================================================
   * SEKCJA 1: WALIDACJA I PRZYGOTOWANIE DANYCH
   *==========================================================================*/
  -- Sprawdzanie integralności danych wejściowych
  NULL;
  
  /*==========================================================================
   * SEKCJA 2: PRZETWARZANIE GŁÓWNE
   *==========================================================================*/
  -- Logika biznesowa przetwarzania zamówień
  NULL;
  
  /*==========================================================================
   * SEKCJA 3: AGREGACJA I RAPORTOWANIE
   *==========================================================================*/
  -- Generowanie statystyk i raportów
  NULL;
  
  /*==========================================================================
   * SEKCJA 4: FINALIZACJA I CZYSZCZENIE
   *==========================================================================*/
  -- Operacje końcowe i czyszczenie danych tymczasowych
  NULL;
  
END kompleksowe_przetwarzanie_zamowien;
/
```

**Wyjaśnienie:** W długich procedurach (powyżej 200 linii) wizualne oddzielenie głównych sekcji znacząco ułatwia nawigację. Programista może szybko zlokalizować interesującą go część kodu. Nagłówki sekcji pełnią rolę "spisu treści" procedury.

---

## Najlepsze praktyki

### 7.1 Używanie aliasów tabel

**Założenie:** Zawsze używaj krótkich, znaczących aliasów dla tabel, szczególnie w JOIN-ach.

**Przykład:**
```sql
-- ✓ POPRAWNIE
SELECT 
  p.id_pracownika,
  p.nazwisko,
  d.nazwa AS nazwa_dzialu,
  m.nazwisko AS nazwisko_menadzera,
  s.wynagrodzenie
FROM 
  pracownicy p
  INNER JOIN dzialy d 
    ON p.id_dzialu = d.id_dzialu
  LEFT JOIN pracownicy m 
    ON p.id_menadzera = m.id_pracownika
  INNER JOIN stanowiska s 
    ON p.id_stanowiska = s.id_stanowiska
WHERE 
  p.data_zatrudnienia >= ADD_MONTHS(SYSDATE, -12)
  AND d.aktywny = 'T';

**Wyjaśnienie:** Aliasy skracają zapytania i zwiększają czytelność, szczególnie przy wielu JOIN-ach. Używanie pełnych nazw tabel jest rozwlekłe i utrudnia szybkie skanowanie kodu. Dobrze dobrane aliasy (p dla pracownicy, d dla dzialy) są intuicyjne i łatwe do zapamiętania.

### 7.2 Formatowanie długich warunków

**Założenie:** Rozbijaj długie warunki logiczne na osobne linie z odpowiednim wcięciem, grupując operatory.

**Przykład:**
```sql
-- ✓ POPRAWNIE
SELECT *
FROM zamowienia z
WHERE 
  -- Warunki czasowe
  (
    z.data_zamowienia >= TO_DATE('2025-01-01', 'YYYY-MM-DD')
    AND z.data_zamowienia < TO_DATE('2026-01-01', 'YYYY-MM-DD')
  )
  -- Warunki statusowe
  AND (
    z.status IN ('NOWE', 'W_REALIZACJI', 'WYSLANE')
    OR (z.status = 'ANULOWANE' AND z.przyczyna_anulowania = 'KLIENT')
  )
  -- Warunki finansowe
  AND (
    z.wartosc_brutto >= 1000
    OR (z.wartosc_brutto >= 500 AND z.klient_vip = 'T')
  )
  -- Warunki dodatkowe
  AND z.kraj IN ('PL', 'DE', 'CZ')
  AND z.waluta = 'PLN';
```

**Wyjaśnienie:** Grupowanie warunków według kategorii (czasowe, statusowe, finansowe) z komentarzami tworzy logiczną strukturę. Odpowiednie wcięcie pokazuje pierwszeństwo operatorów logicznych. To szczególnie ważne przy skomplikowanych warunkach biznesowych, gdzie błąd w nawiasach może prowadzić do nieprawidłowych wyników.

### 7.3 Używanie nazwanych parametrów

**Założenie:** Przy wywołaniach procedur/funkcji używaj nazwanych parametrów zamiast pozycyjnych.

**Przykład:**
```sql
-- ✓ POPRAWNIE
DECLARE
  l_wynik VARCHAR2(100);
BEGIN
  przetworz_platnosc(
    i_id_zamowienia => 12345,
    i_kwota => 1500.00,
    i_waluta => 'PLN',
    i_metoda_platnosci => 'KARTA',
    o_status_wynikowy => l_wynik
  );
  
  DBMS_OUTPUT.PUT_LINE('Status: ' || l_wynik);
END;
/

**Wyjaśnienie:** Nazwane parametry zwiększają czytelność i bezpieczeństwo kodu. Są odporne na zmianę kolejności parametrów w definicji procedury. Przy wywołaniu natychmiast widać, co oznacza każda wartość, bez konieczności sprawdzania sygnatur procedur.

### 7.4 Obsługa wartości NULL

**Założenie:** Zawsze jawnie obsługuj potencjalne wartości NULL w warunkach i porównaniach.

**Przykład:**
```sql
-- ✓ POPRAWNIE
CREATE OR REPLACE FUNCTION porownaj_wartosci (
  i_wartosc1 IN NUMBER,
  i_wartosc2 IN NUMBER
) RETURN VARCHAR2
IS
BEGIN
  IF i_wartosc1 IS NULL AND i_wartosc2 IS NULL THEN
    RETURN 'OBIE_NULL';
  ELSIF i_wartosc1 IS NULL THEN
    RETURN 'PIERWSZA_NULL';
  ELSIF i_wartosc2 IS NULL THEN
    RETURN 'DRUGA_NULL';
  ELSIF i_wartosc1 = i_wartosc2 THEN
    RETURN 'ROWNE';
  ELSIF i_wartosc1 > i_wartosc2 THEN
    RETURN 'PIERWSZA_WIEKSZA';
  ELSE
    RETURN 'DRUGA_WIEKSZA';
  END IF;
END porownaj_wartosci;
/

-- Przykład użycia NVL i COALESCE
SELECT p.id_pracownika,
       p.nazwisko,
       NVL(p.telefon_komorkowy, p.telefon_stacjonarny) AS telefon_kontaktowy,
       COALESCE(p.email_prywatny, p.email_firmowy, 'brak@email.pl') AS email,
       NVL(p.premia, 0) + NVL(p.bonus, 0) AS dodatki
  FROM pracownicy p;
```

**Wyjaśnienie:** NULL w SQL ma specjalną semantykę - nie jest równy niczemu, nawet innemu NULL. Brak obsługi NULL prowadzi do subtelnych błędów logicznych. Funkcje NVL i COALESCE pomagają zapewnić wartości domyślne, ale należy ich używać świadomie, rozumiejąc implikacje biznesowe zastępowania NULL wartością domyślną.

### 7.5 Używanie kursora FOR LOOP

**Założenie:** Preferuj niejawne kursory FOR LOOP zamiast jawnego otwierania i zamykania kursorów.

**Przykład:**
```sql
-- ✓ POPRAWNIE - kursor niejawny
CREATE OR REPLACE PROCEDURE aktualizuj_premie IS
BEGIN
  FOR rec IN (
    SELECT p.id_pracownika,
           p.wynagrodzenie,
           p.ocena_roczna
      FROM pracownicy p
     WHERE p.data_zatrudnienia < ADD_MONTHS(SYSDATE, -12)
       AND p.aktywny = 'T'
  ) LOOP
    UPDATE pracownicy p
       SET p.premia = rec.wynagrodzenie * (rec.ocena_roczna / 100)
     WHERE p.id_pracownika = rec.id_pracownika;
  END LOOP;
  
  COMMIT;
END aktualizuj_premie;
/

-- ✓ AKCEPTOWALNE - kursor jawny z parametrem
CREATE OR REPLACE PROCEDURE generuj_raport (
  i_id_dzialu IN NUMBER
) IS
  CURSOR c_pracownicy (cp_id_dzialu NUMBER) IS
    SELECT p.id_pracownika,
           p.imie,
           p.nazwisko,
           p.wynagrodzenie
      FROM pracownicy p
     WHERE p.id_dzialu = cp_id_dzialu
     ORDER BY p.nazwisko;
  
  l_suma_wynagrodzen NUMBER := 0;
BEGIN
  FOR rec IN c_pracownicy(i_id_dzialu) LOOP
    DBMS_OUTPUT.PUT_LINE(rec.nazwisko || ': ' || rec.wynagrodzenie);
    l_suma_wynagrodzen := l_suma_wynagrodzen + rec.wynagrodzenie;
  END LOOP;
  
  DBMS_OUTPUT.PUT_LINE('Suma: ' || l_suma_wynagrodzen);
END generuj_raport;
/
```

**Wyjaśnienie:** Pętla FOR automatycznie otwiera kursor, pobiera wiersze i zamyka kursor, nawet w przypadku wyjątku. Eliminuje to ryzyko pozostawienia otwartego kursora (wyciek zasobów). Kod jest krótszy i bardziej czytelny. Jawne kursory są przydatne tylko wtedy, gdy potrzebujesz większej kontroli (np. kursory z parametrami, sprawdzanie %NOTFOUND w specyficznych przypadkach).

### 7.6 Bulk operations (BULK COLLECT i FORALL)

**Założenie:** Dla operacji na dużych zbiorach danych używaj bulk operations zamiast row-by-row processing.

**Przykład:**
```sql
-- ✓ POPRAWNIE - bulk operations dla wydajności
CREATE OR REPLACE PROCEDURE archiwizuj_stare_zamowienia IS
  TYPE t_id_table IS TABLE OF zamowienia.id_zamowienia%TYPE;
  TYPE t_data_table IS TABLE OF zamowienia.data_zamowienia%TYPE;
  
  l_ids t_id_table;
  l_daty t_data_table;
  
  c_limit CONSTANT PLS_INTEGER := 1000; -- Przetwarzamy partiami
BEGIN
  -- Pobieranie danych bulk
  SELECT z.id_zamowienia, 
         z.data_zamowienia
    BULK COLLECT INTO l_ids, l_daty
    FROM zamowienia z
   WHERE z.data_zamowienia < ADD_MONTHS(SYSDATE, -24)
     AND z.status = 'ZAKONCZONE';
  
  -- Wstawianie do archiwum bulk
  FORALL i IN 1..l_ids.COUNT
    INSERT INTO zamowienia_archiwum (
      id_zamowienia,
      data_zamowienia,
      data_archiwizacji
    ) VALUES (
      l_ids(i),
      l_daty(i),
      SYSDATE
    );
  
  -- Usuwanie bulk
  FORALL i IN 1..l_ids.COUNT
    DELETE FROM zamowienia z
    WHERE z.id_zamowienia = l_ids(i);
  
  COMMIT;
  
  DBMS_OUTPUT.PUT_LINE('Zarchiwizowano ' || l_ids.COUNT || ' zamówień');
  
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    RAISE;
END archiwizuj_stare_zamowienia;
/
```

```sql
-- ✓ POPRAWNIE - przetwarzanie partiami dla bardzo dużych zbiorów
CREATE OR REPLACE PROCEDURE aktualizuj_masowo IS
  TYPE t_id_table IS TABLE OF pracownicy.id_pracownika%TYPE;
  l_ids t_id_table;
  
  CURSOR c_pracownicy IS
    SELECT p.id_pracownika
      FROM pracownicy p
     WHERE p.data_ostatniej_podwyzki < ADD_MONTHS(SYSDATE, -12);
  
  c_limit CONSTANT PLS_INTEGER := 5000;
BEGIN
  OPEN c_pracownicy;
  LOOP
    FETCH c_pracownicy BULK COLLECT INTO l_ids LIMIT c_limit;
    EXIT WHEN l_ids.COUNT = 0;
    
    FORALL i IN 1..l_ids.COUNT
      UPDATE pracownicy p
         SET p.wynagrodzenie = p.wynagrodzenie * 1.05,
             p.data_ostatniej_podwyzki = SYSDATE
       WHERE p.id_pracownika = l_ids(i);
    
    COMMIT; -- Commit po każdej partii
    
    DBMS_OUTPUT.PUT_LINE('Przetworzono partię: ' || l_ids.COUNT || ' rekordów');
  END LOOP;
  CLOSE c_pracownicy;
  
  DBMS_OUTPUT.PUT_LINE('Zakończono przetwarzanie masowe');
END aktualizuj_masowo;
/
```

**Wyjaśnienie:** Bulk operations drastycznie zwiększają wydajność dla operacji na wielu wierszach, redukując context switching między silnikiem PL/SQL a SQL. BULK COLLECT pobiera wszystkie wiersze jednocześnie, a FORALL wykonuje wszystkie DML w jednej operacji. Dla bardzo dużych zbiorów danych używamy LIMIT, aby uniknąć przepełnienia pamięci. Różnica w wydajności może być 10-50x dla tysięcy rekordów.

### 7.7 Transakcje i COMMIT

**Założenie:** Świadomie zarządzaj transakcjami. Unikaj auto-commit po każdej operacji.

**Przykład:**
```sql
-- ✓ POPRAWNIE - transakcje w jednostkach biznesowych
CREATE OR REPLACE PROCEDURE przetwórz_zamowienie (
  i_id_zamowienia IN NUMBER
) IS
  l_id_faktury NUMBER;
  l_id_wysylki NUMBER;
BEGIN
  -- Jedna transakcja dla całej operacji biznesowej
  SAVEPOINT przed_przetworzeniem;
  
  -- Krok 1: Aktualizacja statusu zamówienia
  UPDATE zamowienia z
     SET z.status = 'W_REALIZACJI'
   WHERE z.id_zamowienia = i_id_zamowienia;
  
  -- Krok 2: Utworzenie faktury
  INSERT INTO faktury (
    id_zamowienia,
    data_wystawienia,
    kwota
  ) VALUES (
    i_id_zamowienia,
    SYSDATE,
    (SELECT z.suma_zamowienia 
       FROM zamowienia z 
      WHERE z.id_zamowienia = i_id_zamowienia)
  ) RETURNING id_faktury INTO l_id_faktury;
  
  -- Krok 3: Utworzenie zlecenia wysyłki
  INSERT INTO wysylki (
    id_zamowienia,
    id_faktury,
    data_utworzenia,
    status
  ) VALUES (
    i_id_zamowienia,
    l_id_faktury,
    SYSDATE,
    'OCZEKUJE'
  ) RETURNING id_wysylki INTO l_id_wysylki;
  
  -- Krok 4: Aktualizacja stanów magazynowych
  UPDATE stany_magazynowe sm
     SET ilosc = ilosc - pz.ilosc
    FROM pozycje_zamowienia pz
   WHERE sm.id_produktu = pz.id_produktu
     AND pz.id_zamowienia = i_id_zamowienia;
  
  -- Wszystko OK - zatwierdzamy całą transakcję biznesową
  COMMIT;
  
  -- Logowanie sukcesu (w osobnej transakcji autonomicznej - patrz 7.8)
  log_operacja('PRZETWORZENIE_ZAMOWIENIA', i_id_zamowienia, 'SUCCESS');
  
EXCEPTION
  WHEN OTHERS THEN
    -- Coś poszło nie tak - wycofujemy całą operację
    ROLLBACK TO przed_przetworzeniem;
    
    -- Logowanie błędu
    log_operacja('PRZETWORZENIE_ZAMOWIENIA', i_id_zamowienia, 
          'ERROR: ' || SQLERRM);
    
    RAISE;
END przetworz_zamowienie;
/
```

**Wyjaśnienie:** Transakcja powinna odpowiadać jednej kompletnej operacji biznesowej. Wszystkie powiązane zmiany w bazie powinny być zatwierdzone razem (atomowość). COMMIT po każdej operacji prowadzi do niespójności danych w przypadku błędu. Używanie SAVEPOINT pozwala na częściowe wycofanie bez utraty całej transakcji.

### 7.8 Autonomous transactions

**Założenie:** Używaj autonomous transactions dla operacji logowania, które powinny być zachowane niezależnie od głównej transakcji.

**Przykład:**
```sql
-- ✓ POPRAWNIE - logowanie w transakcji autonomicznej
CREATE OR REPLACE PROCEDURE log_operacja (
  i_nazwa_operacji IN VARCHAR2,
  i_id_obiektu   IN NUMBER,
  i_status     IN VARCHAR2,
  i_komunikat  IN VARCHAR2 DEFAULT NULL
) IS
  PRAGMA AUTONOMOUS_TRANSACTION; -- Kluczowa dyrektywa
BEGIN
  INSERT INTO logi_aplikacji (
    id_logu,
    data_operacji,
    nazwa_operacji,
    id_obiektu,
    status,
    komunikat,
    uzytkownik,
    sesja
  ) VALUES (
    logi_id_seq.NEXTVAL,
    SYSTIMESTAMP,
    i_nazwa_operacji,
    i_id_obiektu,
    i_status,
    i_komunikat,
    USER,
    SYS_CONTEXT('USERENV', 'SESSIONID')
  );
  
  -- Transakcja autonomiczna MUSI mieć własny COMMIT lub ROLLBACK
  COMMIT;
  
EXCEPTION
  WHEN OTHERS THEN
    -- Nawet jeśli logowanie się nie powiedzie, nie przerywamy głównej operacji
    ROLLBACK;
    -- Możemy spróbować zapisać do pliku lub DBMS_OUTPUT
    DBMS_OUTPUT.PUT_LINE('Błąd logowania: ' || SQLERRM);
END log_operacja;
/

-- Przykład użycia
CREATE OR REPLACE PROCEDURE usun_pracownika (
  i_id_pracownika IN NUMBER
) IS
BEGIN
  -- Logujemy początek operacji (zostanie zachowane nawet przy ROLLBACK)
  log_operacja('USUWANIE_PRACOWNIKA', i_id_pracownika, 'START');
  
  -- Główna operacja
  DELETE FROM pracownicy p WHERE p.id_pracownika = i_id_pracownika;
  
  IF SQL%ROWCOUNT = 0 THEN
    log_operacja('USUWANIE_PRACOWNIKA', i_id_pracownika, 'BRAK_PRACOWNIKA');
    RAISE_APPLICATION_ERROR(-20001, 'Pracownik nie istnieje');
  END IF;
  
  -- Logujemy sukces
  log_operacja('USUWANIE_PRACOWNIKA', i_id_pracownika, 'SUCCESS');
  
  COMMIT;
  
EXCEPTION
  WHEN OTHERS THEN
    -- Logujemy błąd (log zostanie zachowany pomimo ROLLBACK)
    log_operacja('USUWANIE_PRACOWNIKA', i_id_pracownika, 'ERROR', SQLERRM);
    ROLLBACK;
    RAISE;
END usun_pracownika;
/
```

**Wyjaśnienie:** Autonomous transactions wykonują się w oddzielnej, niezależnej transakcji. Są idealne do logowania i audytu, ponieważ logi pozostają w bazie nawet gdy główna transakcja zostanie wycofana. To pozwala śledzić nieudane próby operacji, co jest kluczowe dla debugowania i bezpieczeństwa. UWAGA: Każda autonomous transaction MUSI kończyć się COMMIT lub ROLLBACK.

---

## Podsumowanie

Przestrzeganie przedstawionych praktyk formatowania kodu PL/SQL przynosi następujące korzyści:

1. **Czytelność** - Kod jest łatwiejszy do zrozumienia dla całego zespołu
2. **Utrzymywalność** - Modyfikacje i rozszerzenia są szybsze i bezpieczniejsze
3. **Debugowanie** - Błędy są łatwiejsze do zlokalizowania i naprawienia
4. **Współpraca** - Spójny styl ułatwia code review i pracę zespołową
5. **Dokumentacja** - Dobrze sformatowany kod z komentarzami jest samodokumentujący się
6. **Wydajność** - Stosowanie bulk operations i właściwego zarządzania transakcjami poprawia performance

### Zalecane narzędzia

Dla automatyzacji formatowania kodu PL/SQL zaleca się użycie:

- **Oracle SQL Developer** - wbudowany formatter (Ctrl+F7)
- **PL/SQL Developer** - narzędzie Beautifier
- **SQLcl** - narzędzie command-line z formatowaniem
- **PLDoc** - generator dokumentacji z kodu PL/SQL

### Dodatkowe zasoby

- Oracle Database PL/SQL Language Reference
- Oracle Database Development Guide
- "Oracle PL/SQL Best Practices" - Steven Feuerstein
- Oracle Live SQL - przykłady i tutoriale

---

**Autor:** GitHub Copilot  
**Data utworzenia:** 7 listopada 2025  
**Ostatnia aktualizacja:** 7 listopada 2025  
**Wersja dokumentu:** 2.3

### Historia zmian:
- **v1.0** (2025-11-07): Wersja początkowa z podstawowymi regułami formatowania
- **v2.0** (2025-11-07): Dodanie reguł nazewnictwa obiektów bazodanowych, uspójnienie sufiksów
- **v2.1** (2025-11-07): Zmiana standardu wcięć z 4 na 2 spacje, formatowanie INSERT-ów
- **v2.2** (2025-11-07): Usunięcie wszystkich sekcji "NIEPOPRAWNIE" - pozostawienie tylko dobrych praktyk
- **v2.3** (2025-11-07): Implementacja standardowych konwencji nazewniczych PL/SQL
