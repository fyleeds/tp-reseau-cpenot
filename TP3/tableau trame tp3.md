
| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP |10.3.2.12          | `marcel` `08:00:27:8c:ee:8f` |    10.3.2.0          | `Broadcast`, `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | 10.3.2.254         |`router` `08:00:27:8c:ee:8f`                    | 10.3.2.12              | `marcel` `08:00:27:8c:ee:8f`    |
| 3     | Ping        | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`                       | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`
| 4     | Pong        | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`  | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`
| 5   | Requête ARP         | 10.3.2.12       | `marcel` `08:00:27:12:c8:b1`                     |   10.3.2.254             |        `router`     `08:00:27:8c:ee:8f`                ||
| 6  | Réponse ARP         | 10.3.2.254       | `router` `08:00:27:8c:ee:8f`                     |    10.3.2.12            |       `marcel` `08:00:27:12:c8:b1`                      |
| 7     | Ping        | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`                       | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`
| 8     | Pong        | 10.3.2.12              | `marcel` `08:00:27:12:c8:b1`  | 10.3.2.254          | `router` `08:00:27:8c:ee:8f`
|