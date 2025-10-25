
---

### ✅ `docs/docs/cinema/code.md`  (самая детальная)

```md
# C4 — Диаграмма кода (Proxy + Events)

```puml
@startuml
!include ../../c4/C4_Component.puml
title Код Proxy-Service и Events-Service

' ===== Proxy-Service =====
package "cinemaabyss-proxy-service" {
  class ProxyApplication {
    + main(args: String[]): void
  }

  class ProxyHandler {
    - wc: WebClient
    - monoBase: String <<ENV MONOLITH_BASE_URL>>
    - moviesBase: String <<ENV MOVIES_BASE_URL>>
    - migrationPercent: int <<ENV MOVIES_MIGRATION_PERCENT>>
    + proxy(exchange: ServerWebExchange): Mono<Void>
    .. Strangler logic ..
    - selectBackend(path: String): String
    - forwardTo(url: String): Mono<Void>
    - addResponseHeader("X-Proxy-Target")
  }
}

' ===== Events-Service =====
package "events-service" {
  class EventsApplication {
    + main(args: String[]): void
  }

  class EventsController {
    + createUserEvent(..): Mono<Void>    <<POST /api/events/user>>
    + createPaymentEvent(..): Mono<Void> <<POST /api/events/payment>>
    + createMovieEvent(..): Mono<Void>   <<POST /api/events/movie>>
    + getLastEventsByType(t: String): List<String> <<GET /api/events/{type}/last>>
  }

  class EventsProducer {
    + send(topic: String, payload: Map): Mono<Void>
  }

  class EventsConsumer {
    - last: Map<String,Deque<String>> (cap=20)
    + get(type: String): Deque<String>
    + @KafkaListener user-events
    + @KafkaListener payment-events
    + @KafkaListener movie-events
  }
}

@enduml
