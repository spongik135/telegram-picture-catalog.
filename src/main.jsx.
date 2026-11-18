import React, { useEffect, useMemo, useState } from "react";
import { createRoot } from "react-dom/client";
import { createClient } from "@supabase/supabase-js";
import { ArrowLeft, Search, Plus, Pencil, Trash2, ImagePlus, X, LockKeyhole, LogOut, ChevronRight, FolderOpen } from "lucide-react";
import "./style.css";

const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL || "";
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_ANON_KEY || "";
const supabase = SUPABASE_URL && SUPABASE_ANON_KEY ? createClient(SUPABASE_URL, SUPABASE_ANON_KEY) : null;

const INITIAL_SECTIONS = [
  "STARDEW VALLEY", "MrLololoshka", "Не фандомное", "CookieRun",
  "Интерактивы", "Стримеры", "Песенки", "Cuphead",
  "Семья шпионов", "Эхо террора", "Genshin Impact", "Три кота"
];

function App() {
  const tg = window.Telegram?.WebApp;
  const [sections, setSections] = useState([]);
  const [characters, setCharacters] = useState([]);
  const [section, setSection] = useState(null);
  const [character, setCharacter] = useState(null);
  const [query, setQuery] = useState("");
  const [admin, setAdmin] = useState(false);
  const [session, setSession] = useState(null);
  const [notice, setNotice] = useState("");
  const [loading, setLoading] = useState(true);
  const [modal, setModal] = useState(null);

  useEffect(() => {
    tg?.ready();
    tg?.expand();
    tg?.setHeaderColor?.("#f7f4ff");
    tg?.setBackgroundColor?.("#f7f4ff");
    loadData();
    if (supabase) {
      supabase.auth.getSession().then(({ data }) => setSession(data.session));
      const { data: listener } = supabase.auth.onAuthStateChange((_event, s) => setSession(s));
      return () => listener.subscription.unsubscribe();
    }
  }, []);

  useEffect(() => {
    let active = true;
    async function verifyAdmin() {
      if (!session?.user?.id || !supabase) { setAdmin(false); return; }
      const { data, error } = await supabase.from("admin_users").select("user_id").eq("user_id", session.user.id).maybeSingle();
      if (active) setAdmin(Boolean(data) && !error);
    }
    verifyAdmin();
    return () => { active = false; };
  }, [session]);

  async function loadData() {
    setLoading(true);
    if (!supabase) {
      setSections(INITIAL_SECTIONS.map((name, i) => ({ id: `demo-${i}`, name, sort_order: i })));
      setCharacters([]);
      setLoading(false);
      return;
    }
    const [{ data: s, error: se }, { data: c, error: ce }] = await Promise.all([
      supabase.from("sections").select("*").order("sort_order"),
      supabase.from("characters").select("*").order("sort_order")
    ]);
    if (se || ce) setNotice("Не удалось загрузить каталог. Проверь настройки Supabase.");
    setSections(s || []);
    setCharacters(c || []);
    setLoading(false);
  }

  const isConfigured = Boolean(supabase);
  const isAdmin = Boolean(session) && admin;
  const visibleCharacters = useMemo(() => characters.filter(c =>
    c.section_id === section?.id && c.name.toLowerCase().includes(query.toLowerCase())
  ), [characters, section, query]);

  function goBack() {
    if (character) setCharacter(null);
    else if (section) { setSection(null); setQuery(""); }
    else setAdmin(false);
  }

  async function signIn(email, password) {
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    if (error) { setNotice("Не удалось войти: проверь почту и пароль."); return; }
    setModal(null);
    setNotice("Проверяем права администратора…");
  }

  async function signOut() {
    await supabase.auth.signOut();
    setAdmin(false);
  }

  async function saveSection(name, id) {
    const clean = name.trim();
    if (!clean) return;
    const payload = { name: clean };
    const result = id
      ? await supabase.from("sections").update(payload).eq("id", id)
      : await supabase.from("sections").insert({ ...payload, sort_order: sections.length });
    if (result.error) setNotice("Не получилось сохранить раздел. Проверь права администратора.");
    else { setModal(null); await loadData(); }
  }

  async function deleteSection(item) {
    if (!confirm(`Удалить раздел «${item.name}» и его персонажей?`)) return;
    const { error } = await supabase.from("sections").delete().eq("id", item.id);
    if (error) setNotice("Не удалось удалить раздел.");
    else { setSection(null); await loadData(); }
  }

  async function saveCharacter({ name, id, sectionId, file, oldImage }) {
    let image_url = oldImage || null;
    if (file) {
      const ext = file.name.split(".").pop()?.toLowerCase() || "jpg";
      const path = `${crypto.randomUUID()}.${ext}`;
      const { error: uploadError } = await supabase.storage.from("catalog-images").upload(path, file, { upsert: false });
      if (uploadError) { setNotice("Не удалось загрузить картинку. Проверь хранилище."); return; }
      const { data } = supabase.storage.from("catalog-images").getPublicUrl(path);
      image_url = data.publicUrl;
    }
    const payload = { name: name.trim(), section_id: sectionId, image_url };
    const result = id
      ? await supabase.from("characters").update(payload).eq("id", id)
      : await supabase.from("characters").insert({ ...payload, sort_order: characters.filter(c => c.section_id === sectionId).length });
    if (result.error) setNotice("Не получилось сохранить персонажа.");
    else { setModal(null); await loadData(); }
  }

  async function deleteCharacter(item) {
    if (!confirm(`Удалить персонажа «${item.name}»?`)) return;
    const { error } = await supabase.from("characters").delete().eq("id", item.id);
    if (error) setNotice("Не удалось удалить персонажа.");
    else { setCharacter(null); await loadData(); }
  }

  const backButton = (label) => <button className="back" onClick={goBack}><ArrowLeft size={18}/>{label}</button>;

  return <div className="app-shell">
    <header className="topbar">
      <div className="brand"><span className="brand-mark">✦</span><span>Моя коллекция</span></div>
      <button className="icon-button" aria-label="Администрирование" onClick={() => setModal({ type: "login" })}><LockKeyhole size={18}/></button>
    </header>

    {notice && <div className="notice" onClick={() => setNotice("")}>{notice}<X size={15}/></div>}

    {!isConfigured && <div className="setup-banner">
      <b>Демонстрационный режим</b>
      <span>Пока показаны твои 12 разделов. Для сохранения картинок и изменений нужно подключить базу данных по инструкции в README.</span>
    </div>}

    {character ? <main className="content">
      {backButton(section?.name || "Персонажи")}
      <div className="detail-card">
        {character.image_url ? <img className="detail-image" src={character.image_url} alt={character.name}/> : <div className="empty-image large"><ImagePlus size={36}/><span>Изображение ещё не добавлено</span></div>}
        <h1>{character.name}</h1>
        {isAdmin && <div className="admin-actions">
          <button className="secondary-button" onClick={() => setModal({ type: "character", item: character, sectionId: section.id })}><Pencil size={16}/>Изменить</button>
          <button className="danger-button" onClick={() => deleteCharacter(character)}><Trash2 size={16}/>Удалить</button>
        </div>}
      </div>
    </main> : section ? <main className="content">
      {backButton("Все разделы")}
      <div className="section-heading"><div><p className="eyebrow">РАЗДЕЛ</p><h1>{section.name}</h1></div>
        {isAdmin && <button className="primary-button compact" onClick={() => setModal({ type: "character", sectionId: section.id })}><Plus size={17}/>Добавить</button>}
      </div>
      <label className="search"><Search size={18}/><input value={query} onChange={e => setQuery(e.target.value)} placeholder="Найти персонажа..."/></label>
      {visibleCharacters.length ? <div className="character-list">{visibleCharacters.map(item =>
        <button className="character-row" key={item.id} onClick={() => setCharacter(item)}>
          {item.image_url ? <img src={item.image_url} alt="" /> : <span className="avatar-placeholder">✦</span>}
          <span>{item.name}</span><ChevronRight size={18}/>
        </button>)}</div> : <div className="empty-state"><FolderOpen size={30}/><b>{query ? "Ничего не найдено" : "Пока здесь пусто"}</b><span>{isAdmin ? "Добавь первого персонажа кнопкой выше." : "Персонажи появятся здесь, когда их добавит администратор."}</span></div>}
    </main> : <main className="content">
      <div className="welcome"><p className="eyebrow">ТВОЯ ГАЛЕРЕЯ</p><h1>Выбери раздел</h1><p>Найди любимых персонажей и открой их изображения.</p></div>
      {isAdmin && <div className="admin-strip"><span>Режим администратора</span><button onClick={signOut}><LogOut size={15}/>Выйти</button></div>}
      {isAdmin && <button className="add-section" onClick={() => setModal({ type: "section" })}><Plus size={18}/>Добавить раздел</button>}
      <div className="section-grid">{sections.map((item, index) =>
        <div className="section-tile-wrap" key={item.id}>
          <button className="section-tile" onClick={() => setSection(item)}>
            <span className="tile-number">{String(index + 1).padStart(2, "0")}</span>
            <span className="tile-name">{item.name}</span><ChevronRight size={18}/>
          </button>
          {isAdmin && <div className="tile-tools">
            <button aria-label="Переименовать" onClick={() => setModal({ type: "section", item })}><Pencil size={14}/></button>
            <button aria-label="Удалить" onClick={() => deleteSection(item)}><Trash2 size={14}/></button>
          </div>}
        </div>)}</div>
      {loading && <p className="loading">Загружаем каталог…</p>}
    </main>}

    {modal?.type === "login" && <LoginModal onClose={() => setModal(null)} onLogin={signIn} configured={isConfigured}/>}
    {modal?.type === "section" && <SectionModal item={modal.item} onClose={() => setModal(null)} onSave={saveSection}/>}
    {modal?.type === "character" && <CharacterModal item={modal.item} sectionId={modal.sectionId} onClose={() => setModal(null)} onSave={saveCharacter}/>}
    <footer>Сделано для твоей коллекции · <span>✦</span></footer>
  </div>;
}

function ModalFrame({ title, onClose, children }) {
  return <div className="modal-backdrop" onClick={onClose}><div className="modal" onClick={e => e.stopPropagation()}>
    <div className="modal-head"><h2>{title}</h2><button className="icon-button" onClick={onClose}><X size={18}/></button></div>{children}
  </div></div>;
}
function LoginModal({ onClose, onLogin, configured }) {
  const [email, setEmail] = useState(""); const [password, setPassword] = useState("");
  return <ModalFrame title="Вход администратора" onClose={onClose}>
    {!configured ? <p className="muted">Сначала подключи Supabase по инструкции в README.</p> : <>
      <label className="field">Почта<input type="email" value={email} onChange={e => setEmail(e.target.value)} placeholder="Твоя почта"/></label>
      <label className="field">Пароль<input type="password" value={password} onChange={e => setPassword(e.target.value)} placeholder="Пароль"/></label>
      <button className="primary-button full" onClick={() => onLogin(email, password)}>Войти</button>
    </>}
  </ModalFrame>;
}
function SectionModal({ item, onClose, onSave }) {
  const [name, setName] = useState(item?.name || "");
  return <ModalFrame title={item ? "Переименовать раздел" : "Новый раздел"} onClose={onClose}>
    <label className="field">Название<input value={name} onChange={e => setName(e.target.value)} placeholder="Например, Новый мультфильм"/></label>
    <button className="primary-button full" onClick={() => onSave(name, item?.id)}>Сохранить</button>
  </ModalFrame>;
}
function CharacterModal({ item, sectionId, onClose, onSave }) {
  const [name, setName] = useState(item?.name || "");
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState(item?.image_url || "");
  return <ModalFrame title={item ? "Изменить персонажа" : "Добавить персонажа"} onClose={onClose}>
    <label className="field">Имя персонажа<input value={name} onChange={e => setName(e.target.value)} placeholder="Имя"/></label>
    <label className="upload-box"><ImagePlus size={22}/><b>{file ? file.name : "Выбрать изображение"}</b><span>Нажми, чтобы выбрать фото на телефоне</span><input type="file" accept="image/*" onChange={e => { const f=e.target.files?.[0]; if(f){setFile(f);setPreview(URL.createObjectURL(f));} }}/></label>
    {preview && <img className="upload-preview" src={preview} alt="Предпросмотр"/>}
    <button className="primary-button full" onClick={() => onSave({ name, id: item?.id, sectionId, file, oldImage: item?.image_url })}>Сохранить</button>
  </ModalFrame>;
}

createRoot(document.getElementById("root")).render(<App/>);
