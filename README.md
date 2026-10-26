# tutorhub-cron

Cron **de respaldo** de [TutorHub](https://github.com/Bcintolesim/IIC3143-TutorHub) ([app](https://tutorhub-two.vercel.app)): llama a `POST /internal/tick` de la API en Render, que procesa los trabajos pendientes (cortes de sesiones, reservas vencidas, conciliación de pagos, devoluciones y correos).

Es un repo público aparte porque en repos públicos los minutos de GitHub Actions son gratis (un cron cada 5 minutos gasta unos 8.600 minutos al mes). No contiene código ni secretos: la URL y el secreto están en los secrets del repo.

## Principal y respaldo

El cron principal es **cron-job.org**, que llama al tick cada 5 minutos a la hora. GitHub corre los workflows programados "cuando puede" (en octubre de 2026 llegaban cada 2 a 4 horas), así que este repo es el respaldo: si cron-job.org se cae, la cola sigue avanzando y un trabajo atrasado más de 30 minutos hace fallar el workflow y manda un correo. Los dos pueden correr a la vez porque el tick usa `FOR UPDATE SKIP LOCKED`.

## Secrets

| Secret | Valor |
|---|---|
| `TICK_URL` | `https://tutorhub-api-7rw6.onrender.com/internal/tick` |
| `CRON_SECRET` | El mismo valor que la variable `CRON_SECRET` del servicio en Render |

## Uso

- Corre solo según el `schedule` del workflow. GitHub puede atrasarlo varios minutos en horas de carga; la API corrige el estado igual (evaluación perezosa).
- Para correrlo a mano (por ejemplo en la demo): Actions → tick → Run workflow.
- Si responde 401, el `CRON_SECRET` no coincide con el de Render. Si responde 500 con "Jobs overdue", hay trabajos atrasados más de 30 minutos en la tabla `job`.
- GitHub desactiva los workflows programados tras 60 días sin actividad en el repo. `keepalive.yml` corre cada lunes y hace un commit vacío si el último commit tiene más de 45 días. Si igual aparece desactivado: Actions → tick → Enable workflow.

Los workflows se mantienen en el repo principal (`cron/tick.yml` y `cron/keepalive.yml`); si cambian allá, hay que copiarlos aquí.
