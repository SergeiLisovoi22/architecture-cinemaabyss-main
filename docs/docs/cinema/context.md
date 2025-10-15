
```puml
@startuml
title Кинобездна — Диаграмма контекста (C4: Level 1)

top to bottom direction
!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

' === People ===
Person(User, "Зритель", "Смотрит контент, управляет подпиской")
Person(Admin, "Оператор/Контент-менеджер", "Ведёт каталог, метаданные, поддержка")
Person(Marketing, "Маркетинг/Продажи", "Запускают акции и промо")

' === System Under Design ===
System(SystemUD, "Кинобездна", "Онлайн‑кинотеатр‑агрегатор: каталог, подписки, просмотр, рекомендации")

' === External Systems ===
System_Ext(PayProvider, "Платёжные провайдеры", "Acquirers/PSP")
System_Ext(RecoSys, "Внешняя рекомендательная система", "Формирует подборки")
System_Ext(Loyalty, "Партнёры/Маркетплейсы/Лояльность", "Выдача/погашение бонусов")
System_Ext(CDN, "CDN/Object Storage (S3)", "Хранение контента/обложек/сабов")
System_Ext(EmailSMS, "Email/SMS/Push провайдеры", "Коммуникации с пользователями")
System_Ext(Analytics, "DWH/Аналитика", "Отчётность и витрины данных")
System_Ext(IdP, "IdP/SSO (OIDC)", "Единая аутентификация")

' === Relationships (people -> system) ===
Rel(User, SystemUD, "Авторизация, выбор фильма, просмотр, оценки, избранное", "HTTPS/REST")
Rel(Admin, SystemUD, "Управление каталогом/метаданными, поддержка", "HTTPS")
Rel(Marketing, SystemUD, "Управление кампаниями, сегментами", "HTTPS")

' === Relationships (system <-> externals) ===
Rel(SystemUD, PayProvider, "Инициирует/подтверждает платежи, рефанды, рекуррентные платежи", "HTTPS/Webhooks")
Rel(SystemUD, RecoSys, "Отправляет события, запрашивает подборки", "Kafka/HTTPS")
Rel(SystemUD, Loyalty, "Интеграции по подпискам/купонам/бонусам", "HTTPS/webhooks")
Rel(SystemUD, CDN, "Отдача контента/картинок, загрузка ассетов", "HTTPS/S3")
Rel(SystemUD, EmailSMS, "Транзакционные/маркетинговые уведомления", "HTTPS/SMTP")
Rel(SystemUD, Analytics, "Экспорт событий и данных", "Kafka/S3")
Rel(SystemUD, IdP, "OIDC авторизация/SSO", "OIDC/OAuth2")

@enduml
```
