# Cel projektu
Celem projektu jest symulacja systemu obsługi współdzielonej bazy danych. W systemie tym spośród wszystkich węzłów mających dostęp do bazy danych tylko węzeł koordynatora wybranego przez algorytm Bully będzie miał możliwość zapisu informacji, 
natomiast każdy węzeł będzie miał możliwość odczytu informacji. System zademonstruje odporność na awarie, pokaże jak po utracie lidera zapisy są wstrzymywane i jak węzły wybierają nowego koordynatora. Użytkownikami systemu będą administratorzy (wstrzykujący błędy) oraz zwykli klienci (wprowadzający dane do bazy).

# Wykorzystane narzędzia
  * Język programowania (backend) - Python
  * Język programowania (frontend) - JavaScript
  * Użyte biblioteki/frameworki: FastAPI, React, Vite
    
  * Serwer baz danych - PostgreSQL
  * Broker wiadomości - RabbitMQ


# Architektura systemu
Projekt składa się 3 głównych elementów:
1. Serwera głównego (backend) - obsługuje on autoryzację, autentykację użytkowników, przechowywanie statusu wątków.
    
2. Serwera publicznego (frontend) - obsługuje interfejs graficzny i bezpośrednią komunikację z wątkami.
   
3. Wątek - obsługuje zapisywanie, odczytowanie informacji z bazy danych.

<img width="801" height="462" alt="obraz" src="https://github.com/user-attachments/assets/eee9b208-595c-4f7a-ad78-81b2c3c21e62" />

gdzie:
* Gniazda oznaczono kolorem - zielonym.

* Połączenia z bazą danych, kolorem - niebieskim.

* Zapytania API, kolorem - zółtym.

* Komunikację za pomocą brokera wiadomości, kolorem - czarnym

# Sposób komunikacji wewnątrz systemu
Komunikację między wątkami zapewnia RabbitMQ, tą ścieżką przesyłane są komunikaty algorytmu Bully oraz dane do zapisu lub usunięcia z bazy danych wątków. Każdy z wątków ma bezpośrednie połączenie z głównym serwerem, 
służy ono jedynie do przesyłania aktualizacji swojego statusu. Główny serwer zapewnia między innymi przesyłanie użytkownikowi zmian statusu wątków, uruchamianie ich oraz wyłączanie. 
Użytkownik znając listę wątków może wybrać do którego bezpośrednio się połączy. Taki wątek zaakceptuje połączenie oraz dokona autentykacji użytkownika za pomocą serwera głównego.

# Wstrzykiwanie błędów
System posiada możliwość wprowadzenie trzech różnych błędów:
* Zatrzymanie pracy dowolnego węzła.
* Węzeł nie będący liderem, wysyła komunikaty informujące inne wątki o przejęciu roli lidera.
* Węzeł nie będący liderem, zaczyna wysyłać dużą liczę wiadomości w krótkim czasie do wszystkich wątków.

# Bezpieczeństwo
Aplikacja została zabezpieczona przed następującymi atakami:
* CORS - dostęp do serwera backend został umożliwiony tylko serwerowi publicznemu (frontend).
* XSS - kod aplikacji został stworzony w sposób, uniemożliwiający wykonanie podatności. Serwer publiczny nie manipuluję bezpośrednio obiektem DOM.
* SSQL injection - do komunikacji z bazą danych użyta została biblioteka SQLModel która używa bezpiecznych parametryzowanych zapytań. Poniższy przykład przedstawia sposób pobierania użytkownika z bazy danych do logowania.
* CSRF - uwierzytelnianie użytkownika odbywa się poprzez dwa tokeny: dostępowy oraz odświeżania. Token odświeżania przechowywany jest w ciasteczkach przeglądarki z flagmi ‘HTTPOnly=True‘ oraz ‘SameSite=Lax‘. Token dostępowy przechowywany jest w zmiennej w pamięci, dzięki czemu atakujący nie ma do niego dostępu.
* Limit rządań - wprowadzony został limit 10 żądań na minutę dla punktów końcowych logowania, rejestracji oraz TOTP.
* Uwierzytelnianie wieloskładnikowe - użytkownicy mają możliwość dodatkowego uwierzytelniania za pomocą algorytmu TOTP. Realizowane jest to za pomocą biblioteki ‘pyotp‘
* Przechowywanie haseł - hasła w bazie danych przechowywane są w postaci zakodowanej przez algorytm “Argon2“

# Interfejs graficzny
<img width="1121" height="780" alt="obraz" src="https://github.com/user-attachments/assets/518b3dd4-0d7a-4c50-b9b8-9100618d49ff" />


# Testy jednostkowe i analiza statyczna
Testy jednostkowe zostały stworzone przy użyciu frameworka ‘pytest‘. Analiza statyczna została wykonana przy pomocy narzędzia SonarQube, poprzez ciągłą integrację w GitLab CI/CD.
<img width="1137" height="556" alt="obraz" src="https://github.com/user-attachments/assets/a24f8c1b-697b-40a6-af07-81cea915fdd6" />
