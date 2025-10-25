# C4 — Диаграмма контекста

```puml
@startuml
!include ../../c4/C4_Context.puml
LAYOUT_WITH_LEGEND()

title Кинобездна — Диаграмма контекста

Person(User, "Зритель", "Смотрит контент, оценки, избранное")
Person(Admin, "Оператор", "Управляет каталогом")

System(SystemUD, "Кинобездна", "Онлайн-кинотеатр-агрегатор")

System_Ext(PayProvider, "Платёжные провайдеры", "Оплата/вебхуки")
System_Ext(RecoSys, "Рекомендательная система", "Подборки контента")

Rel(User, SystemUD, "Использует", "HTTPS")
Rel(Admin, SystemUD, "Управляет", "HTTPS")
Rel(SystemUD, PayProvider, "Платежи", "HTTPS/Webhooks")
Rel(SystemUD, RecoSys, "События/рекомендации", "HTTPS/Kafka")

@enduml
