# Планы развития

Здесь хранятся **долгосрочные и предметные планы** развития ERP (доменная модель, этапы внедрения, соглашения по именованию). Документы не заменяют код и миграции — к ним возвращаются при приоритизации задач.

| Документ | Суть |
| -------- | ---- |
| [tenant-external-credentials-plan.md](tenant-external-credentials-plan.md) | **Этап 1.** Хранение внешних credentials по компании (мульти-тенант, в т.ч. LLM API keys), зашифрованно в БД; основа для оплаты провайдера со стороны тенанта |
| [ai-assistent-plan.md](ai-assistent-plan.md) | **Этап 2 (мастер).** AI-платформа: `App\Services\AI`, таблицы `ai_*`, skills в БД, gateway, фазы A–F |
| [ai-leads-skills-plan.md](ai-leads-skills-plan.md) | **Запланировано.** AI skills для лидов: подсчёт на дату, поиск активного лида (имя/телефон/адрес) |
| [internal-messaging-chat-plan.md](internal-messaging-chat-plan.md) | **Этап 2 (UI).** Drawer-чат с ассистентом; срез фазы A мастер-плана; таблицы `ai_conversations` / `ai_messages` |
| [leads-and-sale-order-material.md](leads-and-sale-order-material.md) | Лиды, заказ продаж (`SaleOrder`), строки (`SaleOrderLine`), ядро `Material` / каталог `Product` |
| [portfolio-mvp-plan.md](portfolio-mvp-plan.md) | Проект и трек исполнения для модуля Portfolio (`Portfolio` -> `PortfolioItem`) с жесткой backend/frontend консистентностью |
