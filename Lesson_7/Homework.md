# Домашнее задание: Создание демонов

**Студент:** Михеев Николай

**Группа:** РПО26/1/11

**Преподаватель:** Штейнберг Эмиль Эдуардович

**Дата:** 15.10.2026

## Задание 1. Создание программы-демона

### 1.1. Задание
Создайте программу, которая будет раз в несколько секунд писать в системный журнал (syslog), что данная программа активна. При запуске программы в журнал должна заноситься информация о событии запуска, при завершении (сигналы SIGTERM и SIGINT) — о событии завершения.

### 1.2. Команды
Использую редактор `nano` для создания скрипта `/home/user/my_daedalus.sh` (добавил строку `#!/bin/bash` для корректного интерпретирования):

```bash
#!/bin/bash
DAEMON_TAG="my_daedalus"
SLEEP_INTERVAL=5

shutdown_process() {
    logger -p local3.info -t "$DAEMON_TAG" "[Daedalus] ysmiren"
    exit 0
}

trap 'shutdown_process' SIGTERM SIGINT

logger -p local3.info -t "$DAEMON_TAG" "[Daedalus] zapyshen"

while true; do
    logger -p local3.info -t "$DAEMON_TAG" "[Daedalus] Proverka"
    sleep $SLEEP_INTERVAL
done
```

Выставление прав на исполнение и базовая проверка файла:
```bash
chmod +x /home/user/my_daedalus.sh
grep "my_daedalus" /home/user/my_daedalus.sh
```

## Задание 2. Создание юнит файла

### 2.1. Команды
Создаю файл конфигурации службы с помощью `nano /etc/systemd/system/my_daedalus.service`:

```ini
[Unit]
Description=My Daedalus Service Daemon
After=syslog.service

[Service]
ExecStart=/home/user/my_daedalus.sh
Restart=no
```

### 2.2. Запуск пользовательского демона
Выполните `systemctl daemon-reload` и `systemctl start mydaemon`.

```bash
systemctl daemon-reload
systemctl start my_daedalus
systemctl status my_daedalus
```

### 2.3. Проверка журнала
Убедитесь, что сообщения попадают в `/var/log/syslog`.

```bash
systemctl start rsyslog
grep my_daedalus /var/log/syslog | tail -5
```

### 2.4. Остановка
Остановите демон и проверьте сообщение о завершении.

```bash
systemctl stop my_daedalus
grep my_daedalus /var/log/syslog | tail -3
```

## Задание 3. Reboot

### 3.1. После перезапуска
Проверяю автоматический запуск демона в системном журнале после плановой перезагрузки:

```bash
grep my_daedalus /var/log/syslog | tail -6
```

### 3.2. Отключение автозапуска
Отключите автоматический запуск.

```bash
systemctl disable my_daedalus
systemctl stop my_daedalus
systemctl status my_daedalus
```

## Выводы
В ходе работы были изучены фоновые процессы (демоны) в Linux и управление ими через systemd. На практике был создан Bash-скрипт my_daedalus.sh с обработкой сигналов завершения, настроен системный unit-файл службы my_daedalus.service и включен его автозапуск при старте ОС. С помощью утилит systemctl и grep были успешно отработаны запуск процесса, проверка его логов в /var/log/syslog после перезагрузки сервера и последующее отключение в состояние inactive (dead).
