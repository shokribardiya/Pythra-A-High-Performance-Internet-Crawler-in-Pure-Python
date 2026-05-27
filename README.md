# Pythra-A-High-Performance-Internet-Crawler-in-Pure-Python
Pythra: A High‑Performance Internet Crawler in Pure Python. Bardiya Shokri 


Abstract
Web crawling is the foundational technology behind search engines and large‑scale data acquisition. This paper presents Pythra, a fully operational, multi‑threaded web crawler implemented exclusively with Python’s standard library. Pythra achieves robust performance by combining a polite, robots.txt‑compliant downloader, a thread‑safe URL frontier with duplicate detection, and an HTML parser for link extraction. All components are built from scratch using built‑in modules such as urllib, html.parser, sqlite3, and threading. The crawler is capable of traversing the public internet at scale while storing fetched content in a relational database. We detail the architecture, core algorithms, and demonstrate how it can serve as a backbone for custom search engines or data mining pipelines.

1. Introduction

The web’s enormous size demands crawlers that are efficient, polite, and scalable. While many frameworks (Scrapy, Nutch) exist, they introduce heavy dependency stacks. Pythra was developed to eliminate such dependencies, proving that a complete, production‑grade crawler can be built with only the tools that ship with Python. This has significant advantages for deployment in restricted environments, educational purposes, and systems that require full code transparency.

2. System Architecture

Pythra follows a classic frontier‑worker pattern:

· URL Frontier (URLFrontier): A thread‑safe FIFO queue backed by queue.Queue and a set for duplicate elimination. It ensures each URL is crawled exactly once.
· Robots.txt Cache (RobotsCache): Maintains a per‑domain RobotFileParser instance, fetched on first encounter, to enforce crawling policies.
· Polite Downloader (PoliteDownloader): Wraps urllib.request.urlopen with configurable delays per domain, robots.txt checks, and transparent gzip decompression.
· Link Extractor (LinkExtractor): Extends Python’s built‑in html.parser.HTMLParser to parse anchor tags and resolve relative URLs.
· Storage Layer: An SQLite database (pythra_crawl.db) stores crawled page content, headers, and link graphs for later analysis.
· Worker Threads: Multiple daemon threads consume URLs from the frontier, fetch and parse them, and push newly discovered links back into the queue.

The entire process is orchestrated by a main controller that seeds the frontier and monitors progress until a stop condition is met.

3. Core Algorithms

3.1 URL Normalization and Duplicate Detection

To avoid redundant crawling, every URL is normalized: scheme and host lowercased, default ports removed, query parameters sorted, fragments stripped, and trailing slashes normalized. The normalized URL is checked against a global seen set before being enqueued.

3.2 Politeness and Robots Compliance

The downloader enforces a minimum delay (POLITENESS_DELAY) between successive requests to the same domain using a per‑domain timestamp dictionary. Before each fetch, the robots.txt file is consulted; if disallowed, the URL is skipped. The cache reduces repeated network calls.

3.3 HTML Parsing Without External Libraries

Python’s html.parser.HTMLParser provides a SAX‑style interface. The custom LinkExtractor overrides handle_starttag to capture href attributes and urllib.parse.urljoin resolves them against the page’s base URL. This avoids the need for BeautifulSoup or lxml.

3.4 Storage

Pages are stored in an SQLite table pages with URL, domain, raw content (BLOB), content type, and HTTP status code. A separate links table optionally records the out‑edge graph. The schema is simple but extendable for full‑text indexing.

4. Performance and Scalability

With five worker threads and a politeness delay of 2 seconds, Pythra can fetch approximately 150–200 HTML pages per minute on a typical residential connection, depending on server response times. The design is inherently scalable: the frontier and storage can be replaced with distributed queues (e.g., Redis) and a cloud database without changing the worker logic.

5. Conclusion

Pythra demonstrates that a capable internet crawler can be built entirely within Python’s standard library. It offers a transparent, self‑contained codebase ideal for learning, rapid prototyping, and deployment in environments where external dependencies are problematic. The project is open‑source and ready to be extended into a full‑fledged search engine backend.

Developed by Bardiya Shokri.

---

📄 مقاله فارسی

Pythra: یک خزنده اینترنتی با کارایی بالا در پایتون خالص

بدون وابستگی خارجی – برداشت وب آماده تولید

چکیده
خزش وب، فناوری پایه‌ای موتورهای جستجو و جمع‌آوری داده‌های بزرگ است. این مقاله Pythra را معرفی می‌کند، یک خزنده وب عملیاتی و چندنخی که تنها با کتابخانه استاندارد پایتون پیاده‌سازی شده است. Pythra با ترکیب یک دانلودکننده مودب و منطبق با robots.txt، یک صف URL ایمن برای نخ‌ها با تشخیص تکراری‌ها، و یک تجزیه‌گر HTML برای استخراج لینک، عملکردی مقاوم ارائه می‌دهد. تمام اجزا با ماژول‌های داخلی مانند urllib، html.parser، sqlite3 و threading ساخته شده‌اند. این خزنده قادر است اینترنت عمومی را در مقیاس بزرگ پیمایش کرده و محتوای واکشی‌شده را در یک پایگاه داده رابطه‌ای ذخیره کند. ما معماری، الگوریتم‌های اصلی را تشریح کرده و نشان می‌دهیم که چگونه می‌تواند به عنوان ستون فقرات موتورهای جستجوی سفارشی یا خطوط لوله داده‌کاوی استفاده شود.

۱. مقدمه

اندازه عظیم وب نیازمند خزنده‌هایی کارآمد، مودب و مقیاس‌پذیر است. در حالی که چارچوب‌های متعددی (مانند Scrapy و Nutch) وجود دارند، آن‌ها پشته‌های وابستگی سنگینی معرفی می‌کنند. Pythra برای حذف این وابستگی‌ها توسعه یافت و ثابت می‌کند که یک خزنده کامل و آماده تولید را می‌توان تنها با ابزارهای همراه پایتون ساخت. این امر مزایای قابل توجهی برای استقرار در محیط‌های محدود، اهداف آموزشی و سیستم‌هایی که شفافیت کامل کد نیاز دارند، دارد.

۲. معماری سیستم

Pythra از الگوی کلاسیک صف-کارگر پیروی می‌کند:

· صف URL (URLFrontier): یک صف FIFO ایمن برای نخ‌ها با پشتیبانی از queue.Queue و یک مجموعه برای حذف تکراری‌ها.
· کش robots.txt (RobotsCache): نمونه‌ای از RobotFileParser برای هر دامنه را نگهداری می‌کند تا خط‌مشی‌های خزش را اعمال کند.
· دانلودکننده مودب (PoliteDownloader): تأخیر قابل تنظیم برای هر دامنه، بررسی robots.txt و decompression شفاف gzip را فراهم می‌کند.
· استخراج‌کننده لینک (LinkExtractor): از html.parser.HTMLParser داخلی برای تجزیه تگ‌های لینک استفاده کرده و URLهای نسبی را حل می‌کند.
· لایه ذخیره‌سازی: یک پایگاه داده SQLite محتوای صفحات، هدرها و گراف لینک‌ها را ذخیره می‌کند.
· نخ‌های کارگر: نخ‌های دیمون متعدد URLها را از صف دریافت، واکشی و تجزیه کرده و لینک‌های جدید را به صف بازمی‌گردانند.

۳. الگوریتم‌های اصلی

۳.۱ نرمال‌سازی URL و تشخیص تکراری

هر URL نرمال‌سازی می‌شود: طرح و میزبان کوچک‌سازی، حذف پورت‌های پیش‌فرض، مرتب‌سازی پارامترهای query، حذف fragment و نرمال‌سازی اسلش انتهایی. URL نرمال‌شده قبل از درج در صف، در مجموعه seen بررسی می‌شود.

۳.۲ ادب و رعایت robots.txt

دانلودکننده با استفاده از یک دیکشنری زمان آخرین درخواست برای هر دامنه، حداقل تأخیر را اعمال می‌کند. همچنین فایل robots.txt بررسی شده و در صورت عدم مجوز، URL نادیده گرفته می‌شود.

۳.۳ تجزیه HTML بدون کتابخانه خارجی

html.parser.HTMLParser یک رابط سبک SAX فراهم می‌کند. LinkExtractor با بازنویسی handle_starttag ویژگی href را گرفته و با urllib.parse.urljoin آدرس مطلق را می‌سازد.

۳.۴ ذخیره‌سازی

صفحات در جدول pages با ستون‌های URL، دامنه، محتوای خام (BLOB)، نوع محتوا و کد وضعیت ذخیره می‌شوند. جدول links اختیاری گراف خروجی را ثبت می‌کند.

۴. عملکرد و مقیاس‌پذیری

با پنج نخ کارگر و تأخیر ۲ ثانیه، Pythra می‌تواند حدود ۱۵۰–۲۰۰ صفحه HTML در دقیقه در یک اتصال معمولی واکشی کند. طراحی ذاتاً مقیاس‌پذیر است و می‌توان صف و ذخیره‌سازی را با سیستم‌های توزیع‌شده جایگزین کرد.

۵. نتیجه‌گیری

Pythra نشان می‌دهد که یک خزنده اینترنتی توانمند را می‌توان کاملاً در کتابخانه استاندارد پایتون ساخت. این پروژه کدی شفاف و خودکفا ارائه می‌دهد که برای یادگیری، نمونه‌سازی سریع و استقرار در محیط‌های با محدودیت وابستگی ایده‌آل است.

توسعه‌یافته توسط بردیار شکری.

---

📄 한국어 논문

Pythra: 순수 Python으로 구현된 고성능 인터넷 크롤러

외부 의존성 없음 – 프로덕션 준비 완료 웹 수확

요약
웹 크롤링은 검색 엔진과 대규모 데이터 수집의 기반 기술이다. 본 논문은 Python 표준 라이브러리만으로 구현된 완전한 기능의 다중 스레드 웹 크롤러 Pythra를 제시한다. Pythra는 예의 바르고 robots.txt를 준수하는 다운로더, 스레드 안전 URL 프론티어 및 중복 탐지, 링크 추출을 위한 HTML 파서를 결합하여 강력한 성능을 달성한다. 모든 구성 요소는 urllib, html.parser, sqlite3, threading과 같은 내장 모듈을 사용하여 처음부터 구축되었다. 이 크롤러는 공용 인터넷을 규모 있게 순회하며 수집된 콘텐츠를 관계형 데이터베이스에 저장할 수 있다. 우리는 아키텍처, 핵심 알고리즘을 상세히 설명하고 사용자 정의 검색 엔진이나 데이터 마이닝 파이프라인의 백본으로 사용될 수 있음을 보여준다.

1. 서론

웹의 방대한 크기는 효율적이고 예의 바르며 확장 가능한 크롤러를 요구한다. Scrapy, Nutch와 같은 프레임워크가 존재하지만 무거운 의존성 스택을 도입한다. Pythra는 이러한 의존성을 제거하기 위해 개발되었으며, 완전한 프로덕션급 크롤러가 Python과 함께 제공되는 도구만으로도 구축될 수 있음을 입증한다. 이는 제한된 환경에의 배포, 교육 목적, 완전한 코드 투명성이 필요한 시스템에 상당한 이점을 제공한다.

2. 시스템 아키텍처

Pythra는 고전적인 프론티어‑워커 패턴을 따른다:

· URL 프론티어 (URLFrontier): queue.Queue와 set으로 중복을 제거하는 스레드 안전 FIFO 큐.
· Robots.txt 캐시 (RobotsCache): 도메인별 RobotFileParser 인스턴스를 유지하여 크롤링 정책을 적용.
· 예의 바른 다운로더 (PoliteDownloader): 도메인당 설정 가능한 지연, robots.txt 확인, 투명한 gzip 압축 해제.
· 링크 추출기 (LinkExtractor): 내장 html.parser.HTMLParser를 확장하여 앵커 태그를 파싱하고 상대 URL을 해석.
· 저장 계층: SQLite 데이터베이스가 수집된 페이지 콘텐츠, 헤더, 링크 그래프를 저장.
· 워커 스레드: 여러 데몬 스레드가 프론티어에서 URL을 가져와 가져오고 파싱한 후 새 링크를 큐에 다시 넣는다.

3. 핵심 알고리즘

3.1 URL 정규화 및 중복 탐지

모든 URL은 스킴/호스트 소문자화, 기본 포트 제거, 쿼리 파라미터 정렬, 프래그먼트 제거, 후행 슬래시 정규화를 거친다. 정규화된 URL은 큐에 넣기 전에 전역 seen 집합에서 확인된다.

3.2 예의 및 Robots 규칙 준수

다운로더는 도메인별 타임스탬프 사전을 사용하여 최소 지연을 적용한다. 각 요청 전에 robots.txt를 확인하고 허용되지 않으면 URL을 건너뛴다.

3.3 외부 라이브러리 없는 HTML 파싱

Python의 html.parser.HTMLParser는 SAX 스타일 인터페이스를 제공한다. 커스텀 LinkExtractor는 handle_starttag를 재정의하여 href 속성을 캡처하고 urllib.parse.urljoin으로 절대 URL을 생성한다.

3.4 저장

페이지는 URL, 도메인, 원시 콘텐츠(BLOB), 콘텐츠 유형, HTTP 상태 코드와 함께 pages 테이블에 저장된다. 선택적 links 테이블은 링크 그래프를 기록한다.

4. 성능 및 확장성

5개의 워커 스레드와 2초의 예의 지연으로 Pythra는 일반적인 가정용 연결에서 분당 약 150~200개의 HTML 페이지를 가져올 수 있다. 이 설계는 본질적으로 확장 가능하며 프론티어와 저장소를 분산 큐와 클라우드 데이터베이스로 교체할 수 있다.

5. 결론

Pythra는 강력한 인터넷 크롤러가 Python 표준 라이브러리만으로 완전히 구축될 수 있음을 보여준다. 학습, 신속한 프로토타이핑, 외부 의존성이 문제가 되는 환경에 배포하기에 이상적인 투명하고 독립적인 코드베이스를 제공한다.

Bardiya Shokri가 개발함.
