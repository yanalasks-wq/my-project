# my-project

# Мета: Знайомство з A01:2025 Broken Access Control

**Середовище:** Kali Linux, Docker engine, OWASP WebGoat container.

Для кращого розуміння потрібно пройти попередні кроки з мануала WebGoat.

В меню обираємо розділ **(A1) Broken Access Control** та підрозділ **Hijack a session**.

### Концепція
Розробники прикладного програмного забезпечення, які створюють власні ідентифікатори сесій (session IDs), часто забувають забезпечити рівень складності та рандомізації, необхідний для безпеки. Якщо специфічний ідентифікатор сесії користувача не є складним і випадковим, додаток стає надзвичайно вразливим до атак типу «brute force» (перебір) на сесії.

### Цілі
Отримати доступ до автентифікованої сесії, що належить іншому користувачу.

### Основні терміни:
* **Session ID** — ідентифікатор сесії.
* **Complexity and randomness** — складність та випадковість.
* **Brute force attacks** — атаки методом грубої сили (перебору).
* **Authenticated session** — автентифікована сесія (сеанс).

## Хід виконання роботи

Далі переходимо на другу вкладинку (червоний колір означає, що потрібно буде щось зробити і це буде перевірятись). У цьому уроці ми намагаємося передбачити значення `hijack_cookie`. Цей файл використовується для розрізнення автентифікованих та анонімних користувачів WebGoat.

<img width="974" height="586" alt="image" src="https://github.com/user-attachments/assets/258404e7-c665-4c7a-96db-1d7709cf9112" />

### 1. Дослідження трафіку за допомогою Burp Suite
* Запускаємо Burp Suite, переходимо на вкладку **Proxy**.
* Запускаємо браузер, в якому відкриваємо WebGoat, створюємо користувача та логінуємось (або одразу заходимо, якщо не видаляли контейнер) і переходимо на другий крок.
* Натискаємо **Access**, отримуємо помилку та повертаємось до Burp Suite у вкладинку **HTTP history**.
* Шукаємо відповідь `POST`, в якій міститься інформація про цю помилку (ключ — це зміст `hijack_cookie`, який нам і потрібно «передбачити»).

<img width="1579" height="673" alt="image" src="https://github.com/user-attachments/assets/d13b4935-2457-45de-951a-f1cf6d91f1c0" />

* Натискаємо правою кнопкою миші по запиту та відправляємо його до **Repeater**.

### 2. Аналіз значень cookie
* У Repeater робимо декілька повторів (у цьому прикладі вистачить 5+ запитів).
* Бачимо, як змінюється значення `hijack_cookie`, і копіюємо ці значення в текстовий редактор:

<img width="553" height="244" alt="image" src="https://github.com/user-attachments/assets/ca6cefbf-a8c5-4227-b1b3-1944e5632281" />

### 3. Intruder
## Далі в Intruder формуємо свій запит відповідним чином:

*Додаємо hijack_cookie=501047523917480945-1784919404592 , останні дві цифри будемо змінювати, такий собі лайтовий брутфорс

<img width="1561" height="580" alt="image" src="https://github.com/user-attachments/assets/a70e808a-159e-4dfc-918d-c9e87fb0beb9" />

*Запускаємо start attack та чекаємо результатів перебору, однак майже одразу отримуємо позитивний результат:

<img width="1389" height="612" alt="image" src="https://github.com/user-attachments/assets/92e9dbce-50d6-4684-8d3a-ff9bd83a5407" />

*Та відповідно в завданні вкладинка змінює колір з червоного на зелений:

<img width="1081" height="355" alt="image" src="https://github.com/user-attachments/assets/cc691810-40de-4f24-9ce0-e2e5c818451a" />

### Висновки

Під час виконання лабораторної роботи було практично досліджено вразливість A01:2025 – Broken Access Control на прикладі WebGoat. 

* За допомогою інструментів перехоплення та аналізу трафіку (Burp Suite Proxy та Repeater) було виявлено, що значення спеціального параметра `hijack_cookie` формується за прогнозованою логікою (комбінація ідентифікатора та часової мітки, що порушує вимоги щодо складності та випадковості сесій.
* Застосувавши модуль Intruder для швидкого підбору, вийшло перехопити автентифіковану сесію та отримати доступ до захищеного ресурсу іншого користувача.


### OWASP. A01. IDOR
### Прямі посилання на об'єкти (Direct Object References)

## Приклади
Приклади прямих посилань на об'єкти з використанням методу GET можуть виглядати приблизно так:

https://some.company.tld/dor?id=12345
https://some.company.tld/images?img=12345
https://some.company.tld/dor/12345

## Інші методи
Методи POST, PUT, DELETE та інші також можуть бути вразливими; вони відрізняються переважно самим методом та структурою корисного навантаження (payload).

## Небезпечні прямі посилання на об'єкти (IDOR)
Посилання вважаються небезпечними, якщо вони не обробляються належним чином, що дозволяє обійти авторизацію або розкрити приватні дані. Це може бути використано для виконання операцій або доступу до даних, до яких користувач не повинен мати доступу.

Припустімо, що ви як користувач відкриваєте свій профіль, і URL-адреса виглядає так: https://some.company.tld/app/user/23398
...і ви бачите там свій профіль. Що станеться, якщо ви перейдете за адресою: https://some.company.tld/app/user/23399
...або використаєте інше число в кінці? Якщо ви можете маніпулювати номером (ID користувача) і переглядати чужий профіль, то таке посилання на об'єкт є небезпечним.
Звісно, це можна перевірити або розширити за межі методів GET не лише для перегляду даних, але й для їхньої модифікації.
## Хід роботи
## Етап 1. Автентифікація та розвідка
Авторизація в застосунку: Виконано вхід до лабораторного середовища WebGoat під обліковим записом зі стандартними обліковими даними (tom / cat).
Аналіз трафіку: За допомогою інструмента Burp Suite (Proxy -> HTTP history) було проаналізовано «сирі» (raw) відповіді сервера під час взаємодії з особистим профілем користувача. Це дозволило виявити внутрішню структуру маршрутизації застосунку (RESTful-підхід) та ідентифікувати унікальний числовий ідентифікатор користувача (userId), який передається в шляху запиту.

<img width="689" height="301" alt="image" src="https://github.com/user-attachments/assets/64d6aaa2-e4a1-41d1-9258-a3580e566aba" />

Переглядаємо httphistory у burpsuit та знаходимо відмінності:
<img width="1156" height="463" alt="image" src="https://github.com/user-attachments/assets/eb474cf6-8dbb-4b70-98b1-46d4c4eafe68" />

Як результат - цей крок пройдено:

<img width="564" height="420" alt="image" src="https://github.com/user-attachments/assets/4b979897-9296-45dd-85bd-0c8267cc0d3f" />

## Етап 2. Гра з шаблонами та перегляд чужого профілю
Використовуючи перехоплення запитів (Intercept), було виявлено механізм прямого звернення до об'єктів за їхнім числовим ID.

Шляхом зміни параметра ідентифікатора у запиті за допомогою інструмента Intruder було знайдено ID іншого користувача.

<img width="861" height="348" alt="image" src="https://github.com/user-attachments/assets/1e1f4796-76ec-46fa-9377-0e8da823623b" />

<img width="1565" height="400" alt="image" src="https://github.com/user-attachments/assets/ba4b1080-1e2b-4820-853f-ac5e387ce131" />

<img width="1269" height="552" alt="image" src="https://github.com/user-attachments/assets/c01c1afe-398c-4888-9b0c-e3dbc6eb288a" />

## Етап 3. Експлуатація та модифікація даних (IDOR через PUT-запит)
Для підтвердження критичності вразливості було здійснено спробу зміни даних іншого користувача в обхід механізмів контролю доступу:
Запит було перенаправлено до вкладки Repeater.
HTTP-метод змінено з GET на PUT.
У тіло запиту (Payload) передано JSON-структуру з новими параметрами:

<img width="1416" height="585" alt="image" src="https://github.com/user-attachments/assets/dbbab5b3-2fca-48ea-9bd0-a0045d2b5489" />

<img width="1273" height="565" alt="image" src="https://github.com/user-attachments/assets/a1889146-e8b8-44dd-910a-b3375c494b72" />

## Висновок
Під час виконання лабораторної роботи було успішно досліджено вразливість типу IDOR. За допомогою Burp Suite було перехоплено HTTPзапит, змінивши метод на PUT і підставивши числовий ідентифікатор іншого користувача.


### OWASP. A03. IDOR

## Концепція
Інформація цього модуля призначена для розуміння, що таке Structured Query Language (SQL) і як ним можна маніпулювати для виконання завдань, які не були передбачені розробником.

## Цілі
Користувач отримає базове розуміння того:
як працює SQL
для чого він використовується

## Користувач також отримає базове розуміння:
що таке SQL-ін’єкція
як вона працює

## Користувач продемонструє знання щодо:
DML, DDL та DCL
рядкових SQL-ін’єкцій (String SQL injection)
числових SQL-ін’єкцій (Numeric SQL injection)
того, як SQL-ін’єкція порушує тріаду CIA (конфіденційність, цілісність, доступність)

## Що таке SQL?
SQL — це стандартизована (ANSI у 1986 році, ISO у 1987 році) мова програмування, яка використовується для керування реляційними базами даних та виконання різних операцій із даними в них.
База даних — це сукупність даних. Дані впорядковані у рядки, стовпці та таблиці, а також індексуються, щоб зробити пошук відповідної інформації ефективнішим.
Приклад SQL-таблиці, що містить дані про співробітників; назва таблиці — «employees»:
Таблиця Employees
userid	first_name	last_name	department	salary	auth_tan
32147	Paulina	Travers	Accounting	$46.000	P45JSI
89762	Tobi	Barnett	Development	$77.000	TA9LL1
96134	Bob	Franco	Marketing	$83.700	LO9S2V
34477	Abraham	Holman	Development	$50.000	UU2ALK
37648	John	Smith	Marketing	$64.350	3SL99A зроби з цього таблицю для гут хабу щоб я просто скопіювала

Компанія зберігає у своїх базах даних наступну інформацію про співробітників: унікальний номер співробітника («userid»), прізвище, ім'я, відділ, зарплату та номер автентифікації транзакції («auth_tan»). Кожна з цих частин інформації зберігається в окремому стовпці, а кожен рядок представляє одного співробітника компанії.
SQL-запити можна використовувати для модифікації таблиці бази даних та її індексних структур, а також для додавання, оновлення та видалення рядків даних.

## Існує три основні категорії команд SQL:
Data Manipulation Language (DML) — мова маніпулювання даними.
Data Definition Language (DDL) — мова визначення даних.
Data Control Language (DCL) — мова керування даними.
Кожен із цих типів команд може бути використаний зловмисниками для порушення конфіденційності, цілісності та/або доступності системи. Переходьте до уроку, щоб дізнатися більше про типи команд SQL та їхній зв'язок із цілями захисту.

## Хід роботи

## Етап 1. What is SQL?
Проаналізувавши таблицю Employees та вимогу завдання — знайти відділ, у якому працює Bob Franco. Можна сформувати SQL-запит через SELECT department, потрібен саме стовпчик із відділом, за допомогою FROM Employees, а умовою WHERE first_name = 'Bob' AND last_name = 'Franco'.

<img width="1188" height="315" alt="image" src="https://github.com/user-attachments/assets/789bd39e-6143-45fd-8ff9-2bca3f2b8a23" />

## Етап 2. Data Manipulation Language (DML) — Мова маніпулювання даними
Для виконання цього завдання потрібно сформувати SQL-запит, використати команду UPDATE Employees для вибору таблиці, SET department = 'Sales' для оновлення значення відділу та умову WHERE first_name = 'Tobi' AND last_name = 'Barnett' (або за userid = 89762), щоб вказати конкретного працівника.

<img width="1281" height="309" alt="image" src="https://github.com/user-attachments/assets/817321e0-b5e5-4181-bcf6-3901e5ba2ffd" />

## Етап 3. Data Definition Language (DDL)
Для виконання цього завдання ми використовуємо команду ALTER, яка дозволяє змінити вже існуючу таблицю. За допомогою команди ADD додаємо phone varchar(20).

<img width="983" height="238" alt="image" src="https://github.com/user-attachments/assets/a23ab280-b883-4ccb-8123-b88af7174e08" />

Це завдання демонструє, як ін'єкція може дозволити хакеру не просто вкрасти дані, а змінити саму архітектуру бази, наприклад, додавши нові поля для збору шкідливої інформації або видаливши критично важливі індекси

## Етап 4. Data Control Language (DCL)
Для цього завдання використовцю команду GRANТ, яка надає користувачеві привілеї доступу до об'єктів бази даних. І ввожу команду GRANT ALL ON grant_rights TO unauthorized_user.

<img width="819" height="238" alt="image" src="https://github.com/user-attachments/assets/77fc55d7-272b-439b-b3fa-c67df265da78" />

Це завдання підкреслює найнебезпечніший аспект ін'єкцій: можливість повного захоплення контролю над системою керування базами даних (RDBMS).

## Етап 5. String SQL injection

<img width="1021" height="491" alt="image" src="https://github.com/user-attachments/assets/ee30bebb-435f-4a14-840f-509b5bf74e82" />

## Етап 6. Numeric SQL injection
Для виконання цього завдання ми використовуємо вразливість числової SQL-ін'єкції у параметрі User_Id. За допомогою додавання умови 1 OR 1=1 ми робимо запит завжди правдивим, що дозволяє обійти перевірку та отримати всі дані з таблиці users.

<img width="783" height="563" alt="image" src="https://github.com/user-attachments/assets/9fc5e77e-cb1f-434b-bcef-af0fddf704fb" />

## Етап 7. Compromising confidentiality with String SQL injection
Для виконання цього завдання ми використовуємо вразливість SQL-ін'єкції у полі введення прізвища або TAN-номера. За допомогою додавання умови з оператором OR (наприклад, ' OR '1'='1) ми робимо логічну умову завжди істинною, що дозволяє обійти перевірку автентифікації та отримати всі дані з таблиці employees.

<img width="1192" height="404" alt="image" src="https://github.com/user-attachments/assets/c4949327-4551-4074-9064-754eb1238a85" />

## Етап 8. Compromising Integrity with Query chaining
Для виконання цього завдання ми використовуємо команду UPDATE, яка дозволяє змінити вже існуючі дані в таблиці employees. За допомогою спеціального введення в поле прізвища ми закриваємо поточний запит і додаємо власний запит на оновлення, щоб збільшити свою заробітну плату та заробляти більше за інших.

<img width="1140" height="334" alt="image" src="https://github.com/user-attachments/assets/c81000b1-1016-4ee7-8b09-61824b197bac" />

## Етап 9. Compromising Availability
Для виконання цього завдання ми використовуємо техніку ланцюжків запитів query chaining та команду видалення DROP, яка дозволяє повністю видалити таблицю access_log із бази даних. За допомогою спеціального введення ми закриваємо попередній запит і додаємо нову команду DDL, щоб стерти історію своїх дій і замести сліди.

<img width="927" height="251" alt="image" src="https://github.com/user-attachments/assets/a597d14b-3fbd-4689-94cb-35f2cfd95001" />

<img width="476" height="58" alt="image" src="https://github.com/user-attachments/assets/cc961d64-71b2-42b2-845e-aa1039a3dbd2" />

## Висновок 
Під час виконання цієї лабораторної роботи ми ознайомилися з основними вразливостями вебдодатків, пов'язаними з маніпуляцією даними та SQL-ін'єкціями.
Дослідили механізми DML: навчилися аналізувати структуру запитів і виконувати команди оновлення даних UPDATE, змінюючи інформацію про працівників у базі даних.
Освоїли принципи SQL-ін'єкцій вивчили способи обходу логічних перевірок та автентифікації за допомогою модифікації умов WHERE та використання операторів OR.
Застосували техніку ланцюжків запитів query chaining навчилися закривати поточні інструкції та виконувати додаткові команди, зокрема деструктивні операції DDL DROP TABLE, що дозволило керувати об'єктами бази даних та видаляти журнали дій access_log.


### Завдання на вибір: Cross Site Scripting (stored)

## Concept
After looking at Reflected XSS in the previous lesson, we are now going to take a closer look at another form of Cross-Site Scripting Attack: Stored XSS.

## Goals
    The user will learn what Stored XSS is
    The user will demonstrate knowledge on:
        Stored XSS injection

## Stored XSS
Stored Cross-Site Scripting is different in that the payload is persisted (stored) instead of passed/injected via a link.
Stored XSS Scenario

    Attacker posts malicious script to a message board
    Message is stored in a server database
    Victim reads the message
    The malicious script embedded in the message board post executes in the victim’s browser
        The script steals sensitive information, like the session id, and releases it to the attacker
Victim does not realize attack occurred

## Етап 1.
See the comments below.
Add a comment with a JavaScript payload. Again …​ you want to call the webgoat.customjs.phoneHome function.
As an attacker (offensive security), keep in mind that most apps will not have such a straightforwardly named compromise. Also, you may have to find a way to load your JavaScript dynamically to achieve the goal of extracting data fully.

<img width="1331" height="686" alt="image" src="https://github.com/user-attachments/assets/10d743ea-7d71-4806-89cf-70e9998b9e46" />

Для виконання цього завдання використовуємо техніку збереженого міжсайтового скриптингу Stored XSS та впровадження довільного JavaScript-коду через форму коментарів вебдодатка. За допомогою спеціального тегу <script> вбудовуємо виклик службової функції webgoat.customjs.phoneHome(), яка під час оновлення сторінки звертається до сервера і генерує унікальний ідентифікатор сесії та виводить його у консоль розробника (F12) для успішного проходження верифікації.

Скрипт який потрібно ввести у коментарі: <script>webgoat.customjs.phoneHome()</script>

<img width="553" height="463" alt="image" src="https://github.com/user-attachments/assets/ced104de-e57c-420c-85e4-c26eec7a0107" />

У консолі дивимося ідентифікатор сесії

<img width="1912" height="32" alt="image" src="https://github.com/user-attachments/assets/1bc28554-4346-4b19-8179-937c54d4a8c8" />

Вводжу ідентифікатор: -2010856158

<img width="806" height="155" alt="image" src="https://github.com/user-attachments/assets/378ab2d3-cb29-4bae-be8d-056a6d007686" />

## Висновок
Для виконання цього завдання використовуємо техніку збереженого міжсайтового скриптингу Stored XSS та впровадження довільного JavaScript-коду через форму коментарів вебдодатка. За допомогою спеціального тегу <script> ми вбудовуємо виклик службової функції webgoat.customjs.phoneHome(), яка під час оновлення сторінки автоматично виконується у браузері, звертається до сервера, генерує унікальний ідентифікатор відповіді та дозволяє отримати необхідні дані для успішного проходження верифікації лабораторної роботи.

