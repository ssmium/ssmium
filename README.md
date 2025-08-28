<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Habit Tracker – GitHub Pages</title>
  <!-- Tailwind (CDN) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- React 18 UMD -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <!-- Recharts (UMD) -->
  <script src="https://unpkg.com/recharts/umd/Recharts.min.js"></script>
  <!-- Babel (JSX 컴파일) -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    @media (max-width:640px){ .field-grid{ grid-template-columns: 1fr !important; } }
    input,textarea{ -webkit-appearance:none; appearance:none; }
    .emoji-box{ line-height:1; }
  </style>
</head>
<body class="bg-[#F5F7FA] text-[#111]">
  <div id="root">로딩중...</div>

  <!-- 중요: data-presets="env,react" 추가 -->
  <script type="text/babel" data-presets="env,react">
    const { useState, useEffect, useMemo } = React;
    const { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer, CartesianGrid } = Recharts;

    // ===== Utils
    const STORAGE_KEY = "habit-pages-v1";
    const uid = () => Math.random().toString(36).slice(2, 9);
    const pad2 = (n) => String(n).padStart(2, "0");
    const parseISO = (iso) => { const [y,m,d]=iso.split("-").map(Number); return new Date(y,(m||1)-1,d||1); };
    const toISODate = (d) => `${d.getFullYear()}-${pad2(d.getMonth()+1)}-${pad2(d.getDate())}`;
    const todayISO = () => toISODate(new Date());
    const addDays = (iso, n) => { const d=parseISO(iso); d.setDate(d.getDate()+n); return toISODate(d); };
    const startOfWeek = (iso) => { const d=parseISO(iso); const day=d.getDay(); const diff=day===0?-6:1-day; d.setDate(d.getDate()+diff); return toISODate(d); };
    const rangeDays = (startISO, len=7) => Array.from({length:len},(_,i)=>addDays(startISO,i));
    const WEEK_LABELS = ["월","화","수","목","금","토","일"];
    const isoToWeekIdx = (iso) => { const d=parseISO(iso); return [1,2,3,4,5,6,0][d.getDay()]; };

    // ===== Base64 helpers for share link
    const toBase64 = (s)=>{ try{return btoa(unescape(encodeURIComponent(s)));}catch{return"";} };
    const fromBase64=(b)=>{ try{return decodeURIComponent(escape(atob(b)));}catch{return"";} };
    const HASH_PREFIX = "#data=";
    const migrate = (s)=>({ ...(s||{habits:[],logs:{}}), settings: { autoHash: (s?.settings?.autoHash!==false) }});
    const stateToHash = (s)=> HASH_PREFIX + toBase64(JSON.stringify(s));
    const syncHashIfEnabled = (s)=>{ try{
      if (s?.settings?.autoHash===false) return;
      const h = stateToHash(s);
      if (location.hash !== h) history.replaceState(null,"",location.pathname+location.search+h);
    }catch{} };

    // ===== Defaults
    function defaultState(){
      const base = [
        { id: uid(), name:"기상", time:"06:00", icon:"⏰", category:"아침", color:"#0ea5e9", days:[0,1,2,3,4,5,6], target:1 },
        { id: uid(), name:"스킨케어", time:"07:10", icon:"🧴", category:"케어", color:"#f97316", days:[0,1,2,3,4,5,6], target:1 },
        { id: uid(), name:"물 2잔", time:"09:00", icon:"💧", category:"건강", color:"#3b82f6", days:[0,1,2,3,4,5,6], target:2 },
      ];
      return { habits: base, logs: {}, settings:{autoHash:true} };
    }

    // ===== UI bits
    const Emoji = ({glyph,size=22}) =>
      <span className="emoji-box inline-flex items-center justify-center" style={{width:size,height:size}}>
        <span style={{fontSize:Math.round(size*0.9), lineHeight:1}}>{glyph||"✔"}</span>
      </span>;

    const Stepper = ({value,target,onClick}) =>
      <button onClick={onClick} className={`min-w-[68px] text-center px-3 py-2 rounded-xl border ${value>=target?"bg-[#003366] text-white":"bg-white"}`}>{value}/{target}</button>;

    function DateNav({date,setDate}) {
      const go=(n)=>setDate(addDays(date,n));
      return (
        <div className="flex items-center gap-2 text-sm">
          <button onClick={()=>go(-1)} className="px-2 py-1 rounded-lg border bg-white">◀</button>
          <div className="px-3 py-1 rounded-lg border bg-white">{date}</div>
          <button onClick={()=>go(1)} className="px-2 py-1 rounded-lg border bg-white">▶</button>
          <button onClick={()=>setDate(todayISO())} className="ml-2 px-2 py-1 rounded-lg border bg-white">오늘</button>
        </div>
      );
    }

    function DayMini({ habits, logs, getTarget }) {
      return (
        <div className="grid grid-cols-3 gap-1">
          {habits.slice(0,6).map(h=>{
            const c = logs[h.id]||0, t=getTarget(h.id); const pct=Math.min(1,t?c/t:0);
            return <div key={h.id} className="h-4 rounded-full" style={{background:`linear-gradient(90deg, ${h.color} ${pct*100}%, #E5E7EB ${pct*100}%)`}} title={`${h.name}: ${c}/${t||1}`} />;
          })}
        </div>
      );
    }

    function calcStreak(habit, logs){
      let s=0, d=todayISO();
      for(let i=0;i<365;i++){
        if(!habit.days.includes(isoToWeekIdx(d))){ d=addDays(d,-1); continue; }
        const c=(logs[d] && logs[d][habit.id])||0;
        if(c >= (habit.target||1)) s++; else break;
        d=addDays(d,-1);
      }
      return s;
    }

    // ===== App
    function App(){
      const [tab,setTab]=useState("today");
      const [date,setDate]=useState(todayISO());
      const [state,setState]=useState(()=>{
        try{
          if(location.hash.startsWith(HASH_PREFIX)){
            return migrate(JSON.parse(fromBase64(location.hash.slice(HASH_PREFIX.length))));
          }
        }catch{}
        try{ const raw=localStorage.getItem(STORAGE_KEY); if(raw) return migrate(JSON.parse(raw)); }catch{}
        return defaultState();
      });

      useEffect(()=>{ try{localStorage.setItem(STORAGE_KEY, JSON.stringify(state));}catch{} syncHashIfEnabled(state); },[state]);

      const weekStart = startOfWeek(date);
      const habitsForDay = (iso)=> state.habits.filter(h=>h.days.includes(isoToWeekIdx(iso)));
      const dayLogs = state.logs[date] || {};

      const toggle = (iso, hid) => setState(st=>{
        const logs={...st.logs}, day = logs[iso]?{...logs[iso]}:{};
        const cur = day[hid]||0; const target = st.habits.find(h=>h.id===hid)?.target||1;
        day[hid] = cur >= target ? 0 : cur+1; logs[iso]=day; return {...st, logs};
      });

      const addHabit = (p)=> setState(st=>({...st, habits:[{id:uid(),...p}, ...st.habits]}));
      const delHabit = (id)=> setState(st=>({...st, habits: st.habits.filter(h=>h.id!==id)}));

      return (
        <div className="min-h-screen">
          <header className="sticky top-0 z-20 backdrop-blur bg-white/70 border-b border-[#E6EDF5]">
            <div className="max-w-5xl mx-auto px-4 py-3 flex items-center justify-between">
              <div className="flex items-center gap-3">
                <div className="w-9 h-9 rounded-2xl bg-[#003366] grid place-content-center text-white font-bold shadow-sm">H</div>
                <div>
                  <h1 className="text-lg font-semibold">Habit Tracker – 시뮬레이터</h1>
                  <p className="text-xs text-[#555]">주간/일간 체크 · 스트릭 · 로컬 저장</p>
                </div>
              </div>
            </div>
            <nav className="max-w-5xl mx-auto px-2 pb-2">
              <div className="flex gap-2">
                {["today","habits","stats","settings"].map(id=>(
                  <button key={id} onClick={()=>setTab(id)} className={`px-3 py-2 rounded-2xl text-sm border ${tab===id?"bg-[#003366] text-white":"bg-white hover:bg-[#F5F7FA]"}`}>
                    {id==="today"?"오늘":id==="habits"?"습관":id==="stats"?"통계":"설정"}
                  </button>
                ))}
              </div>
            </nav>
          </header>

          <main className="max-w-5xl mx-auto p-4">
            {tab==="today" && (
              <div className="grid md:grid-cols-3 gap-4">
                <div className="md:col-span-2 bg-white rounded-2xl p-4 shadow-sm border">
                  <div className="flex items-center justify-between mb-3">
                    <h2 className="font-semibold">오늘의 체크</h2>
                    <DateNav date={date} setDate={setDate} />
                  </div>
                  <div className="grid gap-2">
                    {habitsForDay(date).map(h=>(
                      <div key={h.id} className="p-3 border rounded-xl flex items-center justify-between gap-3 bg-white">
                        <div className="flex items-center gap-3">
                          <Emoji glyph={h.icon} />
                          <div>
                            <div className="font-medium" style={{color:h.color}}>{h.name} <span className="text-xs text-[#666]">· {h.time||"--:--"}</span></div>
                            <div className="text-xs text-[#666]">카테고리: {h.category||"기타"} · 목표 {h.target||1}회</div>
                          </div>
                        </div>
                        <Stepper value={dayLogs[h.id]||0} target={h.target||1} onClick={()=>toggle(date,h.id)} />
                      </div>
                    ))}
                  </div>
                </div>
                <div className="bg-white rounded-2xl p-4 shadow-sm border">
                  <h3 className="font-semibold mb-2">이번 주 캘린더</h3>
                  <div className="grid grid-cols-7 gap-1 text-center text-xs mb-2">{WEEK_LABELS.map(w=><div key={w} className="text-[#666]">{w}</div>)}</div>
                </div>
              </div>
            )}

            {tab==="habits" && (
              <div className="bg-white rounded-2xl p-4 shadow-sm border">
                <h2 className="font-semibold mb-3">습관 추가/수정</h2>
                <p className="text-sm text-[#666]">간단 버전에서는 목록/통계 UI를 최소로 표시합니다.</p>
              </div>
            )}

            {tab==="settings" && (
              <div className="bg-white rounded-2xl p-4 shadow-sm border">
                <h2 className="font-semibold mb-2">설정</h2>
                <p className="text-sm">이 페이지는 GitHub Pages에서 동작 중입니다.</p>
              </div>
            )}
          </main>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById("root")).render(<App />);
  </script>
</body>
</html>
