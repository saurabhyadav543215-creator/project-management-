# project-management-
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Project Management Tool</title>
<style>
  :root{--bg:#f4f6fb;--card:#fff;--accent:#2563eb;--muted:#6b7280;--success:#10b981}
  *{box-sizing:border-box}
  body{font-family:Inter,Segoe UI,Arial,Helvetica,sans-serif;background:var(--bg);margin:0;color:#0f172a}
  header{background:var(--card);padding:14px 20px;display:flex;align-items:center;justify-content:space-between;box-shadow:0 4px 20px rgba(2,6,23,0.06)}
  header h1{font-size:18px;margin:0}
  .wrap{max-width:1200px;margin:20px auto;padding:0 16px}
  .toolbar{display:flex;gap:10px;align-items:center;margin:14px 0}
  input,select,textarea{padding:10px;border-radius:8px;border:1px solid #e6eef7}
  button{background:var(--accent);color:white;border:0;padding:10px 12px;border-radius:8px;cursor:pointer}
  .cols{display:grid;grid-template-columns:1fr 1fr 1fr;gap:14px;margin-top:18px}
  .col{background:var(--card);padding:12px;border-radius:10px;min-height:380px}
  .col h3{margin:0 0 8px 0}
  .task{background:#fff;padding:10px;border-radius:8px;margin-bottom:10px;box-shadow:0 6px 18px rgba(2,6,23,0.04);cursor:grab}
  .task .meta{font-size:12px;color:var(--muted);margin-top:8px;display:flex;justify-content:space-between}
  .right{display:flex;gap:8px}
  .small{font-size:13px;color:var(--muted)}
  .panel{background:var(--card);padding:12px;border-radius:10px;margin-top:14px}
  .flex{display:flex;gap:12px}
  .filters{display:flex;gap:8px;align-items:center}
  .kanban-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px}
  .badge{display:inline-block;padding:4px 8px;border-radius:999px;background:#eef2ff;color:#312e81;font-size:12px}
  .empty{color:var(--muted);padding:10px;border-radius:6px;border:1px dashed #e6eef7;text-align:center}
  footer{margin:20px 0;text-align:center;color:var(--muted)}
  @media (max-width:900px){.cols{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
  <h1>ManageFlow — Project Management</h1>
  <div class="small">Local demo • Data stored in your browser</div>
</header>

<main class="wrap">
  <div class="toolbar">
    <input id="search" placeholder="Search tasks or assignee..." />
    <select id="filterProject"><option value="">All Projects</option></select>
    <select id="filterPriority"><option value="">All Priorities</option><option value="Low">Low</option><option value="Medium">Medium</option><option value="High">High</option></select>
    <button id="newTaskBtn">+ New Task</button>
    <div style="margin-left:auto;display:flex;gap:8px">
      <button id="exportBtn" style="background:#10b981">Export</button>
      <button id="importBtn" style="background:#f59e0b">Import</button>
      <input type="file" id="importFile" accept="application/json" style="display:none" />
    </div>
  </div>

  <div class="panel">
    <div style="display:flex;gap:16px;align-items:center"><strong>Projects</strong>
      <div id="projectChips" style="display:flex;gap:8px;margin-left:12px"></div>
      <div style="margin-left:auto" class="small">Drag tasks between columns to change status.</div>
    </div>
  </div>

  <div class="cols" id="board">
    <div class="col" data-status="todo">
      <div class="kanban-header"><h3>To Do <span class="badge" id="count-todo">0</span></h3></div>
      <div class="droparea" id="todoArea"></div>
    </div>
    <div class="col" data-status="inprogress">
      <div class="kanban-header"><h3>In Progress <span class="badge" id="count-inprogress">0</span></h3></div>
      <div class="droparea" id="inprogressArea"></div>
    </div>
    <div class="col" data-status="done">
      <div class="kanban-header"><h3>Done <span class="badge" id="count-done">0</span></h3></div>
      <div class="droparea" id="doneArea"></div>
    </div>
  </div>

  <div class="panel">
    <strong>Activity / Controls</strong>
    <div style="margin-top:8px" class="small">Click any task to edit. Use export/import to share backups.</div>
    <div id="activity" style="margin-top:8px"></div>
  </div>

</main>

<!-- Task modal -->
<div id="modal" style="position:fixed;left:0;top:0;width:100%;height:100%;display:none;align-items:center;justify-content:center;background:rgba(2,6,23,0.5)">
  <div style="background:white;padding:18px;border-radius:10px;width:420px;max-width:92%">
    <h3 id="modalTitle">New Task</h3>
    <div style="display:flex;flex-direction:column;gap:8px;margin-top:8px">
      <input id="taskTitle" placeholder="Task title" />
      <textarea id="taskDesc" rows="4" placeholder="Description"></textarea>
      <div class="flex">
        <input id="taskAssignee" placeholder="Assignee" />
        <input id="taskDue" type="date" />
      </div>
      <div class="flex">
        <select id="taskPriority"><option>Low</option><option selected>Medium</option><option>High</option></select>
        <select id="taskProject"></select>
      </div>
      <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:6px">
        <button id="deleteTaskBtn" style="background:#ef4444;display:none">Delete</button>
        <button id="closeModal" style="background:#6b7280">Cancel</button>
        <button id="saveTaskBtn">Save</button>
      </div>
    </div>
  </div>
</div>

<footer class="wrap"><div class="small">ManageFlow • Demo • Local only</div></footer>

<script>
  // Simple client-side project manager
  const STORAGE_KEY = 'manageflow_v1';
  let state = loadState();
  const el = id => document.getElementById(id);

  // sample data if empty
  if(!state.projects.length && !state.tasks.length){
    state.projects.push({id: 'p1', name: 'Website'});
    state.projects.push({id: 'p2', name: 'Mobile App'});
    state.tasks.push({id:'t1', title:'Design hero section', desc:'Create hero mockups', assignee:'Aman', due:'2025-12-05', priority:'High', project:'p1', status:'todo'});
    state.tasks.push({id:'t2', title:'API auth', desc:'Implement JWT', assignee:'Riya', due:'2025-12-10', priority:'Medium', project:'p2', status:'inprogress'});
    saveState();
  }

  // DOM refs
  const board = el('board');
  const todoArea = el('todoArea'); const inprogressArea = el('inprogressArea'); const doneArea = el('doneArea');
  const projectSelect = el('taskProject'); const filterProject = el('filterProject'); const projectChips = el('projectChips');

  // render functions
  function render(){
    renderProjectOptions(); renderBoard(); renderCounts(); renderProjectChips(); renderQuizLikeFilters(); renderActivity();
  }

  function renderProjectOptions(){
    projectSelect.innerHTML = '';
    filterProject.innerHTML = '<option value="">All Projects</option>';
    state.projects.forEach(p=>{
      const opt = document.createElement('option'); opt.value = p.id; opt.textContent = p.name; projectSelect.appendChild(opt);
      const opt2 = opt.cloneNode(true); filterProject.appendChild(opt2);
    });
  }

  function renderProjectChips(){ projectChips.innerHTML=''; state.projects.forEach(p=>{ const d = document.createElement('div'); d.className='badge'; d.textContent=p.name; projectChips.appendChild(d); }); }

  function renderBoard(){
    const filters = getFilters();
    todoArea.innerHTML=''; inprogressArea.innerHTML=''; doneArea.innerHTML='';
    const filtered = state.tasks.filter(t=>{
      if(filters.search && !(t.title+ ' '+ t.desc + ' '+ t.assignee).toLowerCase().includes(filters.search)) return false;
      if(filters.project && t.project!==filters.project) return false;
      if(filters.priority && t.priority!==filters.priority) return false;
      return true;
    });
    filtered.forEach(t=>{
      const node = taskNode(t);
      if(t.status==='todo') todoArea.appendChild(node);
      if(t.status==='inprogress') inprogressArea.appendChild(node);
      if(t.status==='done') doneArea.appendChild(node);
    });
    [todoArea,inprogressArea,doneArea].forEach(a=>{ if(!a.childElementCount) a.innerHTML='<div class="empty">No tasks</div>' });
  }

  function renderCounts(){ el('count-todo').textContent = state.tasks.filter(t=>t.status==='todo').length; el('count-inprogress').textContent = state.tasks.filter(t=>t.status==='inprogress').length; el('count-done').textContent = state.tasks.filter(t=>t.status==='done').length; }

  function renderQuizLikeFilters(){ /* placeholder if needed */ }

  function renderActivity(){ const act = el('activity'); act.innerHTML=''; const last = state.activity.slice(-10).reverse(); last.forEach(a=>{ const d = document.createElement('div'); d.textContent = a; d.className='small'; act.appendChild(d); }); }

  function taskNode(t){
    const div = document.createElement('div'); div.className='task'; div.draggable = true; div.dataset.id = t.id;
    div.innerHTML = `<div><strong>${t.title}</strong></div><div class="small">${t.assignee || 'Unassigned'} • ${t.due || ''}</div><div class="meta"><div class="small">${projectName(t.project)}</div><div class="right"><div class="small">${t.priority}</div><button data-edit='${t.id}' style='background:transparent;border:0;color:var(--accent);cursor:pointer'>Edit</button></div></div>`;
    // drag handlers
    div.addEventListener('dragstart', (ev)=>{ ev.dataTransfer.setData('text/plain', t.id); setTimeout(()=>div.style.display='none',0); });
    div.addEventListener('dragend', ()=>{ div.style.display='block' });
    // edit handler
    div.querySelector('[data-edit]')?.addEventListener('click', ()=> openEditModal(t.id));
    return div;
  }

  // utilities
  function uuid(prefix='id'){ return prefix + Math.random().toString(36).slice(2,9); }
  function projectName(id){ const p = state.projects.find(x=>x.id===id); return p? p.name : 'No Project'; }

  // modal controls
  const modal = el('modal'); const modalTitle = el('modalTitle'); const taskTitle = el('taskTitle'); const taskDesc = el('taskDesc'); const taskAssignee = el('taskAssignee'); const taskDue = el('taskDue'); const taskPriority = el('taskPriority'); const deleteTaskBtn = el('deleteTaskBtn'); const closeModal = el('closeModal'); const saveTaskBtn = el('saveTaskBtn');
  let editingId = null;

  el('newTaskBtn').addEventListener('click', ()=>{ openNewModal(); });
  closeModal.addEventListener('click', ()=>{ modal.style.display='none'; editingId=null; deleteTaskBtn.style.display='none'; });

  function openNewModal(){ modalTitle.textContent='New Task'; taskTitle.value=''; taskDesc.value=''; taskAssignee.value=''; taskDue.value=''; taskPriority.value='Medium'; projectSelect.value = state.projects[0]?.id || ''; modal.style.display='flex'; deleteTaskBtn.style.display='none'; }

  function openEditModal(id){ const t = state.tasks.find(x=>x.id===id); if(!t) return; editingId = id; modalTitle.textContent='Edit Task'; taskTitle.value = t.title; taskDesc.value = t.desc; taskAssignee.value = t.assignee; taskDue.value = t.due || ''; taskPriority.value = t.priority || 'Medium'; projectSelect.value = t.project || state.projects[0]?.id || ''; modal.style.display='flex'; deleteTaskBtn.style.display='inline-block'; }

  saveTaskBtn.addEventListener('click', ()=>{
    const obj = { title:taskTitle.value.trim(), desc:taskDesc.value.trim(), assignee:taskAssignee.value.trim(), due:taskDue.value || '', priority:taskPriority.value, project:projectSelect.value || '' };
    if(!obj.title){ alert('Title required'); return }
    if(editingId){ const t = state.tasks.find(x=>x.id===editingId); Object.assign(t, obj); logActivity(`Edited task: ${t.title}`); } else { const id = uuid('t'); state.tasks.push({id, ...obj, status:'todo'}); logActivity(`Created: ${obj.title}`); }
    saveState(); modal.style.display='none'; editingId=null; deleteTaskBtn.style.display='none'; render();
  });

  deleteTaskBtn.addEventListener('click', ()=>{
    if(!editingId) return; if(!confirm('Delete task?')) return; const idx = state.tasks.findIndex(x=>x.id===editingId); if(idx>=0){ logActivity(`Deleted: ${state.tasks[idx].title}`); state.tasks.splice(idx,1); saveState(); render(); modal.style.display='none'; editingId=null; }
  });

  // drag & drop on columns
  document.querySelectorAll('.droparea').forEach(area=>{
    area.addEventListener('dragover', (ev)=>{ ev.preventDefault(); area.style.background = '#f8fafc'; });
    area.addEventListener('dragleave', ()=>{ area.style.background = 'transparent'; });
    area.addEventListener('drop', (ev)=>{
      ev.preventDefault(); area.style.background='transparent'; const id = ev.dataTransfer.getData('text/plain'); const t = state.tasks.find(x=>x.id===id); if(t){ t.status = area.parentElement.dataset.status; saveState(); logActivity(`Moved: ${t.title} -> ${t.status}`); render(); }
    });
  });

  // search & filters
  el('search').addEventListener('input', ()=> render());
  el('filterProject').addEventListener('change', ()=> render());
  el('filterPriority').addEventListener('change', ()=> render());

  function getFilters(){ return { search: el('search').value.trim().toLowerCase(), project: el('filterProject').value, priority: el('filterPriority').value } }

  // export/import
  el('exportBtn').addEventListener('click', ()=>{ const data = JSON.stringify(state, null, 2); const blob = new Blob([data],{type:'application/json'}); const url = URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='manageflow_export.json'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url); });
  el('importBtn').addEventListener('click', ()=> el('importFile').click());
  el('importFile').addEventListener('change', (ev)=>{ const f = ev.target.files[0]; if(!f) return; const r = new FileReader(); r.onload = e=>{ try{ const imported = JSON.parse(e.target.result); if(imported && imported.tasks && imported.projects){ state = imported; saveState(); render(); alert('Import successful'); } else alert('Invalid file'); }catch(err){ alert('Error reading file'); } }; r.readAsText(f); });

  // Projects management shortcuts (double-click badge to add project)
  projectChips.addEventListener('dblclick', ()=>{ const name = prompt('New project name'); if(name){ const id = uuid('p'); state.projects.push({id,name}); saveState(); render(); } });

  // helper: log activity
  function logActivity(text){ state.activity.push(`${new Date().toLocaleString()}: ${text}`); if(state.activity.length>200) state.activity.shift(); saveState(); }

  // persistence
  function loadState(){ try{ const raw = localStorage.getItem(STORAGE_KEY); if(raw) return JSON.parse(raw); }catch(e){} return { projects:[], tasks:[], activity:[] }; }
  function saveState(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); }

  // initial render
  render();

  // expose save for dev
  window.saveManageFlow = saveState;
</script>
</body>
</html>
