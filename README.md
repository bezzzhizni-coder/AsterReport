# AsterReport
```
gor@astergurman:/root$ sudo cat /etc/asterisk/manager.conf
;
; Asterisk Call Management support
;

; By default asterisk will listen on localhost only.
[general]
enabled = yes
port = 5038
bindaddr = 127.0.0.1

; No access is allowed by default.
; To set a password, create a file in /etc/asterisk/manager.d
; use creative permission games to allow other serivces to create their own
; files
#include "manager.d/*.conf"

gor@astergurman:~$ sudo cat check_aster-reg.sh
[sudo] password for gor:
#!/bin/bash -x

# Настройки подключения к Asterisk
ASTERISK_USER="user"
ASTERISK_PASS="1qaz@WSX"
ASTERISK_AMI_PORT=5038
REPORT_FILE="/var/log/asterisk/registration_report_$(date +%Y%m%d_%H%M%S).txt"

# Функция для подключения к AMI и получения статуса
check_registrations() {
    # Создаем временный файл для ответа
    TEMP_FILE=$(mktemp)

    # Подключаемся к AMI и получаем статус
    echo -e "Action: Login\r\nUsername: ${ASTERISK_USER}\r\nSecret: ${ASTERISK_PASS}\r\nEvents: off\r\n\r\nAction: SIPshowregistry\r\n\r\nAction: Logoff\r\n" | \
    nc -w 10 127.0.0.1 ${ASTERISK_AMI_PORT} > ${TEMP_FILE}

    # Обрабатываем результат
     if  grep -q 'No Authentication' ${TEMP_FILE}; then
        echo 'alarm' >> ${REPORT_FILE}
     else
```
+mail report
```
#    grep -E 'Registration|Status' ${TEMP_FILE} >> ${REPORT_FILE}
#    awk '{print $1}' | grep -E 'Username' ${TEMP_FILE} >> ${REPORT_FILE}
#    grep -E 'Registration|Event' ${TEMP_FILE} >> ${REPORT_FILE}
    grep -E 'Username' ${TEMP_FILE} >> ${REPORT_FILE}
    grep -E 'State' ${TEMP_FILE} >> ${REPORT_FILE}
        :
     fi
# Удаляем временный файл
    rm ${TEMP_FILE}
}

# Основная логика скрипта
echo "Отчет о регистрациях Asterisk - $(date)" > ${REPORT_FILE}
echo "----------------------------------------" >> ${REPORT_FILE}

# Проверяем статус
check_registrations

# Добавляем время окончания отчета
echo "----------------------------------------" >> ${REPORT_FILE}
echo "Отчет завершен в $(date)" >> ${REPORT_FILE}

# Права доступа к файлу отчета
chmod 640 ${REPORT_FILE}

echo "Отчет успешно создан: ${REPORT_FILE}"
