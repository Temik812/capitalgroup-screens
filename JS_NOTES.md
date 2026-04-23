# Заметки по JS для верстальщика

## Экраны 3-5 — Панель «Статус регистрации»

### Что менять в main.js

Существующий вызов `openModalGlobal("denied-appeal")` открывает **центрированный** попап.
Новая верстка использует `.modal-overlay` + `.modal-broker` (правая панель).

**Замена в main.js** (~строка 20727):
```js
// БЫЛО:
openModalGlobal("denied-appeal");

// СТАЛО:
const overlay = document.getElementById('denied-appeal-overlay');
if (overlay) overlay.classList.add('show');
```

**Закрытие** (кнопка × и клик по оверлею):
```js
document.addEventListener('click', function(e) {
    const closeBtn = e.target.closest('.js-denied-appeal-close');
    const overlay = document.getElementById('denied-appeal-overlay');
    if (!overlay) return;
    if (closeBtn || e.target === overlay) {
        overlay.classList.remove('show');
    }
});
```

**Переключение состояний панели:**
```js
const panel = document.querySelector('.js-denied-appeal-panel');

// Экран 3 (отклонено, пусто):
panel.dataset.state = 'denied-empty';

// Экран 4 (отклонено, текст введён) — вешать на input textarea:
panel.dataset.state = 'denied-filled';

// Экран 5 (апелляция отправлена):
panel.dataset.state = 'appeal-sent';
// + заполнить js-denied-appeal-sent-text текстом апелляции
```

**Активация кнопки при вводе текста:**
```js
const textarea = document.querySelector('.js-denied-appeal-text');
const panel = document.querySelector('.js-denied-appeal-panel');
textarea.addEventListener('input', function() {
    panel.dataset.state = this.value.trim() ? 'denied-filled' : 'denied-empty';
});
```

---

## Экраны 1-2 — Список брокеров + модал причины отказа

### Открытие модала «Причина отказа»:
```js
document.addEventListener('click', function(e) {
    const btn = e.target.closest('.js-open-denial-modal');
    if (!btn) return;

    const overlay = document.querySelector('.js-denial-modal-overlay');
    // Заполняем данные из data-атрибутов кнопки
    overlay.querySelector('.js-denial-broker-org').textContent = btn.dataset.brokerOrg;
    overlay.querySelector('.js-denial-broker-fio').textContent = btn.dataset.brokerFio;
    overlay.querySelector('.js-denial-old-reason').textContent = btn.dataset.denialReason || '—';
    overlay.querySelector('.js-denial-reason-input').value = '';
    overlay.querySelector('.js-denial-reason-input').dataset.brokerId = btn.dataset.brokerId;

    overlay.classList.add('show');
});
```

### Закрытие:
```js
document.addEventListener('click', function(e) {
    const overlay = document.querySelector('.js-denial-modal-overlay');
    if (!overlay) return;
    if (e.target.closest('.js-denial-modal-close') || e.target === overlay) {
        overlay.classList.remove('show');
    }
});
```

### Сохранение причины отказа:
```js
document.addEventListener('click', function(e) {
    if (!e.target.closest('.js-denial-reason-save')) return;
    const input = document.querySelector('.js-denial-reason-input');
    const statusEl = document.querySelector('.js-denial-reason-status');
    const brokerId = input.dataset.brokerId;
    const reason = input.value.trim();

    if (!reason) {
        document.querySelector('.js-denial-reason-error').textContent = 'Введите причину отказа';
        return;
    }

    fetch('/local/api/v1/manager/broker/deny/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ broker_id: brokerId, reason })
    })
    .then(r => r.json())
    .then(res => {
        if (res.success) {
            // Обновить ячейку причины в таблице
            const row = document.querySelector(`[data-broker-id="${brokerId}"]`)?.closest('tr');
            if (row) row.querySelector('.broker-denial-reason').textContent = reason;
            document.querySelector('.js-denial-modal-overlay').classList.remove('show');
        }
    });
});
```

### Переключение статуса в таблице:
```js
document.addEventListener('click', function(e) {
    const option = e.target.closest('.js-status-option');
    if (!option) return;
    const dropdown = option.closest('.js-status-dropdown');
    const selected = dropdown.querySelector('.js-status-selected');
    const brokerId = dropdown.dataset.brokerId;
    const value = option.dataset.value;

    // Обновить UI
    selected.textContent = option.textContent;
    dropdown.className = `status-dropdown status-dropdown--${value} js-status-dropdown`;
    dropdown.classList.remove('open');

    // API вызов
    fetch('/local/api/v1/manager/broker/status/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ broker_id: brokerId, status: value })
    });
});
```
