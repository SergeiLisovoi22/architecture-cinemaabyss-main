# C4 — Диаграмма контекста (To-Be)

```puml
@startuml
!include ../../c4/C4_Context.puml
LAYOUT_WITH_LEGEND()

title Кинобездна — Диаграмма контекста

Person(User, "Зритель", "Авторизуется, подписывается, смотрит, оценивает, делает избранное")
Person(Admin, "Оператор/Контент-менеджер", "Ведёт каталог/метаданные, поддержка")
Person(Marketing, "Маркетинг/Партнёры", "Запускают акции, промо, партнёрки")

System(SystemUD, "Кинобездна", "Онлайн-кинотеатр-агрегатор")

' Внешние системы (расширено)
System_Ext(IdP, "IdP/SSO (OIDC)", "Авторизация/социальные входы")
System_Ext(PayProvider, "Платёжные провайдеры (PSP)", "Оплата/рекурренты/рефанды")
System_Ext(AntiFraud, "Антифрод", "Оценка риска транзакций")
System_Ext(RecoSys, "Рекомендательная система", "Подборки")
System_Ext(CDN, "CDN/Storage/DRM", "Постеры/сабы/трейлеры/DRM-ключи")
System_Ext(EmailSMS, "Email/SMS/Push провайдеры", "Коммуникации")
System_Ext(Loyalty, "Маркетплейсы/Лояльность", "Купоны/бонусы/кросс-промо")
System_Ext(Analytics, "DWH/Аналитика", "Сырые события/витрины/отчёты")
System_Ext(Tax, "Налоги/чек/фискализация", "Отчётность по оплатам")

Rel(User, SystemUD, "Использование веб/мобильного/TV клиента", "HTTPS")
Rel(Admin, SystemUD, "Backoffice/операции", "HTTPS")
Rel(Marketing, SystemUD, "Промо/партнёрки/кампании", "HTTPS")

Rel(SystemUD, IdP, "OIDC авторизация", "OIDC/OAuth2")
Rel(SystemUD, PayProvider, "Оплаты/вебхуки", "HTTPS/Webhooks")
Rel(SystemUD, AntiFraud, "Оценка риска", "HTTPS")
Rel(SystemUD, RecoSys, "События поведения + запрос подборок", "Kafka/HTTPS")
Rel(SystemUD, CDN, "Артефакты/DRM", "HTTPS/S3/KMS")
Rel(SystemUD, EmailSMS, "Триггерные сообщения", "HTTPS/SMTP")
Rel(SystemUD, Loyalty, "Купоны/бонусы", "HTTPS")
Rel(SystemUD, Analytics, "Экспорт событий/батчи", "Kafka/S3")
Rel(SystemUD, Tax, "Фискализация чеков", "HTTPS")

@enduml
