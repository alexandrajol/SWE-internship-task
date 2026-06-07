# NOTES — Alexandra Joldea

## 1. Bug-urile găsite

Pentru fiecare bug, scrie 2-3 propoziții:

### Bug #1
- **Unde era:** 
În fișierul app\main.py
Linia 32: `@app.post("/events", response_model=Event)`
- **Cum l-am găsit:** 
Am observat că testul `test_create_event_returns_201` pică (`assert 200 == 201`).
- **Cum l-am fixat:**
M-am uitat la diferențele dintre `create_user` și `create_event` și am observat că lipsește parametrul `status_code=201` din decoratorul celei din urmă (așa că l-am adăugat acolo). Lipsa lui era o problemă pentru că mesajul implicit de la FastAPI e `200 OK` pentru POST.

### Bug #2
- **Unde era:**
În fișierul app\storage.py
Linia 51: `return all_events[offset + 1: offset + 1 + limit]`
- **Cum l-am găsit:**
Citind codul, am observat că se adaugă 1 la offset. În plus, testul `test_list_events_includes_created_items` pica, și se vedea foarte clar că assertul așteaptă 5 elemente și primește doar 4.
- **Cum l-am fixat:**
Am eliminat `+ 1` din ambele expresii din slicing -> `all_events[offset : offset + limit]`.

### Bug #3
- **Unde era:**
În fișierul app\storage.py
Linia 50: `all_events = list(self._events.values())`
Și linia 55: `if event is None:`
- **Cum l-am găsit:**
Mai multe teste picau (`test_pagination_after_delete_stays_consistent`, `test_list_events_hides_soft_deleted_items`) de unde mi-am dat seama că list_events nu lua în considerare dacă evenimentul a fost ”șters”. Faptul că testul `test_delete_same_event_twice_changes_response` pică, deoarece assertul așteaptă `404` și primește `204`, semnalează că funcția de ștergere a evenimentului nu verifică dacă a fost deja șters.
- **Cum l-am fixat:** 
M-am folosit de atributul `deleted_at` pentru a filtra lista returnată din `list_events` -> `list(event for event in self._events.values() if event.deleted_at is None)`. Pentru `soft_delete_event`, am adăugat o condiție pentru gestionarea cazului de ștergere multiplă -> dacă `event.deleted_at is not None`, returnează `None` (API-ul va returna `404`).

---

## 2. Endpoint-ul nou

- **Decizii de design:** 
  - Am adăugat funcția `get_user_events()` în storage.py care filtrează evenimentele după user_id și deleted_at.
  - Parametrul `since` e opțional (Query cu None default) și parsează string ISO 8601 (după standardele internașionale) în datetime. 
  - Rezultatul nu include evenimentul din exact acel timestamp.
- **Cazuri edge pe care le-ai acoperit:**
  - User inexistent: `404 Not Found`
  - Parametrul `since` invalid (format greșit): `400 Bad Request`
  - Parametrul `since` lipsește: returnează toate evenimentele user-ului
  - Evenimentele soft-deleted nu apar în rezultate
  - Evenimentele altor useri sunt filtrate (dacă se pune ca edge case)
- **Teste adăugate:**
  - `test_get_user_events_returns_only_user_events` - se returnează doar evenimentele user-ului respectiv, nu ale altor useri
  - `test_get_user_events_with_missing_user_returns_404` - user inexistent returnează `404`
  - `test_get_user_events_filters_by_since_parameter` - filtrare corectă cu parametrul since (timestamp ISO)
  - `test_get_user_events_hides_deleted_events` - evenimentele șterse nu apar în listă
  - `test_get_user_events_with_invalid_date_format_returns_400` - format invalid de dată returnează `400 Bad Request`


---

## 3. Folosirea AI-ului

Fii cinstit. Nu pierzi puncte dacă spui adevărul, dimpotrivă.

- **Ce ai folosit:** 
  - Claude Code integrat în IDE (VS Code)
- **Prompturi reprezentative folosite:** 
  - "what is the point of EventCreate and UserCreate models?" - design patterns în API-uri (input vs output models) cu care nu sunt foarte familiară
 - întrebări despre bug-uri: "shouldn't this check if the user already exists?" -> am discutat dacă e bug sau nu că nu există un edge case pentru adăugarea unui user deja existent:)
- **Unde te-a ajutat cel mai mult:**
  - Explicații conceptuale
  - Înțelegerea flow-ului aplicației
  - Implementarea endpoint-ului nou
- **Unde te-a încurcat sau ți-a dat un răspuns greșit:** 
  - Uneori explicațiile nu erau suficient de tehnice: primeam explicații prea vagi, când pe mine nu mă interesa viziunea de ansamblu, ci detaliile de sintaxă
  - În rest a făcut o treabă bună :)
- **Cum ai verificat ce-a generat:**
  - Am rulat testele după fiecare fix 
  - Am citit și înțeles codul generat
  - Am comparat cu codul existent pentru consistență și ca un double-check
- **Anexă opțională — export chat:** (dacă vrei, poți adăuga un export de chat relevant)

---

## 4. Ce-ai face cu mai mult timp

- Aș adăuga validare cu email unic
- Aș înlocui storage-ul in-memory cu un database, cum ar fi PostgreSQL pentru scalabilitate
- Aș adăuga paginare pentru endpoint-ul nou, să fie consistent cu restul API-ului
- Aș implementa cleanup automat (evenimentele mai vechi de 365 de zile să fie șterse)

---

