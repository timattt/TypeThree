# Lab-killer

Веб-приложение для лабораторных вычислений.
Основная часть - это язык программирования, похожий на матлаб, но с физтеховской спецификой.
Документация по языку представленна [здесь](https://github.com/timattt/TypeThree/blob/master/manual.pdf)

# Архитектура

![image](https://github.com/timattt/TypeThree/assets/25401699/e0ee4363-ed55-4f46-b2ba-ab34cd8db1dd)

## Сущности

* **WebClient** - приложение на ReactJS. Пользователь пишет программу, потом она улетает на сервер, там воркеры ее выполняют и результат кладут в БД. А клиент каждые несколько секунд пингует **ResultsProvider**, и когда воркеры закончат он получит результат выполнения своего кода. Выглядеть в браузере это будет примерно так:
<img width="493" alt="image" src="https://github.com/timattt/TypeThree/assets/25401699/7d43d245-ab3e-4978-80a1-c1c6155e22d4">

* **Gateway** - сервис, который занимается перенаправлением трафика. Используем Spring Cloud Gateway. Через него внешний мир получает доступ к нашей инфраструктуре.
* **AuthService** - сервис, который занимается безопасностью пользователей. В данном проекте целесообразно использовать JWT. Пользователь через **WebClient** будет регистрироваться используя сторонние сервисы вроде яндекса или вк, а потом уже ему на основе сторонних токенов будет выдаваться наш локальный.
* **Interpreter** - интерпретатор осуществляет получение кода от клиента - создаем таску и отправляет ее в очередь.
* **InterpreterWorker** - краеугольный камень всей системы. Именно этот сервис обрабатывает языковые запросы и выполняет код.
* **TasksQueue** - кафка-очередь, которая хранит задачи. Куски кода, которые надо выполнить. Каждую таску должен получить ровно один воркер.
* **Results DB** - postgres база данных, в которую воркеры кладут результаты выполнения своих тасок.
* **ResultsProvider** - сервис, к которому есть доступ из клиента - он позволяет проверять - выполнена ли задача или нет. А также получать всю информацию о ней.
* **Utility** - группа сервисов для стандартных задач вроде хранения конфигурации и обнаружения. Реализовываются из коробки с помощью Spring Cloud. Использую яндекс облако, необходимость в них потенциально отпадает, так как там есть все свое.
* **Kafka broker** - брокер для очередей.
