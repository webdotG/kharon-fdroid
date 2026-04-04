Kharon v2.0 Reception Hours Edition  
Проблема которую решал  
Мессенджер требовал постоянного соединения 24/7. Батарея садилась, Android убивал процесс, сообщения терялись.  
Архитектурные решения  
1. Reception Mode протокол  
Клиент при подключении сообщает серверу свой режим через hello и mode_update. Сервер бродкастит изменения всем контактам.     Каждый участник видит режим собеседника в заголовке чата.   
2. Динамический TTL на сервере    
Сервер хранит lastKnownMode для каждого pubKey. При постановке в очередь: TTL = 2 минуты + интервал_PULSE_получателя. Это   значит сообщение живёт ровно столько сколько нужно чтобы получатель успел открыть окно.  
3. AlarmManager PULSE цикл  
scheduleNextPulse() → AlarmManager EXACT → startForegroundService →   
connect → flush queue → queue_end → disconnect → scheduleNextPulse()  
На Android 12+ запрашиваем SCHEDULE_EXACT_ALARM. На 10-11 работает без разрешений.  
4. Временное LIVE при открытии чата  
ACTION_CHAT_OPEN → ensureConnected() поднимает сокет если офлайн.  
ACTION_CHAT_CLOSE → восстанавливает PULSE режим через 1 секунду.  
5. Буферизация и счётчик непрочитанных  
pendingByContact — буфер для офлайн контактов.  
_unreadByContact — счётчик инкрементируется при ЛЮБОМ входящем, сбрасывается при registerChat.  
6. Credits система  
Максимум 10 одновременных SENT сообщений. Защита от спама на уровне клиента.  
7. BootReceiver  
Автозапуск сервиса после перезагрузки через BOOT_COMPLETED.  
Потенциальные проблемы которые стоит проверить  
Двойной scheduleNextPulse — в логах видели двойной вызов. Флаг pulseScheduled добавили но надо проверить что не накапливаются лишние будильники.  
chatClose restoring mode=LIVE — когда закрываем чат и сами в LIVE, сервис пытается "восстановить" режим хотя он уже LIVE. Лишняя операция но не критично.  
unreadCount сбрасывается при обновлении БД — решили через socket.getUnreadCount() в маппинге из getAllFlow.  


Ссылки  
Код: https://github.com/webdotG/kharon-mess  
F-Droid репо: https://webdotg.github.io/kharon-fdroid/repo  
  
Для установки через F-Droid:  

F-Droid → Settings → Repositories → +  
https://webdotg.github.io/kharon-fdroid/repo   
Найди Kharon Messenger   


## Добавление репозитория

### Способ 1: Вручную в F-Droid
Settings → Repositories → + → вставь URL:  
https://webdotg.github.io/kharon-fdroid/repo  

## Прямая установка APK

[Скачать com.kharon.messenger_3.apk](repo/com.kharon.messenger_3.apk)  