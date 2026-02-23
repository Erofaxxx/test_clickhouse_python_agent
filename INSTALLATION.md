# Инструкция по установке и запуску ClickHouse AI Agent на Ubuntu

## Требования

- Ubuntu 20.04 или новее
- Python 3.11 или выше
- Доступ к интернету для установки пакетов
- API ключ от Anthropic (Claude)
- Доступ к ClickHouse базе данных на Яндекс Cloud

## Шаг 1: Подготовка системы

Обновите систему и установите необходимые пакеты:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv wget git
```

## Шаг 2: Создание директории проекта

Создайте директорию для агента и перейдите в неё:

```bash
mkdir -p ~/clickhouse_agent
cd ~/clickhouse_agent
```

## Шаг 3: Скачивание проекта

Если у вас есть доступ к Git репозиторию:

```bash
git clone https://github.com/Erofaxxx/test_clickhouse_python_agent.git
cd test_clickhouse_python_agent/cli_agent
```

Или создайте структуру вручную и скопируйте файлы проекта.

## Шаг 4: Создание виртуального окружения

Создайте и активируйте виртуальное окружение Python:

```bash
python3 -m venv venv
source venv/bin/activate
```

После активации в начале строки терминала появится `(venv)`.

## Шаг 5: Установка зависимостей

Установите необходимые Python пакеты:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Список устанавливаемых пакетов:
- `anthropic>=0.40.0` - SDK для работы с Claude AI
- `clickhouse-connect>=0.7.0` - Клиент для ClickHouse
- `pandas>=2.0.0` - Библиотека для работы с данными
- `numpy>=1.24.0` - Библиотека для численных вычислений
- `pyarrow>=14.0.0` - Библиотека для работы с Parquet форматом
- `python-dotenv>=1.0.0` - Загрузка переменных окружения из .env файла
- `tabulate>=0.9.0` - Форматирование таблиц

## Шаг 6: Скачивание SSL сертификата Яндекс Cloud

Для безопасного подключения к ClickHouse в Яндекс Cloud нужен SSL сертификат:

```bash
wget https://storage.yandexcloud.net/cloud-certs/CA.pem -O YandexInternalRootCA.crt
```

## Шаг 7: Настройка переменных окружения

Создайте файл `.env` на основе `.env.example`:

```bash
cp .env.example .env
nano .env
```

Заполните следующие параметры:

```bash
# API ключ от Anthropic (получить на https://console.anthropic.com/)
ANTHROPIC_API_KEY=sk-ant-ваш-ключ-здесь

# Параметры подключения к ClickHouse
CLICKHOUSE_HOST=rc1b-vsrkuug8qh3pkkeg.mdb.yandexcloud.net
CLICKHOUSE_PORT=8443
CLICKHOUSE_USER=User_main
CLICKHOUSE_PASSWORD=click_security_house_7659
CLICKHOUSE_DATABASE=ym_sanok
CLICKHOUSE_SSL_CERT_PATH=YandexInternalRootCA.crt
```

**Важно:** Замените `ANTHROPIC_API_KEY` на ваш настоящий API ключ от Anthropic.

Сохраните файл (в nano: Ctrl+O, Enter, Ctrl+X).

## Шаг 8: Создание директории для временных файлов

Директория `temp_data` создастся автоматически при первом запуске, но можно создать её заранее:

```bash
mkdir -p temp_data
```

## Шаг 9: Проверка установки

Убедитесь, что все файлы на месте:

```bash
ls -la
```

Вы должны увидеть:
- `cli_agent.py` - основной файл агента
- `requirements.txt` - список зависимостей
- `.env` - файл с настройками (не должен быть в Git)
- `.env.example` - пример настроек
- `YandexInternalRootCA.crt` - SSL сертификат
- `temp_data/` - директория для временных файлов
- `venv/` - виртуальное окружение

## Шаг 10: Запуск агента

Активируйте виртуальное окружение (если ещё не активировано):

```bash
source venv/bin/activate
```

Запустите агента:

```bash
python3 cli_agent.py
```

При успешном запуске вы увидите:

```
============================================================
  ClickHouse Analysis Agent (CLI)
  Model: claude-sonnet-4-6-20250514
============================================================
✅ SSL сертификат: /home/user/clickhouse_agent/YandexInternalRootCA.crt
✅ ClickHouse подключён: rc1b-vsrkuug8qh3pkkeg.mdb.yandexcloud.net:8443/ym_sanok

💬 Введите запрос. 'exit' или 'выход' для завершения.

❓ Ваш запрос:
```

## Примеры использования

### Пример 1: Просмотр структуры базы данных

```
❓ Ваш запрос: покажи список всех таблиц в базе
```

Агент вызовет инструмент `list_tables` и покажет все таблицы с их колонками и типами.

### Пример 2: Выгрузка данных из таблицы

```
❓ Ваш запрос: выгрузи первые 10 строк из таблицы visits_complete
```

Агент:
1. Выполнит SQL запрос `SELECT * FROM visits_complete LIMIT 10`
2. Сохранит результат в Parquet файл
3. Покажет первые 5 строк прямо в терминале

### Пример 3: Анализ данных

```
❓ Ваш запрос: посчитай количество визитов по дням за последнюю неделю
```

Агент:
1. Составит SQL запрос с GROUP BY
2. Выгрузит данные в Parquet
3. Выполнит Python код для анализа
4. Покажет результат в виде таблицы

### Пример 4: Выход из программы

```
❓ Ваш запрос: exit
```

Или нажмите `Ctrl+C`.

## Устранение неполадок

### Ошибка: "ModuleNotFoundError"

**Решение:** Убедитесь, что виртуальное окружение активировано и все зависимости установлены:

```bash
source venv/bin/activate
pip install -r requirements.txt
```

### Ошибка: "ANTHROPIC_API_KEY not found"

**Решение:** Проверьте, что файл `.env` существует и содержит корректный API ключ:

```bash
cat .env | grep ANTHROPIC_API_KEY
```

### Ошибка подключения к ClickHouse

**Решение:** Проверьте:
1. Доступность сервера ClickHouse:
   ```bash
   ping rc1b-vsrkuug8qh3pkkeg.mdb.yandexcloud.net
   ```
2. Корректность учетных данных в `.env`
3. Наличие SSL сертификата:
   ```bash
   ls -la YandexInternalRootCA.crt
   ```

### Ошибка: "SSL certificate verify failed"

**Решение:** Убедитесь, что SSL сертификат скачан:

```bash
wget https://storage.yandexcloud.net/cloud-certs/CA.pem -O YandexInternalRootCA.crt
```

## Автозапуск при загрузке системы (опционально)

Чтобы агент запускался автоматически при старте системы, создайте systemd service:

### 1. Создайте файл сервиса:

```bash
sudo nano /etc/systemd/system/clickhouse-agent.service
```

### 2. Добавьте содержимое:

```ini
[Unit]
Description=ClickHouse AI Agent
After=network.target

[Service]
Type=simple
User=your_username
WorkingDirectory=/home/your_username/clickhouse_agent/cli_agent
ExecStart=/home/your_username/clickhouse_agent/cli_agent/venv/bin/python3 /home/your_username/clickhouse_agent/cli_agent/cli_agent.py
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Важно:** Замените `your_username` на ваше имя пользователя.

### 3. Активируйте сервис:

```bash
sudo systemctl daemon-reload
sudo systemctl enable clickhouse-agent.service
sudo systemctl start clickhouse-agent.service
```

### 4. Проверьте статус:

```bash
sudo systemctl status clickhouse-agent.service
```

## Обновление агента

Для обновления агента до новой версии:

```bash
cd ~/clickhouse_agent/test_clickhouse_python_agent
git pull origin main
cd cli_agent
source venv/bin/activate
pip install --upgrade -r requirements.txt
```

## Удаление агента

Если нужно полностью удалить агент:

```bash
# Остановить сервис (если настроен)
sudo systemctl stop clickhouse-agent.service
sudo systemctl disable clickhouse-agent.service
sudo rm /etc/systemd/system/clickhouse-agent.service

# Удалить директорию проекта
rm -rf ~/clickhouse_agent

# Деактивировать виртуальное окружение
deactivate
```

## Безопасность

**ВАЖНО:**
- Никогда не загружайте файл `.env` в Git - он содержит конфиденциальные данные
- Храните API ключи и пароли в безопасности
- Регулярно меняйте пароли от базы данных
- Ограничьте права доступа к файлу `.env`:
  ```bash
  chmod 600 .env
  ```

## Поддержка

Если у вас возникли вопросы или проблемы:
1. Проверьте логи в терминале
2. Убедитесь, что все требования установлены
3. Проверьте настройки в `.env` файле
4. Создайте issue в GitHub репозитории проекта

## Лицензия

Проект распространяется по лицензии MIT. Подробности в файле LICENSE.
