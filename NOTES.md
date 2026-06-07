# NOTES — [Numele tău]

Copiază acest fișier ca `NOTES.md` și completează-l.

Vrem să fie scurt — maxim 1 pagină. Mai mult contează claritatea decât lungimea.

---

## 1. Bug-urile găsite

Pentru fiecare bug, scrie 2-3 propoziții:

### Bug #1
- **Unde era:** 
În fișierul app\main.py
Linia 32: @app.post("/events", response_model=Event)
- **Cum l-am găsit:** 
Am observat că testul test_create_event_returns_201 pică (assert 200 == 201).
- **Cum l-am fixat:**
M-am uitat la diferențele dintre create_user și create_event și am observat că lipsește parametrul ”status_code=201” din decoratorul celei din urmă (așa că l-am adăugat acolo). Asta e o problemă pentru că mesajul implicit de la FastAPI e 200 OK, deoarece nu are de unde să știe că a fost creat un obiect nou.

### Bug #2
- **Unde era:**
În fișierul app\storage.py
Linia 51: return all_events[offset + 1: offset + 1 + limit]
- **Cum l-am găsit:**
Citind codul, am observat că se adaugă 1 la offset. În plus, testul test_list_events_includes_created_items pica, și se vedea foarte clar că assertul așteaptă 5 elemente și primește doar 4.
- **Cum l-am fixat:**
Am eliminat ”+ 1” din ambele expresii din slicing -> all_events[offset : offset + limit].

### Bug #3
- **Unde era:**
În fișierul app\storage.py
Linia 50: all_events = list(self._events.values())
Și linia 55: if event is None:
- **Cum l-am găsit:**
Mai multe teste picau (test_pagination_after_delete_stays_consistent, test_list_events_hides_soft_deleted_items) de unde mi-am dat seama că list_events nu lua în considerare dacă evenimentul a fost ”șters”. Faptul că testul test_delete_same_event_twice_changes_response pică, deoarece assertul așteaptă 404 și primește 204, semnalează că există o problemă asemănătoare la funcția de stergere a elementului, unde am realizat că nu este tratat deloc cazul de ștergere multiplă a unui eveniment, ceea ce e esențial în acest tip de aplicație în care elementele șterse rămân în același dicționar cu cele neșterse.
- **Cum l-am fixat:** 
M-am folosit de atributul ”deleted_at” pentru a filtra lista returnată -> list(event for event in self._events.values() if event.deleted_at is None). Pentru soft_delete_event, am adăugat o condiție pentru gestionarea cazului de ștergere multiplă -> dacă event.deleted_at is not None, returnează None (API-ul va returna 404).

---

## 2. Endpoint-ul nou

- **Decizii de design:** (ce-ai considerat? ce ai ales și de ce?)
- **Cazuri edge pe care le-ai acoperit:**
- **Teste adăugate:** (ce verifică fiecare)

---

## 3. Folosirea AI-ului

Fii cinstit. Nu pierzi puncte dacă spui adevărul, dimpotrivă.

- **Ce ai folosit:** (ChatGPT / Cursor / Copilot / altele)
- **Prompturi reprezentative folosite:** (scrie prompturile pe care le consideri relevante + context scurt: la ce te-au ajutat)
- **Unde te-a ajutat cel mai mult:**
- **Unde te-a încurcat sau ți-a dat un răspuns greșit:** (foarte interesant pentru noi!)
- **Cum ai verificat ce-a generat:**
- **Anexă opțională — export chat:** (dacă vrei, poți adăuga un export de chat relevant)

---

## 4. Ce-ai face cu mai mult timp

(Lista scurtă, 3-5 puncte. Arată-ne că ai văzut limitele actuale.)

---

## 5. Întrebări / observații

(Orice nu a fost clar, orice ai vrea să discuți cu noi.)
