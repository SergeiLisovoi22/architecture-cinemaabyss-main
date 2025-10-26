
---

# docs/docs/cinema/container.md (главная To-Be, существенно расширена)

```md
# C4 — Диаграмма контейнеров (To-Be, доменная декомпозиция)

```puml
@startuml
!include ../../c4/C4_Container.puml
LAYOUT_WITH_LEGEND()
top to bottom direction

title Кинобездна — Контейнерная диаграмма (To-Be)

Person(User, "Зритель")
Person(Admin, "Оператор")
Person(Marketing, "Маркетинг/Партнёры")

' Внешние (для связей ниже)
System_Ext(IdP, "IdP/SSO (OIDC)")
System_Ext(PayProvider, "Платёжные провайдеры")
System_Ext(AntiFraud, "Антифрод")
System_Ext(RecoSys, "Рекомендательная система")
System_Ext(CDN, "CDN/Storage/DRM")
System_Ext(EmailSMS, "Email/SMS/Push")
System_Ext(Loyalty, "Маркетплейсы/Лояльность")
System_Ext(Analytics, "DWH/Аналитика")
System_Ext(Tax, "Фискализация/Налоги")

System_Boundary(K8S, "Кинобездна (Kubernetes)") {

  Container(Ingress, "Ingress / API Edge", "NGINX/Envoy", "TLS, маршрутизация, WAF")
  Container(Proxy, "Proxy (API Gateway)", "Java • Spring WebFlux", "Единая точка входа, Strangler Fig, A/B/FF")
  Rel(Ingress, Proxy, "HTTPS")

  ' === Домены и микросервисы (To-Be) ===
  Container_Boundary(DUsers, "Users/Auth") {
    Container(Auth, "Auth Service", "Go/Java", "Логин/регистрация/SSO/refresh")
    Container(Profile, "Profile Service", "Go/Java", "Профили/настройки/устройства")
    ContainerDb(UsersDB, "Users DB", "PostgreSQL", "Учётные записи/профили")
    Rel(Proxy, Auth, "REST /api/auth/*")
    Rel(Proxy, Profile, "REST /api/profile/*")
    Rel(Auth, UsersDB, "SQL")
    Rel(Profile, UsersDB, "SQL")
    Rel(Auth, IdP, "OIDC")
  }

  Container_Boundary(DCatalog, "Catalog/Metadata & Library") {
    Container(Movies, "Movies Service", "Go (целевой)/Node(MVP)", "Фильмы/жанры/актеры/метаданные/каталог")
    Container(Ratings, "Ratings Service", "Go/Java", "Оценки/рецензии")
    Container(Favorites, "Favorites Service", "Go/Java", "Избранное/плейлисты")
    ContainerDb(CatalogDB, "Catalog DB", "PostgreSQL", "Каталог/метаданные/индексы")
    Rel(Proxy, Movies, "REST /api/movies*")
    Rel(Proxy, Ratings, "REST /api/ratings/*")
    Rel(Proxy, Favorites, "REST /api/favorites/*")
    Rel(Movies, CatalogDB, "SQL")
    Rel(Ratings, CatalogDB, "SQL")
    Rel(Favorites, CatalogDB, "SQL")
    Rel(Movies, CDN, "Постеры/субтитры", "HTTPS/S3")
  }

  Container_Boundary(DBilling, "Billing: Payments & Subscriptions") {
    Container(Payments, "Payments Service", "Go/Java", "Авторизации/чаржи/рефанды")
    Container(Subs, "Subscriptions Service", "Go/Java", "Подписки/план/статус/возобновления")
    Container(Discounts, "Discounts/Promo Service", "Go/Java", "Купоны/акции/промо")
    ContainerDb(BillingDB, "Billing DB", "PostgreSQL", "Платежи/подписки/промо")
    Rel(Proxy, Payments, "REST /api/payments/*")
    Rel(Proxy, Subs, "REST /api/subscriptions/*")
    Rel(Proxy, Discounts, "REST /api/discounts/*")
    Rel(Payments, BillingDB, "SQL")
    Rel(Subs, BillingDB, "SQL")
    Rel(Discounts, BillingDB, "SQL")
    Rel(Payments, PayProvider, "HTTPS/Webhooks")
    Rel(Payments, AntiFraud, "HTTPS")
    Rel(Payments, Tax, "HTTPS")
    Rel(Discounts, Loyalty, "HTTPS")
  }

  Container_Boundary(DPlayback, "Playback/DRM/Entitlements") {
    Container(Ent, "Entitlements Service", "Go/Java", "Права доступа к контенту")
    Container(Playback, "Playback/Session Service", "Go/Java", "Сессии просмотра/DRM лицензии")
    Rel(Proxy, Ent, "REST /api/entitlements/*")
    Rel(Proxy, Playback, "REST /api/playback/*")
    Rel(Playback, CDN, "DRM/манифест/статик", "HTTPS/KMS")
  }

  Container_Boundary(DNotify, "Notifications & Marketing") {
    Container(Notify, "Notifications Service", "Go/Java", "Email/SMS/Push-триггеры")
    Container(Mkt, "Marketing Orchestrator", "Go/Java", "Сценарии, сегменты, кампании")
    Rel(Proxy, Notify, "REST /api/notifications/*")
    Rel(Proxy, Mkt, "REST /api/marketing/*")
    Rel(Notify, EmailSMS, "SMTP/HTTP")
    Rel(Mkt, Loyalty, "HTTPS")
  }

  Container_Boundary(DEvents, "Events/Analytics") {
    Container(Events, "Events Service", "Java • Spring Boot + Kafka", "MVP: publish/consume user|payment|movie")
    Container(Kafka, "Kafka", "Event Streaming", "Шина событий")
    Rel(Proxy, Events, "REST /api/events/*")
    Rel(Events, Kafka, "Produce/Consume")
    Rel(Kafka, Analytics, "Экспорт")
  }

  Container_Boundary(DReco, "Recommendations") {
    Container(RecoAdapter, "Reco Adapter", "Go/Java", "Запрос подборок/feature store")
    Rel(Proxy, RecoAdapter, "REST /api/recommendations/*")
    Rel(RecoAdapter, RecoSys, "HTTPS/Kafka")
  }

  ' === Наследие (что ещё не вынесли) ===
  Container_Boundary(DLegacy, "Legacy Monolith") {
    Container(Monolith, "Monolith", "Go", "Остаток функционала в миграции")
    Rel(Proxy, Monolith, "REST /api/* (fallback)")
  }

}

Rel(User, Ingress, "HTTPS")
Rel(Admin, Ingress, "HTTPS")
Rel(Marketing, Ingress, "HTTPS")

note bottom
Strangler Fig: Proxy маршрутизирует новые пути на микросервисы,
остальные — в Monolith. Доли трафика управляются флагами/конфигами.
БД постепенно делятся по доменам (сейчас Movies уже вынесен).
@enduml
