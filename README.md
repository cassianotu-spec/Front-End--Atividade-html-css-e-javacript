(function () {
  'use strict';


  const $ = (sel, root = document) => root.querySelector(sel);

  const Templates = {
    home: () => `
      <section class="hero p-5 text-white rounded-3">
        <div class="container">
          <h1 class="display-5 fw-bold">Cadastro de Voluntários</h1>
          <p class="lead">Junte-se a nós! Cadastre-se como voluntário para participar de ações sociais, eventos e projetos comunitários.</p>

          <div class="alert alert-warning" role="alert">
            <strong>Atenção:</strong> Estamos recebendo inscrições para novos voluntários. Preencha o formulário abaixo para se inscrever — entraremos em contato em até 7 dias úteis.
          </div>

          <p class="mt-3">
            <a href="#form" class="btn btn-lg btn-light">Inscrever-se</a>
            <a href="#data" class="btn btn-lg btn-outline-light ms-2">Ver inscrições</a>
          </p>
        </div>
      </section>
    `,

    form: (data = {}) => `
      <div class="card">
        <div class="card-body">
          <h3 class="card-title">Formulário de contato</h3>
          <form id="mainForm" novalidate>
            <input type="hidden" name="id" value="${escape(data.id||'')}">
            <div class="mb-3">
              <label class="form-label">Nome *</label>
              <input name="name" class="form-control" value="${escape(data.name||'')}">
              <div class="invalid-feedback"></div>
            </div>

            <div class="mb-3">
              <label class="form-label">Email *</label>
              <input name="email" class="form-control" value="${escape(data.email||'')}">
              <div class="invalid-feedback"></div>
            </div>

            <div class="mb-3">
              <label class="form-label">Idade</label>
              <input name="age" type="number" class="form-control" value="${escape(data.age||'')}">
              <div class="invalid-feedback"></div>
            </div>

            <div class="mb-3">
              <label class="form-label">Mensagem *</label>
              <textarea name="message" class="form-control" rows="4">${escape(data.message||'')}</textarea>
              <div class="invalid-feedback"></div>
            </div>

            <div class="d-flex gap-2">
              <button class="btn btn-primary" type="submit">Enviar</button>
              <button class="btn btn-secondary" type="button" id="btnClear">Limpar</button>
            </div>
          </form>
        </div>
      </div>
    `,

    dataList: (items) => `
      <div>
        <h3>Entradas salvas (<span id="count">${items.length}</span>)</h3>
        ${items.length === 0 ? '<p class="text-muted">Nenhuma entrada salva.</p>' : ''}
        <div class="list-group">
          ${items.map((it, i) => `
            <div class="list-group-item d-flex justify-content-between align-items-start">
              <div>
                <h5 class="mb-1">${escape(it.name)} <small class="text-muted">(${escape(it.email)})</small></h5>
                <p class="mb-1">${escape(it.message)}</p>
                <small class="text-muted">Idade: ${escape(it.age||'n/a')}</small>
              </div>
              <div>
                <button class="btn btn-sm btn-outline-primary me-2" data-act="edit" data-i="${i}">Editar</button>
                <button class="btn btn-sm btn-outline-danger" data-act="delete" data-i="${i}">Excluir</button>
              </div>
            </div>
          `).join('')}
        </div>
      </div>
    `,
  };

  function escape(txt) {
    if (txt == null) return '';
    return String(txt)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
  }
  const STORAGE_KEY = 'fe3_submissions';
  function loadItems() {
    try {
      return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
    } catch (e) {
      console.warn('Erro lendo localStorage', e);
      return [];
    }
  }
  function saveItems(items) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
  }


  function navigateTo(route, data) {

    if (data) {

      render(route, data);
      return;
    }
    location.hash = route;
  }

  function getRoute() {
    return location.hash.replace('#', '') || 'home';
  }

  
  function showToast(message, type = 'success') {
    const toastEl = document.getElementById('appToast');
    const toastBody = document.getElementById('appToastBody');
    if (!toastEl || !toastBody) {
      // fallback
      alert(message);
      return;
    }
    toastBody.textContent = message;
    // ajustar classe conforme tipo (suporta success, warning, danger, info)
    toastEl.classList.remove('text-bg-success', 'text-bg-warning', 'text-bg-danger', 'text-bg-info');
    const map = { success: 'text-bg-success', warning: 'text-bg-warning', danger: 'text-bg-danger', info: 'text-bg-info' };
    toastEl.classList.add(map[type] || map.success);
    const toast = new bootstrap.Toast(toastEl, { delay: 3500 });
    toast.show();
  }

  const root = document.getElementById('app');
  function render(route, data) {
    let html = '';
    if (route === 'home') html = Templates.home();
    else if (route === 'form') html = Templates.form(data);
    else if (route === 'data') html = Templates.dataList(loadItems());
    else html = '<p>Página não encontrada</p>';

    root.innerHTML = html;
    bindPage(route);
  }

  function validateForm(form) {
    const values = {};
    const errs = {};
    const name = form.name.value.trim();
    const email = form.email.value.trim();
    const age = form.age.value.trim();
    const message = form.message.value.trim();

    if (!name) errs.name = 'Nome é obrigatório.';
    if (!email) errs.email = 'Email é obrigatório.';
    else if (!/^\S+@\S+\.\S+$/.test(email)) errs.email = 'Email inválido.';
    if (age) {
      const n = Number(age);
      if (!Number.isFinite(n) || n < 1 || n > 120) errs.age = 'Idade deve ser um número entre 1 e 120.';
    }
    if (!message || message.length < 5) errs.message = 'Mensagem precisa ter ao menos 5 caracteres.';

    values.name = name; values.email = email; values.age = age; values.message = message;
    return { values, errs };
  }

  function showValidation(form, errs) {

    form.querySelectorAll('.is-invalid').forEach(el => el.classList.remove('is-invalid'));
    form.querySelectorAll('.invalid-feedback').forEach(el => el.textContent = '');

    Object.keys(errs).forEach(key => {
      const input = form[key];
      if (input) {
        input.classList.add('is-invalid');
        const msg = input.parentElement.querySelector('.invalid-feedback');
        if (msg) msg.textContent = errs[key];
      }
    });

  }

  function bindPage(route) {
    if (route === 'form') {
      const form = document.getElementById('mainForm');
      const btnClear = document.getElementById('btnClear');
      form.addEventListener('submit', function (ev) {
        ev.preventDefault();
        const { values, errs } = validateForm(form);
        if (Object.keys(errs).length > 0) {
          showValidation(form, errs);
          const first = form.querySelector('.is-invalid');
          if (first) first.focus();
          return;
        }

        
        values.id = form.id ? form.id.value.trim() : '';
        const items = loadItems();
        if (values.id) {
          const idx = items.findIndex(it => String(it.id) === String(values.id));
          if (idx > -1) items[idx] = values;
          else items.push(values);
        } else {
          values.id = Date.now().toString();
          items.push(values);
        }
        saveItems(items);
        showToast('Entrada salva com sucesso!', 'success');
        navigateTo('data');
      });

      btnClear.addEventListener('click', function () {
        form.reset();
        form.querySelectorAll('.is-invalid').forEach(el => el.classList.remove('is-invalid'));
        form.querySelectorAll('.invalid-feedback').forEach(el => el.textContent = '');
      });
    }

    if (route === 'data') {
      const container = root;
      container.addEventListener('click', function (ev) {
        const btn = ev.target.closest('button');
        if (!btn) return;
        const act = btn.dataset.act;
        const i = Number(btn.dataset.i);
        const items = loadItems();
        if (act === 'delete') {
          if (!confirm('Confirmar exclusão?')) return;
          items.splice(i, 1);
          saveItems(items);
          render('data');
        } else if (act === 'edit') {
          const item = items[i];
          render('form', item);
        }
      });
    }
  }

  function init() {

    render(getRoute());
    window.addEventListener('hashchange', () => render(getRoute()));
  }

  document.addEventListener('DOMContentLoaded', init);

})();
