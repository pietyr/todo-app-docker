# Planer zadań - Aplikacja TODO (Docker Compose)

Projekt zaliczeniowy z przedmiotu Programowanie Aplikacji w Chmurze Obliczeniowej. 

Aplikacja typu full-stack do zarządzania zadaniami w ujęciu tygodniowym (od poniedziałku do piątku), zrealizowana w architekturze trójwarstwowej i w pełni skonteneryzowana przy użyciu Docker oraz Docker Compose.

## Struktura i Technologie

Projekt składa się z trzech niezależnych komponentów odizolowanych w osobnych kontenerach:

1. **Frontend (vue_frontend)**:
   * Technologia: Vue.js 3, Axios, Tailwind CSS.

2. **Backend (laravel_backend)**:
   * Technologia: PHP 8.5, Laravel (REST API).

3. **Baza Danych (mariadb_db)**:
   * Technologia: MariaDB 10.11 (LTS).

## Instrukcja Uruchomienia

### 1. Wymagania systemowe
* Zainstalowane środowisko Docker.
* Zainstalowane narzędzie Docker Compose.

### 2. Uruchomienie projektu
Aplikacja została w pełni zautomatyzowana. Jedno polecenie odpowiada za:
* Pobranie i zainstalowanie zależności backendowych (Composer) oraz frontendowych (NPM).
* Przygotowanie pliku konfiguracyjnego `.env` oraz wygenerowanie kluczy bezpieczeństwa dla frameworka Laravel.
* **Automatyczne oczekiwanie na gotowość bazy danych MariaDB i wykonanie migracji bazy danych.**
* Uruchomienie serwerów deweloperskich.

Aby uruchomić cały projekt, wystarczy w jego katalogu głównym wykonać polecenie:
```bash
docker compose up -d --build
```

### 3. Dostęp do usług
Po poprawnym uruchomieniu, serwisy są dostępne pod następującymi adresami lokalnymi:
* **Frontend (Aplikacja kliencka)**: [http://localhost:5173](http://localhost:5173)
* **Backend API (Laravel)**: [http://localhost:8000/api/tasks](http://localhost:8000/api/tasks)
* **Zewnętrzny dostęp do bazy danych (MariaDB)**: Host: `localhost`, Port: `3307`, Użytkownik: `user`, Hasło: `password`, Baza danych: `app`.
