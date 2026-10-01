# tutorhub-cron

Cron de [TutorHub](https://github.com/Bcintolesim/IIC3143-TutorHub): cada 5 minutos llama a `POST /internal/tick` de la API en Render, que procesa los trabajos pendientes (cortes de sesiones, reservas vencidas, conciliación de pagos, devoluciones y correos).

Es un repo público aparte porque en repos públicos los minutos de GitHub Actions son gratis (un cron cada 5 minutos gasta unos 8.600 minutos al mes). No contiene código ni secretos: la URL y el secreto están en los secrets del repo.

## Secrets

| Secret | Valor |
|---|---|
| `TICK_URL` | `https://tutorhub-api-7rw6.onrender.com/internal/tick` |
| `CRON_SECRET` | El mismo valor que la variable `CRON_SECRET` del servicio en Render |

## Uso

- Corre solo según el `schedule` del workflow. GitHub puede atrasarlo varios minutos en horas de carga; la API corrige el estado igual (evaluación perezosa).
- Para correrlo a mano (por ejemplo en la demo): Actions → tick → Run workflow.
- Si responde 401, el `CRON_SECRET` no coincide con el de Render. Si responde 500 con "Jobs overdue", hay trabajos atrasados más de 30 minutos en la tabla `job`.
- GitHub desactiva los workflows programados tras 60 días sin actividad en el repo: hagan un commit una vez al mes o reactívenlo desde Actions.

El workflow se mantiene en el repo principal (`cron/tick.yml`); si cambia allá, hay que copiarlo aquí.
