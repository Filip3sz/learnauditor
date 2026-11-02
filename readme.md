<h1>LEARNAUDITOR - AI-ASSISTED EXAM PLATFORM</h1>
<p>
  <b>LearnAuditor</b> is a web application for running exams with a local AI grader. Instructors sign in with a one-time email code, create exams with open-ended questions or programming exercises, and review answers and scores in the dashboard. Students join in their browser with an exam code and a nickname. They receive feedback based on the instructor’s grading criteria. The application runs the grading model locally through Ollama.
</p>

<h2>System Capabilities</h2>
<p>
  <b>Passwordless Instructor Access:</b> With SMTP configured, instructors register and sign in with a one-time email code. Registration creates the account after the email address is verified.<br>
  <b>Instructor Dashboard:</b> Create and edit exams, add study material, and change the order of tasks. Each task can include its own grading criteria and reference answer. Instructors can search the exam list and preview an exam from the dashboard. The exam and participant lists show five entries per page.<br>
  <b>Student Exam View:</b> Join with a six-character code and a nickname. Exams can require a password at check-in, and each participant receives an avatar. The prompt and editor sit side by side on wide screens and stack on phones. Long prompts can scroll inside their panel.<br>
  <b>Grading Against Criteria:</b> The local model evaluates text or code using the task description, grading criteria, and reference answer. Each submission receives a score from 0 to 100% and feedback. The student can navigate the exam while the response is being graded.<br>
  <b>Submissions Flagged for Review:</b> A heuristic can flag potential AI use for instructor review. The browser reports paste events as additional context, while the flag depends on signals in the answer itself. A flagged answer receives a provisional score of 0% pending review.<br>
  <b>Instructor Review and AI-Only Mode:</b> Instructors can adjust individual scores. Changing a task score recalculates the result of a completed exam. In AI-only mode, the system uses the model's scores directly and skips the authorship checks.<br>
  <b>Student Results and Statistics:</b> Browse participants, read their answers and AI feedback, and see which submissions need review. The dashboard shows completion counts and average scores for the whole exam or a selected task.<br>
  <b>Final Exam Report:</b> After submission, the student sees the overall score, time taken, and an AI-generated summary. The score is the average across all exam tasks, with unanswered tasks counting as 0%.
</p>

<h2>Tech Stack</h2>
<p>
  <b>Language & Framework:</b> Java 21, Spring Boot 3.3.0, Spring MVC, and Spring Data JPA<br>
  <b>AI Integration:</b> Spring AI 1.0.0-M1 with <code>OllamaChatModel</code><br>
  <b>Local Model Runtime:</b> Ollama with <code>qwen3.5:9b</code> selected through <code>application.properties</code><br>
  <b>Web Interface:</b> Server-rendered Thymeleaf templates, responsive Aurora Interface CSS, and JavaScript<br>
  <b>Authentication:</b> Spring Security, JJWT 0.13.0, and Spring Mail for one-time codes via SMTP<br>
  <b>Data Storage:</b> PostgreSQL stores accounts, exams, options, attempts, and answers. Redis stores short-lived login codes and active JWTs.<br>
  <b>Data Exchange:</b> REST endpoints and JSON mapped with Jackson<br>
  <b>Build & Local Services:</b> Maven, Lombok, and Docker Compose
</p>

<h2>Architecture & Technical Solutions</h2>
<p>
  <b>1. Exam, Attempt, and Submission Model</b><br>
  Spring Data JPA maps exams, options, and ordered tasks to PostgreSQL. <code>Account</code> stores the instructor’s UUID and email. A <code>UserTest</code> represents a student's attempt and records the nickname, avatar, timestamps, and final result. Each <code>UserTask</code> links an answer to the attempt and the original task, along with its score, feedback, and review status. Deleting an exam also removes its related tasks and submissions.
</p>

<p>
  <b>2. Spring MVC and Thymeleaf Views</b><br>
  Spring MVC renders the sign-in page, instructor dashboard, and student portal with Thymeleaf. Sign-in, check-in, and dashboard edits use HTML forms. JavaScript handles navigation, search, and pagination, and sends grading requests asynchronously. After OTP verification, Spring Security validates the JWT carried in an <code>HttpOnly</code> cookie against the active token stored in Redis. Signing out removes that token, and CSRF protection applies to form submissions.
</p>

<p>
  <b>3. AI Responses and Final Reporting</b><br>
  <code>AiEvaluatorService</code> uses Spring AI to map the model's response to an object containing a score, feedback, and a review flag. The backend checks the score before saving the answer. When the exam is submitted, the model receives a JSON summary of the tasks and answers to prepare final feedback. If summary generation fails, the application still saves the exam result.
</p>

<p>
  <b>4. Task Numbering and Submission Checks</b><br>
  Each task has a database ID and a number within its exam. The endpoint <code>/api/user/tasks/{id}/task/{taskNumber}/submit</code> resolves the task from the exam ID and its local number. Nicknames are checked within each exam. A database lock coordinates updates to an existing attempt, and the backend checks for duplicate submissions before requesting an AI grade.
</p>

<h2>Installation & Configuration</h2>
<p>
  <b>1. Prerequisites</b><br>
  Ensure you have <b>JDK 21</b>, <b>Maven 3.x</b>, <b>Ollama</b>, and <b>Docker Desktop</b> installed on your environment.
</p>
<p>
  <b>2. Setup Local Model via Ollama</b><br>
  Pull the required LLM chat in your terminal:
</p>
<pre><code>ollama pull qwen3.5:9b</code></pre>
<p>
  <b>3. Start PostgreSQL & Redis via Docker</b><br>
  Run the PostgreSQL and Redis containers:
</p>
<pre><code>docker compose up -d</code></pre>
<p>
  <b>4. Run the Application</b><br>
  Build and start the Spring Boot backend using Maven:
</p>
<pre><code>mvn clean spring-boot:run</code></pre>
<p>
  Once started, navigate to <code>http://localhost:8080</code> in your browser to access the LearnAuditor dashboard. The student platform will be available at <code>/exam</code>.
</p>

<hr>

<h1>LEARNAUDITOR - PLATFORMA EGZAMINACYJNA WSPOMAGANA PRZEZ AI</h1>
<p>
  <b>LearnAuditor</b> to aplikacja webowa do przeprowadzania egzaminów ocenianych przez lokalny model AI. Prowadzący loguje się kodem z maila, tworzy egzaminy z pytaniami otwartymi lub zadaniami programistycznymi i przegląda odpowiedzi oraz wyniki w panelu. Uczniowie dołączają w przeglądarce za pomocą kodu egzaminu i nicku. Otrzymują komentarze oparte na kryteriach ustalonych przez prowadzącego. Model oceniający działa lokalnie w Ollamie.
</p>

<h2>Możliwości Systemu</h2>
<p>
  <b>Dostęp prowadzącego bez hasła:</b> Po skonfigurowaniu SMTP prowadzący rejestruje się i loguje jednorazowym kodem z e-maila. Konto powstaje po potwierdzeniu adresu.<br>
  <b>Panel prowadzącego:</b> Twórz i edytuj egzaminy, dodawaj materiały do przeczytania i zmieniaj kolejność zadań. Każde zadanie może mieć własne kryteria oceniania i odpowiedź wzorcową. W panelu można wyszukiwać egzaminy i otwierać ich podgląd. Listy egzaminów i uczestników pokazują po pięć pozycji na stronie.<br>
  <b>Widok ucznia:</b> Uczniowie dołączają za pomocą sześcioznakowego kodu i nicku. Egzamin może wymagać hasła przy dołączeniu, a każdy uczestnik otrzymuje awatar. Na szerokim ekranie treść zadania i edytor są obok siebie, a na telefonie jeden pod drugim. Długie polecenie można przewijać w jego panelu.<br>
  <b>Ocena AI według kryteriów:</b> Model analizuje tekst lub kod na podstawie polecenia, wymagań i rozwiązania wzorcowego. Każda przesłana odpowiedź otrzymuje wynik od 0 do 100% oraz komentarz. Uczeń może przechodzić między zadaniami w trakcie oceniania.<br>
  <b>Oznaczanie odpowiedzi do weryfikacji:</b> Heurystyka wskazuje odpowiedzi, które mogą wymagać sprawdzenia pod kątem użycia AI. Liczba wklejeń z przeglądarki uzupełnia kontekst, a flaga zależy od sygnałów w treści odpowiedzi. Oznaczona praca otrzymuje tymczasowe 0% do czasu weryfikacji przez prowadzącego.<br>
  <b>Weryfikacja przez prowadzącego i tryb Tylko AI:</b> Prowadzący może zmienić ocenę zadania, co automatycznie przelicza wynik ukończonego egzaminu. W trybie „Tylko AI” obowiązuje punktacja modelu, a sprawdzanie sygnałów dotyczących autorstwa jest wyłączone.<br>
  <b>Wyniki uczniów i statystyki:</b> Przeglądaj listę uczestników, ich odpowiedzi, komentarze AI i oznaczenia do weryfikacji. Panel pokazuje liczbę uczestników, postęp i średnią ocen dla całego egzaminu albo wybranego zadania.<br>
  <b>Raport końcowy:</b> Po oddaniu egzaminu uczeń widzi wynik, czas trwania i podsumowanie przygotowane przez model. Wynik to średnia ocen ze wszystkich zadań. Zadania bez odpowiedzi liczą się jako 0%.
</p>

<h2>Stos Technologiczny</h2>
<p>
  <b>Język i framework:</b> Java 21, Spring Boot 3.3.0, Spring MVC i Spring Data JPA<br>
  <b>Integracja z AI:</b> Spring AI 1.0.0-M1 z <code>OllamaChatModel</code><br>
  <b>Lokalne uruchamianie modelu:</b> Ollama z <code>qwen3.5:9b</code> wybieranym w <code>application.properties</code><br>
  <b>Interfejs webowy:</b> Szablony Thymeleaf renderowane po stronie serwera, responsywny motyw Aurora Interface w CSS i JavaScript<br>
  <b>Uwierzytelnianie:</b> Spring Security, JJWT 0.13.0 i Spring Mail do wysyłki jednorazowych kodów przez SMTP<br>
  <b>Przechowywanie danych:</b> PostgreSQL przechowuje konta, egzaminy, ustawienia, podejścia i odpowiedzi. Redis przechowuje krótkotrwałe kody logowania i aktywne tokeny JWT.<br>
  <b>Wymiana danych:</b> Endpointy REST i obsługa JSON za pomocą Jacksona<br>
  <b>Budowanie i usługi lokalne:</b> Maven, Lombok i Docker Compose
</p>

<h2>Architektura i Rozwiązania Techniczne</h2>
<p>
  <b>1. Model egzaminu, podejścia i odpowiedzi</b><br>
  Spring Data JPA zapisuje w PostgreSQL egzaminy, ustawienia i uporządkowane zadania. <code>Account</code> przechowuje UUID i e-mail prowadzącego. <code>UserTest</code> reprezentuje podejście ucznia i przechowuje nick, awatar, czas rozpoczęcia i zakończenia oraz wynik końcowy. <code>UserTask</code> wiąże odpowiedź z podejściem i zadaniem oraz zapisuje ocenę, komentarz i status weryfikacji. Usunięcie egzaminu usuwa również jego zadania i powiązane zgłoszenia.
</p>

<p>
  <b>2. Widoki Spring MVC i Thymeleaf</b><br>
  Spring MVC renderuje przez Thymeleaf ekran logowania, panel prowadzącego i portal ucznia. Logowanie, dołączanie do egzaminu i edycja w panelu korzystają z formularzy HTML. JavaScript obsługuje nawigację, wyszukiwanie, paginację i asynchroniczną wysyłkę żądań do oceny. Po potwierdzeniu OTP Spring Security weryfikuje JWT z ciasteczka <code>HttpOnly</code> i sprawdza jego zgodność z aktywnym tokenem w Redisie. Wylogowanie usuwa aktywny token, a wysyłkę formularzy chroni CSRF.
</p>

<p>
  <b>3. Odpowiedzi modelu i raport końcowy</b><br>
  <code>AiEvaluatorService</code> korzysta ze Spring AI, aby odczytać odpowiedź modelu jako obiekt z punktacją, komentarzem i flagą weryfikacji. Backend sprawdza zakres punktacji przed zapisem odpowiedzi. Przy zakończeniu egzaminu model dostaje historię zadań i odpowiedzi w formacie JSON i przygotowuje końcowy komentarz. Gdy generowanie podsumowania się nie powiedzie, aplikacja nadal zapisuje wynik egzaminu.
</p>

<p>
  <b>4. Numeracja zadań i kontrola zgłoszeń</b><br>
  Zadanie ma własny identyfikator w bazie i numer w ramach egzaminu. Endpoint <code>/api/user/tasks/{id}/task/{taskNumber}/submit</code> znajduje zadanie na podstawie identyfikatora egzaminu i tego numeru. Nicki są sprawdzane w obrębie wybranego testu. Blokada rekordu koordynuje aktualizacje istniejącego podejścia, a przed zleceniem oceny backend sprawdza, czy uczeń nie przesłał już rozwiązania danego zadania.
</p>

<h2>Instalacja i Konfiguracja</h2>
<p>
  <b>1. Wymagania Wstępne</b><br>
  Upewnij się, że w Twoim środowisku deweloperskim zainstalowane są: <b>JDK 21</b>, <b>Maven</b>, silnik <b>Ollama</b> oraz <b>Docker Desktop</b>.
</p>
<p>
  <b>2. Pobranie Lokalnego Modelu Lokalnego w Ollama</b><br>
  Pobierz wymagany model dla czatu:
</p>
<pre><code>ollama pull qwen3.5:9b</code></pre>
<p>
  <b>3. Uruchomienie PostgreSQL i Redisa w Dockerze</b><br>
  Uruchom kontenery PostgreSQL i Redis:
</p>
<pre><code>docker compose up -d</code></pre>
<p>
  <b>4. Uruchomienie Aplikacji</b><br>
  Skompiluj i uruchom aplikację Spring Boot przy użyciu Mavena:
</p>
<pre><code>mvn clean spring-boot:run</code></pre>
<p>
  Po pomyślnym uruchomieniu przejdź pod adres <code>http://localhost:8080/dashboard</code> w przeglądarce, aby otworzyć panel aplikacji LearnAuditor. Platforma ucznia znajduje się pod <code>/exam</code>.
</p>