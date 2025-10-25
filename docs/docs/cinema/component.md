
---

### ✅ `docs/docs/cinema/component.md`

```md
# C4 — Компонентная диаграмма Proxy + Events

```puml
@startuml
!include ../../c4/C4_Component.puml
LAYOUT_WITH_LEGEND()

title Proxy & Events — компоненты

Container_Boundary(PROXY, "Proxy") {
  Component(ProxyAPI, "HTTP Router", "WebFlux", "/api/** маршрутизация")
  Component(Flag, "Traffic Shaper", "ConfigMap", "MOVIES_MIGRATION_PERCENT")
  Component(MonClient, "MonolithClient", "WebClient")
  Component(MovClient, "MoviesClient", "WebClient")
  Rel(ProxyAPI, Flag, "Чтение процента маршрутизации")
  Rel(ProxyAPI, MovClient, "/api/movies*")
  Rel(ProxyAPI, MonClient, "остальные пути /api/")
}

Container_Boundary(EVENTS, "Events") {
  Component(EvtAPI, "EventsController", "HTTP", "POST/GET события")
  Component(EvtProd, "Producer", "KafkaTemplate")
  Component(EvtCons, "Consumer", "@KafkaListener")
  Rel(EvtAPI, EvtProd, "produce")
  Rel(EvtCons, EvtAPI, "GET /last возвращает in-memory")
}

@enduml
