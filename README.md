# Решение домашнего задания к занятию "13.Системы мониторинга"
https://github.com/netology-code/mnt-homeworks/blob/MNT-video/10-monitoring-02-systems/README.md

### 1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы выведите в мониторинг и почему?
1. Загрузка ЦПУ (CPU Usage) Поскольку вычисления загружают ЦПУ  
2. Диск I/O (Disk Input/Output): Поскольку отчеты сохраняются на диск, важно следить за производительностью операций чтения и записи.  
3. Диск свободное место (Disk free): Поскольку переполнение диска модет привести к отсказу основной функции системы.  
4. Использование памяти (Memory Usage): Для вычислительных задач важно следить за использованием оперативной памяти.  
   
### 2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы можете ему предложить?
1. Метрика, показывающая время отклика HTTP . Можно связать метрику с показателем  SLA - клиент должен получить ответ вовремя например, в среднем не более 5 секунд на ответ  
2. Метрика, показывающая долю успешно выполненных http запросов. Если SLA=99% по http кодам ответов, по этой метрике можно контролировать SLI  

### 3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, чтобы разработчики получали ошибки приложения?
Настройка алертов средствами системы мониторинга. Автоматическая отправка алертов разарбочикам (месенджеры, email)

### 4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?
Вероятно, в сиситеме могут использоваться следующие группы ответов, не учтеные в этом выражении  
1. 3ХХ, например редиректы  
2. 1хх, например вебсокеты поверх http или другие стримы  

### 5. Опишите основные плюсы и минусы pull и push систем мониторинга.

Плюсы push-модели:  
упрощение репликации данных в разные системы мониторинга или их резервные копии 
более гибкая настройка отправки пакетов данных с метриками  
UDP — это менее затратный способ передачи данных, из-за чего может возрасти производительность сбора метрик  

Минусы push-модели:  
Сложность реализации: Push-модели могут быть сложнее в настройке, особенно если требуется обеспечить надежную передачу данных.  
Риск потери данных: Если целевая система отправляет данные, но мониторинг временно недоступен, информация может быть потеряна.  
Управление нагрузкой: Необходимо следить за количеством отправляемых данных, чтобы избежать перегрузки сети или серверов мониторинга.  

Плюсы pull-модели:  
легче контролировать подлинность данных  
можно настроить единый proxy server до всех агентов с TLS  
упрощённая отладка получения данных с агентов    

Минусы pull-модели:  
задержка в получении данных: Данные обновляются только в момент опроса, что может привести к задержкам в обнаружении проблем.
неполное покрытие: В случае редкого опроса можно пропустить кратковременные проблемы.
нагрузка на сеть: Регулярные запросы могут создавать дополнительный трафик, особенно при большом количестве узлов.  

### 6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные? Prometheus TICK Zabbix VictoriaMetrics Nagios

Pull модели:  
Prometheus: Использует pull модель, запрашивая метрики у целевых систем по расписанию. Но есть вариант закостылить в Push модель через Pushgateway  
Zabbix: В основном использует pull модель для сбора данных с агентов, хотя также поддерживает push через Zabbix Sender.  
VictoriaMetrics: Основная модель — pull, но также поддерживает push через специальный API.   

Push модели:  
TICK: Включает в себя Telegraf, который использует push модель для отправки данных в InfluxDB.  
Nagios: Обычно работает по модели pull  

Гибридные модели:  
Zabbix: Как упоминалось, в основном использует pull, но также поддерживает push через Zabbix Sender.  
VictoriaMetrics: Поддерживает как pull, так и push модели.  

### 8. Скриншот с метриками CPU
![image](https://github.com/user-attachments/assets/784ffa52-75e2-4b33-b49a-e46f1645e3b6)

### 9. Список метрик, в том числе связанных с docker
![image](https://github.com/user-attachments/assets/37761bf0-226a-4389-a467-219836a37524)

### 1*. Сбор метрик скриптом в лог
#### a. Скрипт
```
import os
import json
import time
from datetime import datetime

def collect():
    metrics = {}
    timestamp = int(time.time())
    metrics['timestamp'] = timestamp

    with open('/proc/loadavg', 'r') as f:
        loadavg = f.read().strip().split()
        metrics['load_1m'] = float(loadavg[0])
        metrics['load_5m'] = float(loadavg[1])
        metrics['load_15m'] = float(loadavg[2])

    with open('/proc/meminfo', 'r') as f:
        for line in f:
            if line.startswith('MemTotal:'):
                metrics['mem_total'] = int(line.split()[1])
            elif line.startswith('MemAvailable:'):
                metrics['mem_available'] = int(line.split()[1])

    with open(f"/proc/uptime", "r") as f:
        uptime = f.read().split()
        metrics['uptime'] = uptime[0]

    return metrics

def writeToLog(metrics):
    today = datetime.now()
    logFilename = f"/home/ubuntu/{today.strftime('%y-%m-%d')}-awesome-monitoring.log"

    with open(logFilename, 'a') as logFile:
        logFile.write(json.dumps(metrics) + '\n')

if __name__ == "__main__":
    metrics = collect()
    writeToLog(metrics)
```

#### б. Cron
```
* * * * * python3 /home/ubuntu/metrics.py
```
#### в. Лог
![image](https://github.com/user-attachments/assets/31678f80-8af0-4199-b35a-8ec1c9c1c1de)


