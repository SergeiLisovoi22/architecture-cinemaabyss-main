
---

### ✅ `docs/docs/cinema/container.md`  (главная To-Be диаграмма)

```md
# C4 — Диаграмма контейнеров (To-Be)

```puml
@startuml
!include ../../c4/C4_Container.puml
LAYOUT_WITH_LEGEND()
top to bottom direction

title Кинобездна — Контейнерная диаграмма (To-Be)

Person(User, "Зритель")
Person(Admin, "Оператор")

System_Boundary(K8S, "Kubernetes") {

  Container(Ingress, "Ingress", "NGINX", "Единая точка входа HTTPS")
  Container(Proxy, "Proxy-Service", "Java • Spring WebFlux", "Стратегия Strangler Fig, Feature-flag routing")
  Rel(Ingress, Proxy, "HTTPS")

  Container(Movies, "Movies-Service", "Go/Node (MVP)", "/api/movies — новый микросервис")
  Container(Monolith, "Monolith", "Go", "Остальной функционал")
  Rel(Proxy, Movies, "REST /api/movies*")
  Rel(Proxy, Monolith, "REST /api/*")

  Container(Events, "Events-Service", "Java • Kafka", "Produce/Consume user|payment|movie events")
  Container(Kafka, "Kafka", "Event Streaming", "Шина событий")
  Rel(Events, Kafka, "Publish/Consume")

  ContainerDb(DB, "PostgreSQL", "DB", "Каталог/рейтинги (времен. общая)")
  Rel(Movies, DB, "SQL")
  Rel(Monolith, DB, "SQL")

}

Rel(User, Ingress, "HTTPS")
Rel(Admin, Ingress, "HTTPS")

@enduml
