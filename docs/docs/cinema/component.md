
---

# docs/docs/cinema/component.md (уточнено: Proxy с маршрутами доменов)

```md
# C4 — Компоненты: Proxy + Events (и маршрутизация доменов)

```puml
@startuml
!include ../../c4/C4_Component.puml
LAYOUT_WITH_LEGEND()

title Proxy & Events — компоненты и маршруты

' ===== Proxy =====
Container_Boundary(PROXY, "Proxy (Spring WebFlux)") {
  Component(ProxyAPI, "HTTP Router", "WebFlux", "Единый вход: /api/**")
  Component(TS, "Traffic Shaper/FF", "ENV/ConfigMap", "MOVIES_MIGRATION_PERCENT и иные флаги")
  Component(MonCli, "MonolithClient", "WebClient", "Fallback/наследие")
  Component(MovCli, "MoviesClient", "WebClient", "Каталог/метаданные")
  Component(AuthCli, "AuthClient", "WebClient", "Auth/Users")
  Component(ProfileCli, "ProfileClient", "WebClient", "Профили")
  Component(PayCli, "PaymentsClient", "WebClient", "Платежи")
  Component(SubsCli, "SubsClient", "WebClient", "Подписки")
  Component(DiscCli, "DiscountsClient", "WebClient", "Купоны/промо")
  Component(RateCli, "RatingsClient", "WebClient", "Оценки")
  Component(FavCli, "FavoritesClient", "WebClient", "Избранное")
  Component(EntCli, "EntitlementsClient", "WebClient", "Права")
  Component(PbCli, "PlaybackClient", "WebClient", "Сессии/DRM")
  Component(NotifCli, "NotificationsClient", "WebClient", "Сообщения")
  Component(MktCli, "MarketingClient", "WebClient", "Кампании")
  Component(RecoCli, "RecoAdapterClient", "WebClient", "Подборки")
  Component(EventsCli, "EventsClient", "WebClient", "События")

  Rel(ProxyAPI, TS, "чтение флагов")
  Rel(ProxyAPI, MovCli, "/api/movies*")
  Rel(ProxyAPI, AuthCli, "/api/auth/*")
  Rel(ProxyAPI, ProfileCli, "/api/profile/*")
  Rel(ProxyAPI, PayCli, "/api/payments/*")
  Rel(ProxyAPI, SubsCli, "/api/subscriptions/*")
  Rel(ProxyAPI, DiscCli, "/api/discounts/*")
  Rel(ProxyAPI, RateCli, "/api/ratings/*")
  Rel(ProxyAPI, FavCli, "/api/favorites/*")
  Rel(ProxyAPI, EntCli, "/api/entitlements/*")
  Rel(ProxyAPI, PbCli, "/api/playback/*")
  Rel(ProxyAPI, NotifCli, "/api/notifications/*")
  Rel(ProxyAPI, MktCli, "/api/marketing/*")
  Rel(ProxyAPI, RecoCli, "/api/recommendations/*")
  Rel(ProxyAPI, EventsCli, "/api/events/*")
  Rel(ProxyAPI, MonCli, "прочие /api/* (fallback)")
}

' ===== Events =====
Container_Boundary(EVENTS, "Events (Spring + Kafka)") {
  Component(EvtAPI,  "EventsController", "HTTP", "POST /api/events/{user|payment|movie}\nGET /api/events/{type}/last")
  Component(EvtProd, "Producer", "KafkaTemplate", "publish JSON")
  Component(EvtCons, "Consumer", "@KafkaListener", "consume + keep last N in memory")
  Rel(EvtAPI, EvtProd, "produce")
}
@enduml
