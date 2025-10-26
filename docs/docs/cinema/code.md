
---

# docs/docs/cinema/code.md (актуально к проекту; добавил небольшие пояснения)

```md
# C4 — Диаграмма кода (соответствует текущему проекту)

```puml
@startuml
!include ../../c4/C4_Component.puml
title Код Proxy-Service и Events-Service

' ===== Proxy-Service (Spring WebFlux) =====
package "cinemaabyss-proxy-service" {
  class ProxyApplication { + main(args: String[]): void }

  class ProxyHandler {
    - wc: WebClient
    - monoBase: String        <<ENV MONOLITH_BASE_URL>>
    - moviesBase: String      <<ENV MOVIES_BASE_URL>>
    - migrationPercent: int   <<ENV MOVIES_MIGRATION_PERCENT>>
    + proxy(exchange: ServerWebExchange): Mono<Void> <<@RequestMapping("/api/**")>>
    .. internals ..
    - selectBackend(path: String): String  ' /api/movies* → % на movies vs monolith
    - forwardTo(url: String, exchange): Mono<Void>
    - addResponseHeader("X-Proxy-Target")
  }
}

' ===== Events-Service (Spring Boot + Kafka) =====
package "events-service" {
  class EventsApplication { + main(args: String[]): void }

  class EventsController {
    + createUserEvent(body: Map): Mono<Void>    <<POST /api/events/user>>
    + createPaymentEvent(body: Map): Mono<Void> <<POST /api/events/payment>>
    + createMovieEvent(body: Map): Mono<Void>   <<POST /api/events/movie>>
    + getLastEventsByType(type: String): List<String> <<GET /api/events/{type}/last>>
  }

  class EventsProducer { + send(topic: String, payload: Map): Mono<Void> }
  class EventsConsumer {
    - last: Map<String,Deque<String>> (cap=20)
    + get(type: String): Deque<String>
    + onUser(msg: String)    <<@KafkaListener("user-events")>>
    + onPayment(msg: String) <<@KafkaListener("payment-events")>>
    + onMovie(msg: String)   <<@KafkaListener("movie-events")>>
  }
}

@enduml
