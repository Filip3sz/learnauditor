<p>
  <b>DEMO</b>
  <a href="https://filip3sz.github.io/learnauditor/learnauditor.html">filip3sz.github.io/learnauditor/learnauditor.html ↗</a>
</p>

<h1>LEARNAUDITOR - AI-ASSISTED EXAM PLATFORM</h1>
<p>
  <b>LearnAuditor</b> is a web application for running exams with a local AI grader. Instructors sign in with a one-time email code, create exams with open-ended questions or programming exercises, and review answers and scores in the dashboard. Students join in their browser with an exam code and a nickname. They receive feedback based on the instructor’s grading criteria. The application runs the grading model locally through Ollama.
</p>

<h2>System Capabilities</h2>
<p>
  <b>Passwordless Instructor Access & Plans:</b> Instructors register and sign in with a one-time email code. Accounts support <code>Standard</code>, <code>Plus</code>, and <code>Premium</code> plans with weekly exam quotas and feature tiers.<br>
  <b>Instructor Dashboard & DocsHelper Import:</b> Create, edit, search, and reorder exam tasks with grading criteria and reference answers, or import ready-made exam drafts generated from notes in DocsHelper.<br>
  <b>Student Exam View, Autosave & Countdown:</b> Join with a six-character code, nickname, and optional password. Supports live draft autosave, a 10-minute session resume window, single-tab lock, and optional countdown timers that auto-submit saved drafts.<br>
  <b>Single & Multi-Agent AI Grading:</b> Evaluates open-ended text or code (0–100% plus feedback) via <code>qwen3.5:9b</code>, or combines three models (<code>qwen3.5:9b</code>, <code>deepseek-r1:14b</code>, <code>llama3.1:8b</code>) using the median score.<br>
  <b>Authorship & Vibe-Coding Flags:</b> Flags AI-like patterns in answers (with paste and focus-loss counts as context), assigning a provisional 0% until reviewed; skipped in AI-only mode.<br>
  <b>Results, Review & Final Report:</b> Instructors override scores with automatic exam recalculation and view per-task statistics; students get an AI summary and a 30-day shareable result link.
</p>

<h2>Tech Stack</h2>
<p>
  <b>Language & Framework:</b> Java 21, Spring Boot 3.3.0, Spring MVC, and Spring Data JPA<br>
  <b>AI Integration:</b> Spring AI 1.0.0-M1 with <code>ChatClient</code> and <code>OllamaApi</code><br>
  <b>Local Model Runtime:</b> Ollama with <code>qwen3.5:9b</code> (plus <code>deepseek-r1:14b</code> and <code>llama3.1:8b</code> for multi-agent grading)<br>
  <b>Web Interface:</b> Server-rendered Thymeleaf templates, responsive Aurora Interface CSS, and JavaScript<br>
  <b>Authentication:</b> Spring Security, JJWT 0.13.0, and Spring Mail for one-time codes via SMTP<br>
  <b>Data Storage:</b> PostgreSQL 16 stores accounts, plans, exams, drafts, attempts, and answers. Redis 7 stores OTP codes, active JWTs, and temporary DocsHelper exam drafts.<br>
  <b>Data Exchange:</b> REST endpoints, centralized JSON error handling, and Jackson mapping<br>
  <b>Build & Local Services:</b> Maven, Lombok, and Docker Compose
</p>

<h2>Architecture & Technical Solutions</h2>
<p>
  <b>1. Exam, Plan, and Attempt Model</b><br>
  Spring Data JPA maps accounts, plans, exams, options, and ordered tasks to PostgreSQL with pessimistic locking for quota checks and concurrent edits. <code>UserTest</code> stores student attempts, saved drafts, deadlines, and final scores, while <code>ExamDeadlineScheduler</code> automatically finalizes expired timed attempts and <code>AccountService</code> expires paid plans on a schedule.
</p>
<p>
  <b>2. Spring MVC, JWT Sessions, and DocsHelper Bridge</b><br>
  Thymeleaf renders the sign-in view, dashboard, and student portal protected by CSRF and stateless JWT cookies validated against Redis (with Lua scripts verifying OTP attempts). <code>DocshelperService</code> stores incoming exam drafts from DocsHelper in Redis and imports them onto the instructor's account immediately or right after OTP sign-in via a short-lived cookie.
</p>
<p>
  <b>3. Single/Multi-Agent AI Grading and Reporting</b><br>
  <code>AiEvaluatorService</code> maps structured JSON model outputs into scores, feedback, and review flags, while <code>OllamaConfig</code> disables reasoning overhead (<code>think: false</code>) for supported models. In multi-agent mode, three models grade independently to compute the median score and majority review flag; upon completion, the primary model generates a final summary.
</p>
<p>
  <b>4. Task Numbering, Drafts, and Submission Locks</b><br>
  Tasks use sequential numbers within each exam (<code>/api/user/tasks/{id}/task/{taskNumber}/submit</code>). Signed JWT cookies bind a browser to an active attempt, Web Locks prevent opening the same attempt in multiple tabs, and database row locks prevent duplicate task submissions before invoking AI grading.
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
<pre><code>ollama pull qwen3.5:9b
deepseek-r1:14b
llama3.1:8b</code></pre>
<p>
  <b>3. Start PostgreSQL & Redis via Docker</b><br>
  Before run the PostgreSQL and Redis containers set <code>POSTGRES_USER</code> and <code>POSTGRES_PASSWORD</code> in your shell or local <code>.env</code> file before starting Docker.
  For an existing database volume, use the credentials already configured in PostgreSQL.
</p>
<pre><code>docker compose up -d</code></pre>
<p>
  <b>4. Run the Application</b><br>
  Provide <code>POSTGRES_USER</code> and <code>POSTGRES_PASSWORD</code> in the local <code>.env</code> file or as environment variables for Spring Boot. Then Build and start the Spring Boot backend using Maven:
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
  <b>Dostęp bez hasła i plany kont:</b> Rejestracja i logowanie jednorazowym kodem z e-maila. Konta obsługują plany <code>Standard</code>, <code>Plus</code> i <code>Premium</code> z tygodniowymi limitami testów oraz dostępem do rozszerzonych opcji.<br>
  <b>Panel prowadzącego i import z DocsHelpera:</b> Tworzenie, edycja, wyszukiwanie i zmiana kolejności zadań z kryteriami i kluczem odpowiedzi oraz import gotowych szkiców egzaminów wygenerowanych z notatek w DocsHelperze.<br>
  <b>Widok ucznia, autozapis i limit czasu:</b> Dołączanie kodem i nickiem (opcjonalnie z hasłem), automatyczny zapis szkiców, 10-minutowe okno wznowienia sesji, blokada wielu kart oraz odliczanie czasu z automatycznym oddaniem szkiców.<br>
  <b>Ocena pojedyncza i wieloagentowa AI:</b> Ocena tekstu lub kodu (0–100% i komentarz) przez <code>qwen3.5:9b</code> albo w trybie wieloagentowym przez trzy modele (<code>qwen3.5:9b</code>, <code>deepseek-r1:14b</code>, <code>llama3.1:8b</code>) z medianą wyniku.<br>
  <b>Oznaczanie odpowiedzi do weryfikacji:</b> Heurystyka wykrywa sygnały użycia AI w treści (z liczbą wklejeń i utrat fokusu jako kontekstem) i nadaje tymczasowe 0% do czasu weryfikacji; wyłączana w trybie „Tylko AI”.<br>
  <b>Wyniki, korekta i raport końcowy:</b> Prowadzący może zmieniać oceny cząstkowe z automatycznym przeliczeniem wyniku i przeglądać statystyki zadań, a uczeń otrzymuje podsumowanie AI i 30-dniowy link do wyniku.
</p>

<h2>Stos Technologiczny</h2>
<p>
  <b>Język i framework:</b> Java 21, Spring Boot 3.3.0, Spring MVC i Spring Data JPA<br>
  <b>Integracja z AI:</b> Spring AI 1.0.0-M1 z <code>ChatClient</code> i <code>OllamaApi</code><br>
  <b>Lokalne uruchamianie modeli:</b> Ollama z <code>qwen3.5:9b</code> (oraz <code>deepseek-r1:14b</code> i <code>llama3.1:8b</code> w trybie wieloagentowym)<br>
  <b>Interfejs webowy:</b> Szablony Thymeleaf renderowane po stronie serwera, responsywny motyw Aurora Interface w CSS i JavaScript<br>
  <b>Uwierzytelnianie:</b> Spring Security, JJWT 0.13.0 i Spring Mail do wysyłki jednorazowych kodów przez SMTP<br>
  <b>Przechowywanie danych:</b> PostgreSQL 16 przechowuje konta, plany, egzaminy, szkice, podejścia i odpowiedzi. Redis 7 przechowuje kody OTP, aktywne tokeny JWT i tymczasowe szkice z DocsHelpera.<br>
  <b>Wymiana danych:</b> Endpointy REST, centralna obsługa błędów JSON i mapowanie Jacksonem<br>
  <b>Budowanie i usługi lokalne:</b> Maven, Lombok i Docker Compose
</p>

<h2>Architektura i Rozwiązania Techniczne</h2>
<p>
  <b>1. Model egzaminu, planów i podejść</b><br>
  Spring Data JPA mapuje konta, plany, egzaminy, opcje i uporządkowane zadania w PostgreSQL z blokadami pesymistycznymi przy limitach i edycji. <code>UserTest</code> zapisuje podejścia uczniów, szkice odpowiedzi, terminy zakończenia i wyniki, <code>ExamDeadlineScheduler</code> automatycznie zamyka przeterminowane podejścia, a <code>AccountService</code> cyklicznie wygasza zakończone plany płatne.
</p>
<p>
  <b>2. Widoki MVC, sesje JWT i most z DocsHelperem</b><br>
  Thymeleaf renderuje logowanie, panel prowadzącego i portal ucznia chronione przez CSRF i bezstanowe ciasteczka JWT weryfikowane w Redisie (ze skryptami Lua dla OTP). <code>DocshelperService</code> przyjmuje szkice egzaminów z DocsHelpera do Redisa i tworzy z nich testy od razu lub automatycznie po zalogowaniu OTP dzięki krótkotrwałemu ciasteczku.
</p>
<p>
  <b>3. Ocena pojedyncza/wieloagentowa AI i raport końcowy</b><br>
  <code>AiEvaluatorService</code> mapuje odpowiedzi JSON modelu na punktację, komentarz i flagę weryfikacji, a <code>OllamaConfig</code> wyłącza narzut trybu myślenia (<code>think: false</code>). W trybie wieloagentowym trzy modele oceniają pracę niezależnie, wyliczając medianę punktów i większościową flagę weryfikacji, a na koniec główny model generuje podsumowanie egzaminu.
</p>
<p>
  <b>4. Numeracja zadań, szkice i kontrola zgłoszeń</b><br>
  Zadania mają lokalną numerację w obrębie egzaminu (<code>/api/user/tasks/{id}/task/{taskNumber}/submit</code>). Podpisane ciasteczka JWT wiążą przeglądarkę z podejściem, Web Locks blokuje otwarcie tego samego podejścia w wielu kartach, a blokada rekordu w bazie zapobiega duplikatom przed zleceniem oceny AI.
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
<pre><code>ollama pull qwen3.5:9b
deepseek-r1:14b
llama3.1:8b</code></pre>
<p>
  <b>3. Uruchomienie PostgreSQL i Redisa w Dockerze</b><br>
  Zanim uruchomisz kontenery PostgreSQL i Redis ustaw <code>POSTGRES_USER</code> oraz <code>POSTGRES_PASSWORD</code> w terminalu lub lokalnym pliku <code>.env</code>.
  Przy istniejącym wolumenie użyj danych konta skonfigurowanego już w PostgreSQL.
</p>
<pre><code>docker compose up -d</code></pre>
<p>
  <b>4. Uruchomienie Aplikacji</b><br>
  Ustaw <code>POSTGRES_USER</code> i <code>POSTGRES_PASSWORD</code> w lokalnym pliku <code>.env</code> lub w środowisku aplikacji. Następnie skompiluj i uruchom aplikację Spring Boot przy użyciu Mavena:
</p>
<pre><code>mvn clean spring-boot:run</code></pre>
<p>
  Po pomyślnym uruchomieniu przejdź pod adres <code>http://localhost:8080/dashboard</code> w przeglądarce, aby otworzyć panel aplikacji LearnAuditor. Platforma ucznia znajduje się pod <code>/exam</code>.
</p>