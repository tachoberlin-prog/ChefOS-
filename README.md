
/**
 * ChefOS – Restaurant & Catering AI Operating System
 * Version 1.0 – Launch Ready
 *
 * DEPLOY IN 5 MINUTEN:
 * 1. npm install
 * 2. .env.example → .env (API Keys eintragen)
 * 3. npm run dev  (lokal testen)
 * 4. vercel deploy --prod
 */

import { useState, useRef, useEffect, useCallback } from "react";

// ─────────────────────────────────────────────────────────────────────────────
// DESIGN TOKENS
// ─────────────────────────────────────────────────────────────────────────────
const T = {
  bg:       "#0C0C0A",
  surface:  "#141412",
  card:     "#1C1C19",
  cardHov:  "#222220",
  border:   "#2A2A26",
  borderHi: "#3A3A35",
  gold:     "#C9973A",
  goldDim:  "#8A6420",
  goldGlow: "rgba(201,151,58,0.11)",
  gold2:    "#E8B84B",
  green:    "#3D9E6A",
  greenDim: "rgba(61,158,106,0.13)",
  red:      "#C94A3A",
  redDim:   "rgba(201,74,58,0.13)",
  blue:     "#3A6EC9",
  blueDim:  "rgba(58,110,201,0.13)",
  purple:   "#8B5CF6",
  text:     "#EDEAE2",
  muted:    "#7A7A72",
  faint:    "#323230",
};

// ─────────────────────────────────────────────────────────────────────────────
// INITIAL DATA
// ─────────────────────────────────────────────────────────────────────────────
const TEAM = [
  { id:"chef",    name:"Chef",   role:"Küchenchef",  avatar:"C", color:T.gold   },
  { id:"marco",   name:"Marco",  role:"Sous Chef",   avatar:"M", color:T.blue   },
  { id:"lisa",    name:"Lisa",   role:"Einkauf",     avatar:"L", color:T.green  },
  { id:"thomas",  name:"Thomas", role:"Service",     avatar:"T", color:T.purple },
];

const INIT_CATERINGS = [
  { id:1, name:"Hochzeit Meyer",      date:"2025-06-12", pax:120, budget:6800,  location:"Gut Brunnenhof",  status:"confirmed", menu:"5-Gang Menü",    allergien:"Nüsse, Gluten", personal:8,  notes:"Blumendeko vom Kunden gestellt" },
  { id:2, name:"Firmenevent BMW",     date:"2025-06-15", pax:80,  budget:4200,  location:"BMW Welt München",status:"offer",     menu:"Flying Buffet",   allergien:"Laktose",       personal:6,  notes:"Vegane Option für 20 Pers." },
  { id:3, name:"Gala Stadtmuseum",    date:"2025-06-18", pax:200, budget:12000, location:"Stadtmuseum",     status:"planning",  menu:"Gala Dinner",     allergien:"div.",          personal:14, notes:"Black Tie, Rotwein pre-selected" },
  { id:4, name:"Geburtstag Dr. Kraus",date:"2025-06-22", pax:40,  budget:2800,  location:"Privatvilla",     status:"confirmed", menu:"BBQ & Dessertbuffet", allergien:"keine",   personal:4,  notes:"Outdoor, Zelt vorhanden" },
];

const INIT_TASKS = [
  { id:1, title:"Angebot BMW Firmenevent versenden",      prio:"high", person:"chef",   deadline:"2025-06-09", status:"open",    module:"catering"  },
  { id:2, title:"Einkauf Hochzeit Meyer planen",          prio:"high", person:"lisa",   deadline:"2025-06-10", status:"working", module:"einkauf"   },
  { id:3, title:"HACCP Dokumentation aktualisieren",      prio:"mid",  person:"marco",  deadline:"2025-06-15", status:"open",    module:"dokumente" },
  { id:4, title:"Schichtplan Juni finalisieren",          prio:"mid",  person:"chef",   deadline:"2025-06-12", status:"done",    module:"personal"  },
  { id:5, title:"Lieferanten Preisvergleich Q3",          prio:"low",  person:"lisa",   deadline:"2025-06-20", status:"open",    module:"einkauf"   },
  { id:6, title:"Gala Stadtmuseum: Menü finalisieren",    prio:"high", person:"chef",   deadline:"2025-06-11", status:"working", module:"catering"  },
  { id:7, title:"Kühlkette Protokoll Mai abschließen",    prio:"mid",  person:"marco",  deadline:"2025-06-10", status:"open",    module:"haccp"     },
];

const INIT_EMAILS = [
  { id:1, from:"anna.mueller@gmail.com",    name:"Anna Müller",    subject:"Anfrage Hochzeit August 2025",         body:"Sehr geehrte Damen und Herren,\n\nwir planen unsere Hochzeitsfeier am 16. August 2025 und suchen einen erfahrenen Caterer.\n\nWir erwarten ca. 90 Gäste und haben ein Budget von ungefähr €5.000–6.000. Die Feier findet im Schloss Nymphenburg statt.\n\nBitte teilen Sie uns mit, ob Sie an diesem Datum verfügbar sind und welche Pakete Sie anbieten.\n\nMit freundlichen Grüßen,\nAnna Müller",    date:"Heute 09:14", read:false, cat:"inquiry",  pax:90,  urgent:true  },
  { id:2, from:"events@siemens-munich.com", name:"Siemens Events", subject:"Catering Jahresabschlussfeier Dez.",   body:"Guten Tag,\n\nwir suchen für unsere Jahresabschlussfeier am 12. Dezember 2025 einen Caterer.\n\nDetails:\n- Ca. 150 Mitarbeiter\n- Location: Siemens Forum München\n- Budget: offen\n- Wünsche: Fingerfood & Flying Buffet\n\nBitte senden Sie uns ein unverbindliches Angebot.\n\nMit freundlichen Grüßen,\nEvents-Team Siemens",                   date:"Gestern 16:42", read:false, cat:"inquiry",  pax:150, urgent:false },
  { id:3, from:"preise@metro-cc.de",        name:"METRO C&C",      subject:"Neue Preisliste Q3 2025",              body:"Sehr geehrter Kunde,\n\nanbei finden Sie unsere aktualisierte Preisliste für Q3 2025.\n\nHöhepunkte:\n- Rindfleisch: +3%\n- Geflügel: stabil\n- Fisch & Meeresfrüchte: -2%\n- Molkereiprodukte: +5%\n\nIhre Einkaufskonditionen bleiben unverändert.\n\nBeste Grüße,\nIhr METRO-Team",                                                               date:"08. Jun",       read:true,  cat:"supplier", pax:0,   urgent:false },
  { id:4, from:"buchung@gutbrunnenhof.de",  name:"Gut Brunnenhof", subject:"Bestätigung Reservierung Meyer 12.6.", body:"Sehr geehrte Damen und Herren,\n\nwir bestätigen hiermit die Reservierung der Scheune für den 12. Juni 2025.\n\nDetails:\n- Einlass: 15:00 Uhr\n- Kapazität: bis 150 Personen\n- Küche verfügbar ab 14:00 Uhr\n- Parkplätze: 80\n\nBitte melden Sie sich bei Änderungen.\n\nMit freundlichen Grüßen,\nGut Brunnenhof",                          date:"07. Jun",       read:true,  cat:"venue",    pax:0,   urgent:false },
];

const INIT_RECIPES = [
  { id:1, name:"Beef Tenderloin",  cat:"Warme Küche",  base:10, time:45, station:"Warme Küche",  allergens:["Milch","Senf"],
    steps:["Filet parieren und würzen","Scharf anbraten 2 Min. je Seite","Im Ofen bei 180°C 12 Min. garen","5 Min. ruhen lassen","Aufschneiden und anrichten"],
    ingredients:[
      { id:"i1", name:"Rinderfilet",  amount:2.5, unit:"kg", price:42.00, type:"linear" },
      { id:"i2", name:"Butter",       amount:200, unit:"g",  price:0.80,  type:"linear" },
      { id:"i3", name:"Rosmarin",     amount:4,   unit:"Zw", price:0.20,  type:"capped" },
      { id:"i4", name:"Knoblauch",    amount:3,   unit:"Zeh",price:0.10,  type:"capped" },
      { id:"i5", name:"Meersalz",     amount:15,  unit:"g",  price:0.05,  type:"linear" },
      { id:"i6", name:"Schwarzpfeff.",amount:5,   unit:"g",  price:0.08,  type:"linear" },
    ]},
  { id:2, name:"Vitello Tonnato",  cat:"Kalte Küche",  base:10, time:30, station:"Kalte Küche",  allergens:["Ei","Fisch","Senf"],
    steps:["Kalbsfleisch pochieren 45 Min.","Abkühlen und aufschneiden","Thunfischmayo zubereiten","Anrichten mit Kapern","Gekühlt servieren"],
    ingredients:[
      { id:"i7",  name:"Kalbsfleisch", amount:1.8, unit:"kg", price:28.00, type:"linear" },
      { id:"i8",  name:"Thunfisch",    amount:200, unit:"g",  price:3.20,  type:"linear" },
      { id:"i9",  name:"Mayonnaise",   amount:300, unit:"g",  price:1.20,  type:"linear" },
      { id:"i10", name:"Kapern",       amount:50,  unit:"g",  price:0.80,  type:"capped" },
      { id:"i11", name:"Zitrone",      amount:2,   unit:"St", price:0.50,  type:"capped" },
    ]},
  { id:3, name:"Tiramisu",         cat:"Patisserie",   base:12, time:60, station:"Patisserie",   allergens:["Ei","Milch","Gluten"],
    steps:["Mascarpone-Creme aufschlagen","Löffelbiskuits tränken","Schichten aufbauen","Kühl stellen 4h","Vor Service mit Kakao bestäuben"],
    ingredients:[
      { id:"i12", name:"Mascarpone",   amount:500, unit:"g",  price:4.50,  type:"linear" },
      { id:"i13", name:"Eier",         amount:4,   unit:"St", price:1.20,  type:"linear" },
      { id:"i14", name:"Zucker",       amount:100, unit:"g",  price:0.20,  type:"linear" },
      { id:"i15", name:"Löffelbiskuits",amount:200,unit:"g",  price:1.80,  type:"linear" },
      { id:"i16", name:"Espresso",     amount:300, unit:"ml", price:0.60,  type:"linear" },
      { id:"i17", name:"Kakao",        amount:30,  unit:"g",  price:0.40,  type:"capped" },
    ]},
];

const STATUS_META = {
  confirmed: { label:"Bestätigt",  color:T.green  },
  offer:     { label:"Angebot",    color:T.gold   },
  planning:  { label:"In Planung", color:T.blue   },
  cancelled: { label:"Storniert",  color:T.red    },
};
const PRIO_COLOR = { high:T.red, mid:T.gold, low:T.muted };
const PRIO_LABEL = { high:"Hoch",  mid:"Mittel", low:"Niedrig" };
const TASK_STATUS = {
  open:    { icon:"□", next:"working", color:T.muted  },
  working: { icon:"◐", next:"done",    color:T.gold   },
  done:    { icon:"☑", next:"open",    color:T.green  },
};

// ─────────────────────────────────────────────────────────────────────────────
// HELPERS
// ─────────────────────────────────────────────────────────────────────────────
const eur = (n) => new Intl.NumberFormat("de-DE",{style:"currency",currency:"EUR"}).format(n);
const fmtDate = (s) => new Date(s).toLocaleDateString("de-DE",{day:"2-digit",month:"short"});
const fmtDateShort = (s) => new Date(s).toLocaleDateString("de-DE",{day:"2-digit",month:"2-digit"});

async function callClaude(system, userMsg, history=[]) {
  try {
    const apiKey = import.meta.env?.VITE_ANTHROPIC_KEY || "";
    const res = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: { "Content-Type":"application/json", ...(apiKey ? {"x-api-key": apiKey} : {}) },
      body: JSON.stringify({
        model: "claude-sonnet-4-20250514",
        max_tokens: 1500,
        system,
        messages: [...history, { role:"user", content:userMsg }]
      })
    });
    const data = await res.json();
    if (data.error) return `⚠ API Fehler: ${data.error.message}. Bitte VITE_ANTHROPIC_KEY in .env setzen.`;
    return data.content?.map(b=>b.text||"").join("") || "Keine Antwort.";
  } catch(e) {
    return "⚠ Verbindungsfehler. API Key prüfen.";
  }
}

const SYSTEM_PROMPT = `Du bist ChefOS – KI-Betriebssystem für ein professionelles Restaurant- und Catering-Unternehmen.
Du agierst gleichzeitig als: Küchenchef, Catering Manager, Operations Manager, Executive Assistant, Controller.
IMMER auf Deutsch antworten. Präzise, professionell, direkt umsetzbar.
Nutze Markdown-Strukturen (Tabellen, Listen, Überschriften) für klare Ausgaben.
Entscheidungsreihenfolge: 1. Lebensmittelsicherheit 2. Qualität 3. Wirtschaftlichkeit 4. Aufwand 5. Nachhaltigkeit.
Bei Voice Notes: transkribieren → strukturieren → Aufgaben ableiten → Kalender/Einkauf aktualisieren.
Bei E-Mails: analysieren → Eckdaten extrahieren → professionelle Antwort formulieren.`;

// ─────────────────────────────────────────────────────────────────────────────
// UI PRIMITIVES
// ─────────────────────────────────────────────────────────────────────────────
function Badge({ color, children, sm }) {
  return (
    <span style={{
      background: color+"1E", color, border:`1px solid ${color}40`,
      padding: sm ? "1px 7px" : "2px 11px",
      borderRadius:20, fontSize: sm?10:11, fontWeight:600, letterSpacing:0.4, whiteSpace:"nowrap"
    }}>{children}</span>
  );
}

function Card({ children, style={}, onClick, active }) {
  const [hov, setHov] = useState(false);
  return (
    <div
      onClick={onClick}
      onMouseEnter={()=>onClick&&setHov(true)}
      onMouseLeave={()=>onClick&&setHov(false)}
      style={{
        background: active ? T.goldGlow : hov ? T.cardHov : T.card,
        border: `1px solid ${active ? T.gold+"55" : hov ? T.borderHi : T.border}`,
        borderRadius:14, padding:20, transition:"all 0.15s",
        cursor: onClick ? "pointer" : "default", ...style
      }}
    >{children}</div>
  );
}

function Btn({ children, variant="primary", onClick, disabled, style={}, sm, full }) {
  const [hov, setHov] = useState(false);
  const sizes = sm ? { fontSize:12, padding:"6px 14px" } : { fontSize:13, padding:"9px 20px" };
  const variants = {
    primary: { background: hov ? "#D4A840" : T.gold, color:"#000" },
    ghost:   { background:"transparent", color:T.muted, border:`1px solid ${T.border}` },
    outline: { background: hov ? T.goldGlow : "transparent", color:T.gold, border:`1px solid ${T.gold}44` },
    danger:  { background: hov ? T.redDim+"99" : T.redDim, color:T.red, border:`1px solid ${T.red}44` },
    success: { background: hov ? T.greenDim+"99" : T.greenDim, color:T.green, border:`1px solid ${T.green}44` },
  };
  return (
    <button
      onClick={onClick} disabled={disabled}
      onMouseEnter={()=>setHov(true)} onMouseLeave={()=>setHov(false)}
      style={{
        border:"none", borderRadius:9, fontWeight:600, fontFamily:"inherit",
        cursor: disabled?"not-allowed":"pointer", opacity:disabled?0.5:1,
        transition:"all 0.12s", width: full?"100%":"auto",
        ...sizes, ...variants[variant], ...style
      }}
    >{children}</button>
  );
}

function Input({ value, onChange, placeholder, rows, onKeyDown }) {
  const base = {
    width:"100%", background:T.surface, border:`1px solid ${T.border}`,
    borderRadius:9, padding:"9px 13px", color:T.text, fontSize:13,
    outline:"none", boxSizing:"border-box", fontFamily:"inherit",
    resize: rows ? "vertical" : "none"
  };
  if (rows) return <textarea value={value} onChange={onChange} placeholder={placeholder} rows={rows} style={base}/>;
  return <input value={value} onChange={onChange} placeholder={placeholder} onKeyDown={onKeyDown} style={base}/>;
}

function SectionTitle({ title, sub, action }) {
  return (
    <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:22 }}>
      <div>
        <h2 style={{ color:T.text, fontFamily:"'Playfair Display',serif", fontWeight:600, fontSize:22, margin:0 }}>{title}</h2>
        {sub && <div style={{ color:T.muted, fontSize:12, marginTop:3 }}>{sub}</div>}
      </div>
      {action}
    </div>
  );
}

function KPI({ label, value, sub, color=T.gold, icon }) {
  return (
    <Card style={{ flex:1, minWidth:130 }}>
      <div style={{ display:"flex", justifyContent:"space-between" }}>
        <div style={{ color:T.muted, fontSize:10, letterSpacing:1.2, textTransform:"uppercase", marginBottom:10 }}>{label}</div>
        {icon && <span style={{ opacity:0.4, fontSize:16 }}>{icon}</span>}
      </div>
      <div style={{ color, fontSize:26, fontWeight:700, fontFamily:"'Playfair Display',serif", lineHeight:1 }}>{value}</div>
      {sub && <div style={{ color:T.muted, fontSize:11, marginTop:6, lineHeight:1.4 }}>{sub}</div>}
    </Card>
  );
}

function Avatar({ member, size=32 }) {
  return (
    <div style={{
      width:size, height:size, borderRadius:"50%",
      background: member.color+"22", border:`1.5px solid ${member.color}`,
      display:"flex", alignItems:"center", justifyContent:"center",
      color: member.color, fontWeight:700, fontSize:size*0.38, flexShrink:0
    }}>{member.avatar}</div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// VOICE NOTE MODAL
// ─────────────────────────────────────────────────────────────────────────────
function VoiceModal({ sender, onClose, onTaskCreated }) {
  const [phase, setPhase] = useState("idle");
  const [transcript, setTranscript]   = useState("");
  const [result, setResult]           = useState(null);
  const [loading, setLoading]         = useState(false);
  const [seconds, setSeconds]         = useState(0);
  const mediaRef   = useRef(null);
  const recognRef  = useRef(null);
  const timerRef   = useRef(null);

  const startRecording = useCallback(async () => {
    setTranscript(""); setResult(null); setSeconds(0);
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio:true });
      mediaRef.current = new MediaRecorder(stream);
      mediaRef.current.ondataavailable = ()=>{};
      mediaRef.current.onstop = () => stream.getTracks().forEach(t=>t.stop());
      mediaRef.current.start();

      const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
      if (SR) {
        const r = new SR();
        r.lang = "de-DE";
        r.continuous = true;
        r.interimResults = true;
        r.onresult = (e) => {
          let t = "";
          for (let i=0; i<e.results.length; i++) t += e.results[i][0].transcript;
          setTranscript(t);
        };
        r.start();
        recognRef.current = r;
      }
      timerRef.current = setInterval(() => setSeconds(s=>s+1), 1000);
      setPhase("recording");
    } catch {
      setPhase("error");
    }
  }, []);

  const stopRecording = useCallback(() => {
    mediaRef.current?.stop();
    recognRef.current?.stop();
    clearInterval(timerRef.current);
    setPhase("stopped");
  }, []);

  const process = async (text) => {
    if (!text?.trim()) return;
    setLoading(true); setPhase("processing");
    const r = await callClaude(
      SYSTEM_PROMPT + `\nSender: ${sender.name} (${sender.role})`,
      `VOICE NOTE TRANSKRIPT von ${sender.name}:\n"${text}"\n\nBitte:\n1. Inhalt strukturiert zusammenfassen\n2. Konkrete Aufgaben ableiten (Titel, Verantwortlicher, Deadline, Priorität)\n3. Prüfen ob Einkauf / Kalender / Catering-Infos enthalten\n4. Handlungsempfehlung geben`
    );
    setResult(r);
    setLoading(false);
    setPhase("done");
  };

  const fmt = (s) => `${String(Math.floor(s/60)).padStart(2,"0")}:${String(s%60).padStart(2,"0")}`;

  return (
    <div style={{ position:"fixed", inset:0, background:"rgba(0,0,0,0.82)", display:"flex", alignItems:"center", justifyContent:"center", zIndex:1000, backdropFilter:"blur(6px)" }}>
      <Card style={{ width:580, maxHeight:"88vh", overflow:"auto", border:`1px solid ${T.gold}33` }}>
        {/* Header */}
        <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:22 }}>
          <div style={{ display:"flex", alignItems:"center", gap:12 }}>
            <Avatar member={sender} size={38}/>
            <div>
              <div style={{ color:T.gold, fontWeight:700, fontSize:15 }}>✦ Voice Note</div>
              <div style={{ color:T.muted, fontSize:12 }}>{sender.name} · {sender.role}</div>
            </div>
          </div>
          <Btn variant="ghost" onClick={onClose} sm>✕</Btn>
        </div>

        {/* Recorder */}
        <div style={{ textAlign:"center", padding:"24px 0 20px" }}>
          <div
            onClick={phase==="idle"||phase==="stopped" ? startRecording : phase==="recording" ? stopRecording : undefined}
            style={{
              width:88, height:88, borderRadius:"50%", margin:"0 auto 14px",
              background: phase==="recording" ? T.redDim : T.goldGlow,
              border: `2px solid ${phase==="recording" ? T.red : T.gold}`,
              display:"flex", alignItems:"center", justifyContent:"center",
              fontSize:36, cursor: loading ? "default" : "pointer",
              boxShadow: phase==="recording" ? `0 0 32px ${T.red}55` : "none",
              transition:"all 0.2s",
              animation: phase==="recording" ? "pulse 1.5s ease-in-out infinite" : "none"
            }}>
            {loading ? "⚙" : phase==="recording" ? "⏹" : "🎙"}
          </div>

          {phase==="recording" && (
            <div style={{ color:T.red, fontSize:13, fontWeight:600, letterSpacing:1 }}>
              ● REC {fmt(seconds)}
            </div>
          )}
          {phase==="idle" && <div style={{ color:T.muted, fontSize:13 }}>Tippen zum Starten</div>}
          {phase==="stopped" && <div style={{ color:T.gold, fontSize:13 }}>Aufnahme gestoppt · Verarbeiten?</div>}
          {phase==="processing" && <div style={{ color:T.gold, fontSize:13 }}>✦ KI analysiert...</div>}
          {phase==="error" && <div style={{ color:T.red, fontSize:13 }}>Mikrofon-Zugriff verweigert</div>}
        </div>

        {/* Transcript box */}
        {(transcript || phase==="stopped" || phase==="idle") && (
          <div style={{ marginBottom:16 }}>
            <div style={{ color:T.muted, fontSize:11, letterSpacing:1, textTransform:"uppercase", marginBottom:8 }}>Transkript / Text eingeben</div>
            <Input
              value={transcript}
              onChange={e=>setTranscript(e.target.value)}
              rows={4}
              placeholder="Sprich jetzt... oder tippe direkt hier deine Idee, Bestellung oder Änderung."
            />
            {phase!=="recording" && transcript.trim() && (
              <div style={{ marginTop:10 }}>
                <Btn onClick={()=>process(transcript)} disabled={loading}>
                  {loading ? "Analysiere..." : "✦ KI verarbeiten"}
                </Btn>
              </div>
            )}
          </div>
        )}

        {/* Result */}
        {result && (
          <div>
            <div style={{ color:T.muted, fontSize:11, letterSpacing:1, textTransform:"uppercase", marginBottom:8 }}>✦ ChefOS Analyse</div>
            <div style={{
              background:T.surface, border:`1px solid ${T.gold}33`, borderRadius:10,
              padding:16, color:T.text, fontSize:13, lineHeight:1.75,
              whiteSpace:"pre-wrap", maxHeight:320, overflowY:"auto"
            }}>{result}</div>
            <div style={{ display:"flex", gap:8, marginTop:14 }}>
              <Btn onClick={() => { onTaskCreated?.(transcript, result); onClose(); }}>
                ✓ Aufgaben übernehmen & schließen
              </Btn>
              <Btn variant="ghost" sm onClick={onClose}>Nur schließen</Btn>
            </div>
          </div>
        )}
      </Card>
      <style>{`
        @keyframes pulse { 0%,100%{transform:scale(1);opacity:1} 50%{transform:scale(1.06);opacity:0.85} }
      `}</style>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// DASHBOARD
// ─────────────────────────────────────────────────────────────────────────────
function Dashboard({ caterings, tasks, emails, setActive }) {
  const unread   = emails.filter(e=>!e.read);
  const openTasks= tasks.filter(t=>t.status!=="done").slice(0,6);
  const upcoming = caterings.slice(0,4);

  const alerts = [
    unread.length>0 && { icon:"📧", text:`${unread.length} ungelesene Mails · ${unread.filter(e=>e.cat==="inquiry").length} neue Anfragen`, mod:"emails" },
    { icon:"⚠️", text:"Olivenöl: Bestand 2L · Mindestmenge 5L", mod:"lager" },
    { icon:"📅", text:"Hochzeit Meyer: Einkaufsfrist in 2 Tagen", mod:"einkauf" },
    tasks.filter(t=>t.status!=="done"&&t.prio==="high").length>0 && {
      icon:"🔴", text:`${tasks.filter(t=>t.status!=="done"&&t.prio==="high").length} dringende Aufgaben offen`, mod:"tasks"
    },
  ].filter(Boolean);

  return (
    <div>
      <div style={{ marginBottom:28 }}>
        <div style={{ color:T.muted, fontSize:11, letterSpacing:2, textTransform:"uppercase" }}>
          {new Date().toLocaleDateString("de-DE",{weekday:"long",day:"numeric",month:"long",year:"numeric"})}
        </div>
        <h1 style={{ color:T.text, fontFamily:"'Playfair Display',serif", fontWeight:400, fontSize:30, margin:"6px 0 0", lineHeight:1.2 }}>
          Guten Morgen, Chef.{" "}
          <span style={{ color:T.gold }}>{openTasks.length} Aufgaben</span> offen.
        </h1>
      </div>

      <div style={{ display:"flex", gap:12, marginBottom:20, flexWrap:"wrap" }}>
        <KPI label="Umsatz Juni" value="€18.400" sub="+12% vs. Vormonat" icon="💰"/>
        <KPI label="Caterings" value={caterings.length} sub="nächste 30 Tage" color={T.blue} icon="🍽"/>
        <KPI label="Wareneinsatz" value="28%" sub="Ziel ≤30% ✓" color={T.green} icon="📊"/>
        <KPI label="Offene Angebote" value="€9.200" sub="2 ausstehend" icon="📄"/>
        <KPI label="Neue E-Mails" value={unread.length} sub={`${unread.filter(e=>e.cat==="inquiry").length} Anfragen`} color={unread.length>0?T.red:T.green} icon="📧"/>
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:16, marginBottom:16 }}>
        <Card>
          <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:16 }}>
            <span style={{ color:T.text, fontWeight:600 }}>Nächste Caterings</span>
            <span onClick={()=>setActive("catering")} style={{ color:T.gold, fontSize:12, cursor:"pointer" }}>Alle →</span>
          </div>
          {upcoming.map((e,i)=>(
            <div key={e.id} style={{ display:"flex", alignItems:"center", gap:12, padding:"10px 0", borderBottom:i<upcoming.length-1?`1px solid ${T.border}`:"none" }}>
              <div style={{ background:T.goldGlow, color:T.gold, borderRadius:8, padding:"4px 10px", fontSize:11, fontWeight:700, minWidth:54, textAlign:"center" }}>
                {fmtDate(e.date)}
              </div>
              <div style={{ flex:1, minWidth:0 }}>
                <div style={{ color:T.text, fontSize:13, fontWeight:500, overflow:"hidden", textOverflow:"ellipsis", whiteSpace:"nowrap" }}>{e.name}</div>
                <div style={{ color:T.muted, fontSize:11 }}>{e.pax} Pers. · {e.location}</div>
              </div>
              <Badge color={STATUS_META[e.status].color}>{STATUS_META[e.status].label}</Badge>
            </div>
          ))}
        </Card>

        <Card>
          <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:16 }}>
            <span style={{ color:T.text, fontWeight:600 }}>Offene Aufgaben</span>
            <span onClick={()=>setActive("tasks")} style={{ color:T.gold, fontSize:12, cursor:"pointer" }}>Alle →</span>
          </div>
          {openTasks.map((t,i)=>{
            const member = TEAM.find(m=>m.id===t.person);
            return (
              <div key={t.id} style={{ display:"flex", alignItems:"center", gap:10, padding:"9px 0", borderBottom:i<openTasks.length-1?`1px solid ${T.border}`:"none" }}>
                <span style={{ fontSize:16, color:TASK_STATUS[t.status].color }}>{TASK_STATUS[t.status].icon}</span>
                <span style={{ flex:1, color:T.text, fontSize:13, overflow:"hidden", textOverflow:"ellipsis", whiteSpace:"nowrap" }}>{t.title}</span>
                {member && <Avatar member={member} size={22}/>}
                <Badge color={PRIO_COLOR[t.prio]} sm>{t.deadline?fmtDateShort(t.deadline):""}</Badge>
              </div>
            );
          })}
        </Card>
      </div>

      <Card>
        <div style={{ color:T.text, fontWeight:600, marginBottom:14 }}>Warnungen & Aktionen</div>
        <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:10 }}>
          {alerts.map((a,i)=>(
            <div key={i} onClick={()=>a.mod&&setActive(a.mod)}
              style={{ display:"flex", gap:10, padding:12, background:T.surface, borderRadius:10, border:`1px solid ${T.border}`, cursor:"pointer", alignItems:"center", transition:"border-color 0.15s" }}>
              <span style={{ fontSize:20 }}>{a.icon}</span>
              <span style={{ color:T.muted, fontSize:12, flex:1, lineHeight:1.4 }}>{a.text}</span>
              <span style={{ color:T.faint, fontSize:14 }}>→</span>
            </div>
          ))}
        </div>
      </Card>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// CATERING
// ─────────────────────────────────────────────────────────────────────────────
function CateringModule({ caterings, setCaterings }) {
  const [selected, setSelected] = useState(null);
  const [showNew, setShowNew]   = useState(false);
  const [genType, setGenType]   = useState(null);
  const [genResult, setGenResult] = useState(null);
  const blank = { name:"",date:"",pax:"",budget:"",location:"",menu:"",allergien:"",personal:"",notes:"" };
  const [form, setForm] = useState(blank);
  const f = (k) => (e)=>setForm(p=>({...p,[k]:e.target.value}));

  const job = caterings.find(j=>j.id===selected);

  const save = () => {
    if (!form.name||!form.date) return;
    setCaterings(c=>[...c, { ...form, id:Date.now(), status:"planning", pax:+form.pax||0, budget:+form.budget||0, personal:+form.personal||0 }]);
    setForm(blank); setShowNew(false);
  };

  const generate = async (type) => {
    if (!job) return;
    setGenType(type); setGenResult(null);
    const map = {
      einkauf: `Erstelle eine vollständige Einkaufsliste für:\n${JSON.stringify(job,null,2)}\n\nFormat: Tabelle mit Produkt | Menge | Einheit | Lieferant-Empfehlung | Priorität. Gruppenweise (Fleisch, Gemüse, Trockenwaren...).`,
      prep:    `Erstelle einen detaillierten Prep-Plan nach Stationen für:\n${JSON.stringify(job,null,2)}\n\nFormat: Station | Aufgabe | Menge | Verantwortlicher | Uhrzeit. Chronologisch.`,
      ablauf:  `Erstelle einen Ablaufplan für das Catering:\n${JSON.stringify(job,null,2)}\n\nVon Aufbau bis Abbau. Format: Uhrzeit | Aktivität | Team | Ort.`,
      angebot: `Formuliere ein professionelles Catering-Angebot (auf Deutsch) für:\n${JSON.stringify(job,null,2)}\n\nEnthält: Anrede, Leistungsbeschreibung, Preisstruktur, Zahlungsbedingungen, Gültigkeitsdatum, Grußformel.`,
      rechnung:`Erstelle Rechnungspositionen für:\n${JSON.stringify(job,null,2)}\n\nFormat: Position | Beschreibung | Menge | Einzelpreis | Gesamtpreis. Inkl. MwSt-Ausweisung.`,
      haccp:   `Erstelle eine HACCP-Checkliste für dieses Catering:\n${JSON.stringify(job,null,2)}\n\nKritische Kontrollpunkte, Temperaturüberwachung, Allergenkontrolle.`,
    };
    const r = await callClaude(SYSTEM_PROMPT, map[type]);
    setGenResult({ type, content:r });
    setGenType(null);
  };

  return (
    <div>
      <SectionTitle title="Catering Aufträge" sub={`${caterings.length} Aufträge aktiv`}
        action={<Btn onClick={()=>setShowNew(true)}>+ Neuer Auftrag</Btn>}/>

      {showNew && (
        <Card style={{ marginBottom:16, border:`1px solid ${T.gold}44` }}>
          <div style={{ color:T.gold, fontWeight:700, marginBottom:14 }}>✦ Neuer Auftrag</div>
          <div style={{ display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:10 }}>
            {[["name","Kundenname"],["date","Datum","date"],["pax","Personen","number"],["budget","Budget €","number"],["location","Location"],["menu","Menüart"],["allergien","Allergien / Diäten"],["personal","Personal (Anzahl)","number"]].map(([k,l,t="text"])=>(
              <div key={k}>
                <div style={{ color:T.muted, fontSize:11, marginBottom:4 }}>{l}</div>
                <Input value={form[k]} onChange={f(k)} placeholder={l}/>
              </div>
            ))}
          </div>
          <div style={{ marginTop:10 }}>
            <div style={{ color:T.muted, fontSize:11, marginBottom:4 }}>Notizen</div>
            <Input value={form.notes} onChange={f("notes")} rows={2} placeholder="Besondere Anforderungen, Wünsche, Hinweise..."/>
          </div>
          <div style={{ display:"flex", gap:8, marginTop:14 }}>
            <Btn onClick={save}>Speichern</Btn>
            <Btn variant="ghost" onClick={()=>{setShowNew(false);setForm(blank);}}>Abbrechen</Btn>
          </div>
        </Card>
      )}

      <div style={{ display:"grid", gridTemplateColumns:job?"1fr 420px":"1fr", gap:16 }}>
        <div style={{ display:"flex", flexDirection:"column", gap:10 }}>
          {caterings.map(j=>(
            <Card key={j.id} active={j.id===selected} onClick={()=>{ setSelected(j.id===selected?null:j.id); setGenResult(null); }}>
              <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start" }}>
                <div style={{ flex:1, minWidth:0 }}>
                  <div style={{ color:T.text, fontWeight:600, fontSize:15 }}>{j.name}</div>
                  <div style={{ color:T.muted, fontSize:12, marginTop:2 }}>{fmtDate(j.date)} · {j.location}</div>
                </div>
                <Badge color={STATUS_META[j.status].color}>{STATUS_META[j.status].label}</Badge>
              </div>
              <div style={{ display:"flex", gap:16, marginTop:10, flexWrap:"wrap" }}>
                <span style={{ color:T.muted, fontSize:12 }}>👥 {j.pax} Pers.</span>
                <span style={{ color:T.muted, fontSize:12 }}>💰 {eur(j.budget)}</span>
                <span style={{ color:T.muted, fontSize:12 }}>🍽 {j.menu}</span>
                {j.allergien && j.allergien!=="keine" && <span style={{ color:T.red, fontSize:12 }}>⚠ {j.allergien}</span>}
              </div>
            </Card>
          ))}
        </div>

        {job && (
          <div style={{ display:"flex", flexDirection:"column", gap:12 }}>
            <Card style={{ border:`1px solid ${T.gold}33` }}>
              <div style={{ color:T.gold, fontWeight:700, fontSize:16, marginBottom:16 }}>{job.name}</div>
              {[["Datum",fmtDate(job.date)],["Location",job.location],["Personen",job.pax],
                ["Budget",eur(job.budget)],["Menü",job.menu],["Allergien",job.allergien||"–"],
                ["Personal",`${job.personal} Personen`],["Notizen",job.notes||"–"]].map(([k,v])=>(
                <div key={k} style={{ display:"flex", justifyContent:"space-between", padding:"8px 0", borderBottom:`1px solid ${T.border}` }}>
                  <span style={{ color:T.muted, fontSize:12 }}>{k}</span>
                  <span style={{ color:T.text, fontSize:13, textAlign:"right", maxWidth:240 }}>{v}</span>
                </div>
              ))}
            </Card>

            <Card>
              <div style={{ color:T.muted, fontSize:11, textTransform:"uppercase", letterSpacing:1, marginBottom:12 }}>✦ Dokumente generieren</div>
              <div style={{ display:"grid", gridTemplateColumns:"1fr 1fr", gap:8 }}>
                {[["einkauf","📋 Einkaufsliste"],["prep","🔪 Prep-Plan"],["ablauf","🗓 Ablaufplan"],["angebot","📄 Angebot"],["rechnung","💶 Rechnung"],["haccp","✓ HACCP"]].map(([t,l])=>(
                  <Btn key={t} variant="outline" sm full onClick={()=>generate(t)} disabled={genType===t}>
                    {genType===t ? "Generiere..." : l}
                  </Btn>
                ))}
              </div>
            </Card>

            {genResult && (
              <Card style={{ border:`1px solid ${T.gold}33` }}>
                <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:10 }}>
                  <div style={{ color:T.gold, fontSize:12, fontWeight:700, textTransform:"uppercase", letterSpacing:1 }}>
                    ✦ {genResult.type}
                  </div>
                  <Btn sm variant="ghost" onClick={()=>setGenResult(null)}>✕</Btn>
                </div>
                <div style={{ color:T.text, fontSize:12, lineHeight:1.75, whiteSpace:"pre-wrap", maxHeight:400, overflowY:"auto" }}>
                  {genResult.content}
                </div>
              </Card>
            )}
          </div>
        )}
      </div>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// EMAILS (Gmail Integration)
// ─────────────────────────────────────────────────────────────────────────────
function EmailModule({ emails, setEmails }) {
  const [selected, setSelected] = useState(null);
  const [draft, setDraft]       = useState("");
  const [draftType, setDraftType] = useState(null);
  const [loading, setLoading]   = useState(false);
  const [filter, setFilter]     = useState("all");

  const email = emails.find(e=>e.id===selected);

  const pick = (id) => {
    setSelected(id);
    setEmails(em=>em.map(e=>e.id===id?{...e,read:true}:e));
    setDraft(""); setDraftType(null);
  };

  const generateDraft = async (type) => {
    if (!email) return;
    setLoading(true); setDraftType(type);
    const prompts = {
      confirm: `Schreibe eine professionelle Bestätigungs-E-Mail (auf Deutsch).\nVon: ${email.from}\nBetreff: ${email.subject}\nInhalt: ${email.body}\nTon: freundlich, verbindlich, professionell.`,
      offer:   `Schreibe ein Catering-Angebot per E-Mail (auf Deutsch).\nVon: ${email.from}\nBetreff: ${email.subject}\nInhalt: ${email.body}\nEnthält: Begrüßung, Leistungsübersicht, Preisindikation, nächste Schritte, Grußformel.`,
      info:    `Schreibe eine E-Mail die höflich weitere Informationen anfragt (auf Deutsch).\nVon: ${email.from}\nBetreff: ${email.subject}\nFrage nach: exaktem Datum, Personenzahl, Budget, Allergien, Location, Wünsche.`,
      decline: `Schreibe eine freundliche, wertschätzende Absage (auf Deutsch).\nVon: ${email.from}\nBetreff: ${email.subject}\nTon: bedauernd, Tür für Zukunft offen lassen, Alternativtermin andeuten.`,
      followup:`Schreibe eine freundliche Nachfass-E-Mail (auf Deutsch) für ein gesendetes Angebot.\nVon: ${email.from}\nBetreff: ${email.subject}\nTon: nicht aufdringlich, hilfsbereit.`,
    };
    const r = await callClaude(SYSTEM_PROMPT, prompts[type]);
    setDraft(r);
    setLoading(false);
  };

  const catColor = { inquiry:T.gold, supplier:T.blue, venue:T.green };
  const catLabel = { inquiry:"Anfrage", supplier:"Lieferant", venue:"Location" };
  const filtered = filter==="all" ? emails : emails.filter(e=>e.cat===filter);
  const unread = emails.filter(e=>!e.read).length;

  return (
    <div>
      <SectionTitle title="E-Mail & Gmail"
        sub={`${unread} ungelesen · Gmail OAuth einrichten → .env`}
        action={
          <div style={{ display:"flex", gap:8 }}>
            <Btn variant="outline" sm>↻ Sync</Btn>
            <a href="https://console.cloud.google.com" target="_blank" rel="noreferrer">
              <Btn sm>Gmail verbinden →</Btn>
            </a>
          </div>
        }/>

      {/* Setup Banner */}
      <Card style={{ marginBottom:16, border:`1px solid ${T.gold}44`, background:T.goldGlow }}>
        <div style={{ color:T.gold, fontWeight:700, marginBottom:10, fontSize:14 }}>📧 Gmail OAuth Setup – einmalig</div>
        <div style={{ display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:12 }}>
          {[
            ["Schritt 1", "Google Cloud Console → Neues Projekt → Gmail API aktivieren", "console.cloud.google.com"],
            ["Schritt 2", "Credentials → OAuth 2.0 Client ID → Web App → Redirect URI der App eintragen", null],
            ["Schritt 3", "Client ID in .env: VITE_GOOGLE_CLIENT_ID=... → Vercel neu deployen", null],
          ].map(([t,d,link])=>(
            <div key={t} style={{ background:T.surface, borderRadius:10, padding:12 }}>
              <div style={{ color:T.gold, fontSize:12, fontWeight:700, marginBottom:4 }}>{t}</div>
              <div style={{ color:T.muted, fontSize:11, lineHeight:1.6 }}>{d}</div>
              {link && <a href={`https://${link}`} target="_blank" rel="noreferrer" style={{ color:T.gold, fontSize:11 }}>{link} →</a>}
            </div>
          ))}
        </div>
      </Card>

      <div style={{ display:"grid", gridTemplateColumns:"300px 1fr", gap:16, minHeight:500 }}>
        {/* List */}
        <Card style={{ padding:0, overflow:"hidden" }}>
          <div style={{ padding:"12px 14px", borderBottom:`1px solid ${T.border}`, display:"flex", gap:6 }}>
            {[["all","Alle"],["inquiry","Anfragen"],["supplier","Lieferanten"],["venue","Locations"]].map(([v,l])=>(
              <span key={v} onClick={()=>setFilter(v)} style={{
                color:filter===v?T.gold:T.muted, fontSize:11, cursor:"pointer",
                padding:"3px 8px", borderRadius:6, background:filter===v?T.goldGlow:"transparent",
                fontWeight:filter===v?600:400
              }}>{l}</span>
            ))}
          </div>
          {filtered.map(e=>(
            <div key={e.id} onClick={()=>pick(e.id)} style={{
              padding:"12px 14px", borderBottom:`1px solid ${T.border}`, cursor:"pointer",
              background: selected===e.id ? T.goldGlow : !e.read ? T.surface+"99" : "transparent",
              borderLeft:`3px solid ${selected===e.id ? T.gold : !e.read ? T.blue : "transparent"}`
            }}>
              <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                <span style={{ color:e.read?T.muted:T.text, fontSize:13, fontWeight:e.read?400:600, overflow:"hidden", textOverflow:"ellipsis", whiteSpace:"nowrap", maxWidth:160 }}>{e.name}</span>
                <div style={{ display:"flex", gap:4, alignItems:"center", flexShrink:0 }}>
                  {e.urgent && <span style={{ color:T.red, fontSize:10 }}>●</span>}
                  <span style={{ color:T.muted, fontSize:10 }}>{e.date}</span>
                </div>
              </div>
              <div style={{ color:T.text, fontSize:12, fontWeight:e.read?400:500, marginBottom:3, overflow:"hidden", textOverflow:"ellipsis", whiteSpace:"nowrap" }}>{e.subject}</div>
              <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center" }}>
                <span style={{ color:T.muted, fontSize:11, overflow:"hidden", textOverflow:"ellipsis", whiteSpace:"nowrap", flex:1 }}>{e.body.slice(0,60)}...</span>
                {e.cat && <Badge color={catColor[e.cat]||T.muted} sm>{catLabel[e.cat]}</Badge>}
              </div>
            </div>
          ))}
        </Card>

        {/* Detail */}
        {email ? (
          <div style={{ display:"flex", flexDirection:"column", gap:12 }}>
            <Card>
              <div style={{ color:T.text, fontWeight:700, fontSize:16, marginBottom:8 }}>{email.subject}</div>
              <div style={{ display:"flex", gap:12, color:T.muted, fontSize:12, marginBottom:16, flexWrap:"wrap" }}>
                <span>Von: {email.from}</span>
                <span>{email.date}</span>
                {email.pax>0 && <span>👥 ca. {email.pax} Pers.</span>}
                {email.cat && <Badge color={catColor[email.cat]||T.muted} sm>{catLabel[email.cat]}</Badge>}
              </div>
              <div style={{ color:T.text, fontSize:13, lineHeight:1.8, padding:"16px 0", borderTop:`1px solid ${T.border}`, borderBottom:`1px solid ${T.border}`, marginBottom:16, whiteSpace:"pre-wrap" }}>
                {email.body}
              </div>
              <div>
                <div style={{ color:T.muted, fontSize:11, textTransform:"uppercase", letterSpacing:1, marginBottom:10 }}>✦ KI-Antwort generieren</div>
                <div style={{ display:"flex", gap:8, flexWrap:"wrap" }}>
                  {[["confirm","✓ Bestätigung"],["offer","📄 Angebot"],["info","? Infos anfragen"],["followup","↻ Nachfassen"],["decline","✗ Absage"]].map(([t,l])=>(
                    <Btn key={t} variant={t==="decline"?"danger":"outline"} sm onClick={()=>generateDraft(t)} disabled={loading}>
                      {loading&&draftType===t ? "Generiere..." : l}
                    </Btn>
                  ))}
                </div>
              </div>
            </Card>

            {(draft||loading) && (
              <Card style={{ border:`1px solid ${T.gold}44` }}>
                <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:10 }}>
                  <div style={{ color:T.gold, fontWeight:700, fontSize:13 }}>
                    ✦ Entwurf: {draftType}
                  </div>
                  <div style={{ display:"flex", gap:8 }}>
                    <Btn sm>📤 In Gmail öffnen</Btn>
                    <Btn variant="ghost" sm onClick={()=>generateDraft(draftType)}>↻ Neu</Btn>
                    <Btn variant="ghost" sm onClick={()=>{setDraft("");setDraftType(null);}}>✕</Btn>
                  </div>
                </div>
                {loading ? (
                  <div style={{ color:T.muted, fontSize:13, padding:"24px 0", textAlign:"center" }}>✦ Formuliere Antwort...</div>
                ) : (
                  <Input value={draft} onChange={e=>setDraft(e.target.value)} rows={14}/>
                )}
                {!loading && (
                  <div style={{ display:"flex", gap:8, marginTop:12 }}>
                    <Btn>📤 Als Entwurf in Gmail speichern</Btn>
                    <Btn variant="ghost" sm onClick={()=>{setDraft("");setDraftType(null);}}>Verwerfen</Btn>
                  </div>
                )}
              </Card>
            )}
          </div>
        ) : (
          <Card style={{ display:"flex", alignItems:"center", justifyContent:"center", color:T.muted, fontSize:14, minHeight:300 }}>
            E-Mail auswählen
          </Card>
        )}
      </div>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// REZEPTE
// ─────────────────────────────────────────────────────────────────────────────
function RecipeModule({ recipes }) {
  const [sel, setSel]   = useState(0);
  const [scale, setScale] = useState(10);
  const [aiHelp, setAiHelp] = useState(""); 
  const [loading, setLoading] = useState(false);
  const r = recipes[sel];
  const factor = scale / r.base;
  const totalCost = r.ingredients.reduce((s,z)=>{
    const m = z.type==="linear" ? z.amount*factor : z.amount;
    return s + m*z.price;
  }, 0);

  const getHelp = async (type) => {
    setLoading(true); setAiHelp("");
    const prompts = {
      scale:   `Gibt es besondere Hinweise beim Skalieren von "${r.name}" auf ${scale} Portionen? Worauf achten?`,
      haccp:   `Erstelle eine HACCP-Checkliste für "${r.name}" (kritische Kontrollpunkte, Temperaturen, Lagerung).`,
      einkauf: `Erstelle eine Einkaufsliste für "${r.name}" skaliert auf ${scale} Portionen. Mit Lieferant-Empfehlungen.`,
      label:   `Erstelle ein Etikett-Text für "${r.name}" (Allergene, Haltbarkeit, Lagerhinweise, Produktionsdatum).`,
    };
    const r2 = await callClaude(SYSTEM_PROMPT, prompts[type]);
    setAiHelp(r2);
    setLoading(false);
  };

  return (
    <div>
      <SectionTitle title="Rezeptbuch" sub={`${recipes.length} Rezepte`} action={<Btn>+ Neues Rezept</Btn>}/>
      <div style={{ display:"flex", gap:8, marginBottom:20, flexWrap:"wrap" }}>
        {recipes.map((rec,i)=>(
          <Btn key={i} variant={sel===i?"primary":"ghost"} onClick={()=>{setSel(i);setScale(rec.base);setAiHelp("");}}>
            {rec.name}
          </Btn>
        ))}
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"1fr 300px", gap:16 }}>
        <Card>
          <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:22 }}>
            <div>
              <div style={{ color:T.text, fontWeight:700, fontSize:20, fontFamily:"'Playfair Display',serif" }}>{r.name}</div>
              <div style={{ color:T.muted, fontSize:12, marginTop:3 }}>{r.cat} · {r.station} · {r.time} Min · Allergene: {r.allergens.join(", ")}</div>
            </div>
            <div style={{ display:"flex", alignItems:"center", gap:10 }}>
              <span style={{ color:T.muted, fontSize:12 }}>Portionen:</span>
              <Btn variant="ghost" sm onClick={()=>setScale(Math.max(1,scale-5))}>−</Btn>
              <span style={{ color:T.gold, fontWeight:700, fontSize:24, minWidth:36, textAlign:"center" }}>{scale}</span>
              <Btn variant="ghost" sm onClick={()=>setScale(scale+5)}>+</Btn>
            </div>
          </div>

          <table style={{ width:"100%", borderCollapse:"collapse", marginBottom:20 }}>
            <thead>
              <tr style={{ borderBottom:`1px solid ${T.border}` }}>
                {["Zutat","Basis","Skaliert auf "+scale,"Typ","Kosten"].map(h=>(
                  <th key={h} style={{ color:T.muted, fontSize:11, textAlign:"left", padding:"8px 6px", fontWeight:500, letterSpacing:0.4 }}>{h}</th>
                ))}
              </tr>
            </thead>
            <tbody>
              {r.ingredients.map(z=>{
                const sc = z.type==="linear" ? (z.amount*factor).toFixed(1) : z.amount;
                const cost = (z.type==="linear" ? z.amount*factor : z.amount) * z.price;
                return (
                  <tr key={z.id} style={{ borderBottom:`1px solid ${T.border}22` }}>
                    <td style={{ color:T.text, padding:"10px 6px", fontSize:13 }}>{z.name}</td>
                    <td style={{ color:T.muted, fontSize:13 }}>{z.amount} {z.unit}</td>
                    <td style={{ color:T.gold, fontSize:13, fontWeight:600 }}>{sc} {z.unit}</td>
                    <td><Badge color={z.type==="linear"?T.green:T.blue} sm>{z.type==="linear"?"linear":"gedeckelt"}</Badge></td>
                    <td style={{ color:T.muted, fontSize:13 }}>€{cost.toFixed(2)}</td>
                  </tr>
                );
              })}
            </tbody>
          </table>

          <div style={{ background:T.surface, borderRadius:10, padding:14 }}>
            <div style={{ color:T.muted, fontSize:11, textTransform:"uppercase", letterSpacing:1, marginBottom:10 }}>Zubereitung</div>
            {r.steps.map((s,i)=>(
              <div key={i} style={{ display:"flex", gap:12, padding:"8px 0", borderBottom:i<r.steps.length-1?`1px solid ${T.border}`:"none" }}>
                <div style={{ width:24, height:24, borderRadius:"50%", background:T.goldGlow, border:`1px solid ${T.gold}44`, display:"flex", alignItems:"center", justifyContent:"center", color:T.gold, fontSize:11, fontWeight:700, flexShrink:0 }}>{i+1}</div>
                <span style={{ color:T.text, fontSize:13, lineHeight:1.5 }}>{s}</span>
              </div>
            ))}
          </div>
        </Card>

        <div style={{ display:"flex", flexDirection:"column", gap:12 }}>
          <Card>
            <div style={{ color:T.muted, fontSize:11, textTransform:"uppercase", letterSpacing:1, marginBottom:14 }}>Kalkulation</div>
            {[
              ["Portionen", scale, T.gold],
              ["Warenkosten", `€${totalCost.toFixed(2)}`, T.gold],
              ["Kosten/Portion", `€${(totalCost/scale).toFixed(2)}`, T.green],
              ["VK 30% WE", `€${((totalCost/scale)/0.3).toFixed(2)}`, T.text],
              ["DB-Beitrag", `${(100-(totalCost/scale/((totalCost/scale)/0.3)*100)).toFixed(0)}%`, T.green],
            ].map(([l,v,c])=>(
              <div key={l} style={{ display:"flex", justifyContent:"space-between", padding:"9px 0", borderBottom:`1px solid ${T.border}` }}>
                <span style={{ color:T.muted, fontSize:12 }}>{l}</span>
                <span style={{ color:c, fontSize:13, fontWeight:600 }}>{v}</span>
              </div>
            ))}
          </Card>

          <Card>
            <div style={{ color:T.muted, fontSize:11, textTransform:"uppercase", letterSpacing:1, marginBottom:10 }}>✦ KI & Export</div>
            {[["scale","💡 Skalier-Tipps"],["haccp","✓ HACCP"],["einkauf","📋 Einkaufsliste"],["label","🏷 Etikett"]].map(([t,l])=>(
              <Btn key={t} variant="ghost" full style={{ textAlign:"left", marginBottom:6, justifyContent:"flex-start" }} sm onClick={()=>getHelp(t)} disabled={loading}>
                {loading?"Generiere...":l}
              </Btn>
            ))}
          </Card>

          {aiHelp && (
            <Card style={{ border:`1px solid ${T.gold}33` }}>
              <div style={{ color:T.text, fontSize:12, lineHeight:1.7, whiteSpace:"pre-wrap", maxHeight:300, overflowY:"auto" }}>
                {aiHelp}
              </div>
            </Card>
          )}
        </div>
      </div>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// AUFGABEN
// ─────────────────────────────────────────────────────────────────────────────
function TaskModule({ tasks, setTasks }) {
  const [filter, setFilter] = useState("all");
  const cycle = (id) => setTasks(t=>t.map(x=>x.id===id?{...x,status:TASK_STATUS[x.status].next}:x));
  const today = new Date().toISOString().split("T")[0];
  const shown = filter==="all" ? tasks : tasks.filter(t=>t.status===filter);

  return (
    <div>
      <SectionTitle title="Aufgaben" action={<Btn>+ Neue Aufgabe</Btn>}/>
      <div style={{ display:"flex", gap:12, marginBottom:20, flexWrap:"wrap" }}>
        {[["open","Offen",T.muted],["working","In Arbeit",T.gold],["done","Erledigt",T.green]].map(([s,l,c])=>(
          <div key={s} onClick={()=>setFilter(filter===s?"all":s)} style={{
            background: filter===s ? c+"22" : T.card, border:`1px solid ${filter===s?c+"44":T.border}`,
            borderRadius:10, padding:"10px 20px", cursor:"pointer"
          }}>
            <div style={{ color:c, fontSize:22, fontWeight:700, lineHeight:1 }}>{tasks.filter(t=>t.status===s).length}</div>
            <div style={{ color:T.muted, fontSize:11, marginTop:3 }}>{l}</div>
          </div>
        ))}
      </div>

      <Card>
        <table style={{ width:"100%", borderCollapse:"collapse" }}>
          <thead>
            <tr style={{ borderBottom:`1px solid ${T.border}` }}>
              {["","Aufgabe","Priorität","Person","Deadline","Modul"].map(h=>(
                <th key={h} style={{ color:T.muted, fontSize:11, textAlign:"left", padding:"8px 12px", fontWeight:500, letterSpacing:0.4 }}>{h}</th>
              ))}
            </tr>
          </thead>
          <tbody>
            {shown.map(t=>{
              const member = TEAM.find(m=>m.id===t.person);
              const isOverdue = t.deadline && t.deadline < today && t.status!=="done";
              return (
                <tr key={t.id} style={{ borderBottom:`1px solid ${T.border}22`, opacity:t.status==="done"?0.45:1 }}>
                  <td style={{ padding:"12px" }}>
                    <span onClick={()=>cycle(t.id)} style={{ fontSize:18, cursor:"pointer", color:TASK_STATUS[t.status].color }}>{TASK_STATUS[t.status].icon}</span>
                  </td>
                  <td style={{ color:T.text, fontSize:13, padding:"12px", textDecoration:t.status==="done"?"line-through":"none" }}>{t.title}</td>
                  <td style={{ padding:"12px" }}><Badge color={PRIO_COLOR[t.prio]} sm>{PRIO_LABEL[t.prio]}</Badge></td>
                  <td style={{ padding:"12px" }}>
                    {member ? (
                      <div style={{ display:"flex", alignItems:"center", gap:6 }}>
                        <Avatar member={member} size={22}/>
                        <span style={{ color:T.muted, fontSize:12 }}>{member.name}</span>
                      </div>
                    ) : <span style={{ color:T.muted, fontSize:12 }}>{t.person}</span>}
                  </td>
                  <td style={{ padding:"12px" }}>
                    <span style={{ color:isOverdue?T.red:t.deadline===today?T.gold:T.muted, fontSize:12, fontWeight:isOverdue||t.deadline===today?700:400 }}>
                      {t.deadline ? fmtDateShort(t.deadline) : "–"}
                      {isOverdue && " ⚠"}
                    </span>
                  </td>
                  <td style={{ color:T.muted, fontSize:11, padding:"12px" }}>{t.module}</td>
                </tr>
              );
            })}
          </tbody>
        </table>
      </Card>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// FINANZEN
// ─────────────────────────────────────────────────────────────────────────────
function FinanzModule() {
  const months = ["Jan","Feb","Mär","Apr","Mai","Jun"];
  const rev   = [14200,16800,12400,19200,17600,18400];
  const costs = [9800, 11200,8900, 13100,11900,12200];
  const db    = rev.map((r,i)=>r-costs[i]);
  const maxV  = Math.max(...rev);

  return (
    <div>
      <SectionTitle title="Finanzen" sub="H1 2025 · Restaurant & Catering"/>
      <div style={{ display:"flex", gap:12, marginBottom:20, flexWrap:"wrap" }}>
        <KPI label="Umsatz Juni" value="€18.400" sub="+12% vs. Vormonat" icon="💰"/>
        <KPI label="Wareneinsatz" value="28%" sub="€5.152" color={T.green} icon="🥩"/>
        <KPI label="Personalkosten" value="35%" sub="€6.440" color={T.blue} icon="👥"/>
        <KPI label="Deckungsbeitrag" value="€6.248" sub="33.9% Marge" color={T.green} icon="📈"/>
        <KPI label="Offene Rechnungen" value="€3.800" sub="2 Rechnungen" color={T.red} icon="⚠"/>
        <KPI label="Cateringanteil" value="62%" sub="€11.408 Catering" color={T.gold} icon="🍽"/>
      </div>

      <div style={{ display:"grid", gridTemplateColumns:"2fr 1fr", gap:16 }}>
        <Card>
          <div style={{ color:T.text, fontWeight:600, marginBottom:20 }}>Umsatz, Kosten & DB – H1 2025</div>
          <div style={{ display:"flex", alignItems:"flex-end", gap:10, height:190 }}>
            {months.map((m,i)=>(
              <div key={m} style={{ flex:1, display:"flex", flexDirection:"column", alignItems:"center", gap:5 }}>
                <div style={{ width:"100%", display:"flex", gap:2, alignItems:"flex-end", height:160 }}>
                  <div style={{ flex:1, background:T.gold, borderRadius:"4px 4px 0 0", height:`${(rev[i]/maxV)*160}px`, transition:"height 0.6s" }}/>
                  <div style={{ flex:1, background:T.faint, borderRadius:"4px 4px 0 0", height:`${(costs[i]/maxV)*160}px`, transition:"height 0.6s" }}/>
                  <div style={{ flex:1, background:T.green+"99", borderRadius:"4px 4px 0 0", height:`${(db[i]/maxV)*160}px`, transition:"height 0.6s" }}/>
                </div>
                <div style={{ color:T.muted, fontSize:11 }}>{m}</div>
              </div>
            ))}
          </div>
          <div style={{ display:"flex", gap:16, marginTop:12 }}>
            {[[T.gold,"Umsatz"],[T.faint,"Kosten"],[T.green+"99","Deckungsbeitrag"]].map(([c,l])=>(
              <div key={l} style={{ display:"flex", alignItems:"center", gap:6 }}>
                <div style={{ width:12, height:12, background:c, borderRadius:2 }}/>
                <span style={{ color:T.muted, fontSize:11 }}>{l}</span>
              </div>
            ))}
          </div>
        </Card>

        <div style={{ display:"flex", flexDirection:"column", gap:12 }}>
          <Card>
            <div style={{ color:T.text, fontWeight:600, marginBottom:12 }}>Offene Rechnungen</div>
            {[["Hochzeit Schmidt","€2.400","15. Jun",T.red],["Catering Siemens","€1.400","22. Jun",T.gold]].map(([n,b,d,c])=>(
              <div key={n} style={{ display:"flex", justifyContent:"space-between", padding:"10px 0", borderBottom:`1px solid ${T.border}` }}>
                <div>
                  <div style={{ color:T.text, fontSize:13 }}>{n}</div>
                  <div style={{ color:T.muted, fontSize:11 }}>Fällig: {d}</div>
                </div>
                <div style={{ color:c, fontWeight:700, fontSize:14 }}>{b}</div>
              </div>
            ))}
          </Card>
          <Card>
            <div style={{ color:T.text, fontWeight:600, marginBottom:12 }}>Kostenstruktur</div>
            {[["Wareneinsatz","28%",T.gold],["Personal","35%",T.blue],["Fixkosten","18%",T.muted],["Marketing","4%",T.purple],["Sonstiges","6%",T.faint]].map(([k,v,c])=>(
              <div key={k} style={{ marginBottom:10 }}>
                <div style={{ display:"flex", justifyContent:"space-between", marginBottom:4 }}>
                  <span style={{ color:T.muted, fontSize:12 }}>{k}</span>
                  <span style={{ color:c, fontSize:12, fontWeight:600 }}>{v}</span>
                </div>
                <div style={{ background:T.border, borderRadius:4, height:5 }}>
                  <div style={{ background:c, width:v, height:5, borderRadius:4, transition:"width 0.6s" }}/>
                </div>
              </div>
            ))}
          </Card>
        </div>
      </div>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// PERSONAL
// ─────────────────────────────────────────────────────────────────────────────
function PersonalModule() {
  const shifts = {
    "Mo 09.": ["chef","marco","lisa","thomas"],
    "Di 10.": ["chef","marco","thomas"],
    "Mi 11.": ["chef","lisa"],
    "Do 12.": ["chef","marco","lisa","thomas"],
    "Fr 13.": ["chef","marco","lisa","thomas"],
    "Sa 14.": ["marco","lisa","thomas"],
    "So 15.": ["marco","thomas"],
  };
  return (
    <div>
      <SectionTitle title="Personal & Schichten" sub="KW 24 · 2025" action={<Btn>+ Schicht planen</Btn>}/>
      <div style={{ display:"flex", gap:12, marginBottom:20 }}>
        {TEAM.map(m=>(
          <Card key={m.id} style={{ flex:1, textAlign:"center" }}>
            <Avatar member={m} size={48}/>
            <div style={{ color:T.text, fontSize:14, fontWeight:600, marginTop:10 }}>{m.name}</div>
            <div style={{ color:T.muted, fontSize:12, marginTop:2 }}>{m.role}</div>
            <div style={{ color:m.color, fontSize:11, marginTop:6 }}>
              {Object.values(shifts).filter(s=>s.includes(m.id)).length} Schichten
            </div>
          </Card>
        ))}
      </div>
      <Card>
        <div style={{ color:T.text, fontWeight:600, marginBottom:14 }}>Schichtplan</div>
        <table style={{ width:"100%", borderCollapse:"collapse" }}>
          <thead>
            <tr style={{ borderBottom:`1px solid ${T.border}` }}>
              <th style={{ color:T.muted, fontSize:11, textAlign:"left", padding:"8px 12px", fontWeight:500 }}>Tag</th>
              {TEAM.map(m=>(
                <th key={m.id} style={{ color:T.muted, fontSize:11, padding:"8px 12px", fontWeight:500, textAlign:"center" }}>{m.name}</th>
              ))}
              <th style={{ color:T.muted, fontSize:11, padding:"8px 12px", fontWeight:500 }}>Besetzt</th>
            </tr>
          </thead>
          <tbody>
            {Object.entries(shifts).map(([day,workers])=>(
              <tr key={day} style={{ borderBottom:`1px solid ${T.border}22` }}>
                <td style={{ color:T.text, padding:"11px 12px", fontSize:13, fontWeight:500 }}>{day}</td>
                {TEAM.map(m=>(
                  <td key={m.id} style={{ textAlign:"center", padding:"11px 12px" }}>
                    {workers.includes(m.id) ? <span style={{ color:m.color, fontSize:18 }}>●</span> : <span style={{ color:T.faint }}>–</span>}
                  </td>
                ))}
                <td style={{ padding:"11px 12px" }}>
                  <Badge color={workers.length>=3?T.green:workers.length>=2?T.gold:T.red}>
                    {workers.length}/{TEAM.length}
                  </Badge>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </Card>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// AI ASSISTENT
// ─────────────────────────────────────────────────────────────────────────────
function AssistantModule() {
  const [msgs, setMsgs]   = useState([{
    role:"assistant",
    text:"Guten Morgen! Ich bin ChefOS.\n\nIch helfe dir mit:\n• 📧 E-Mail-Anfragen analysieren & beantworten\n• 📋 Einkaufslisten & Prep-Pläne erstellen\n• 💰 Kalkulationen & Angebote formulieren\n• 🔪 Rezepte skalieren\n• 📅 Ablaufpläne & Produktionspläne\n• 🎙 Voice Notes strukturieren\n• ⚠ HACCP & Lebensmittelsicherheit\n\nWas steht heute an?"
  }]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const bottomRef = useRef(null);

  useEffect(()=>bottomRef.current?.scrollIntoView({behavior:"smooth"}),[msgs]);

  const send = async (msg) => {
    const txt = (msg||input).trim();
    if (!txt||loading) return;
    setInput("");
    setMsgs(m=>[...m,{role:"user",text:txt}]);
    setLoading(true);
    const history = msgs.map(m=>({role:m.role,content:m.text}));
    const r = await callClaude(SYSTEM_PROMPT, txt, history);
    setMsgs(m=>[...m,{role:"assistant",text:r}]);
    setLoading(false);
  };

  const prompts = [
    "Cateringanfrage analysieren",
    "Einkaufsliste für 80 Personen",
    "HACCP Kühlketten-Checkliste",
    "Angebot Hochzeit 120 Pers.",
    "Wareneinsatz 28% – ist das gut?",
    "Prep-Plan für morgen erstellen"
  ];

  return (
    <div style={{ display:"flex", flexDirection:"column", height:"calc(100vh - 120px)" }}>
      <SectionTitle title="✦ AI Assistent" sub="ChefOS · Dein persönlicher Betriebsassistent"/>
      <div style={{ display:"flex", gap:6, marginBottom:14, flexWrap:"wrap" }}>
        {prompts.map(p=>(
          <button key={p} onClick={()=>send(p)} style={{
            background:T.surface, color:T.muted, border:`1px solid ${T.border}`,
            padding:"5px 13px", borderRadius:20, fontSize:12, cursor:"pointer"
          }}>{p}</button>
        ))}
      </div>

      <div style={{ flex:1, overflowY:"auto", display:"flex", flexDirection:"column", gap:10, marginBottom:14 }}>
        {msgs.map((m,i)=>(
          <div key={i} style={{ display:"flex", justifyContent:m.role==="user"?"flex-end":"flex-start" }}>
            <div style={{
              maxWidth:"82%", padding:"12px 16px", borderRadius:14,
              background: m.role==="user" ? T.gold : T.card,
              color: m.role==="user" ? "#000" : T.text,
              border: m.role==="assistant" ? `1px solid ${T.border}` : "none",
              fontSize:13, lineHeight:1.75, whiteSpace:"pre-wrap"
            }}>
              {m.role==="assistant" && (
                <div style={{ color:T.gold, fontWeight:700, fontSize:10, marginBottom:6, letterSpacing:1.2 }}>✦ CHEFOS</div>
              )}
              {m.text}
            </div>
          </div>
        ))}
        {loading && (
          <div style={{ display:"flex" }}>
            <div style={{ background:T.card, border:`1px solid ${T.border}`, borderRadius:14, padding:"12px 16px", color:T.muted, fontSize:13 }}>
              <span style={{ color:T.gold }}>✦ </span>Analysiere...
            </div>
          </div>
        )}
        <div ref={bottomRef}/>
      </div>

      <div style={{ display:"flex", gap:10 }}>
        <Input
          value={input}
          onChange={e=>setInput(e.target.value)}
          onKeyDown={e=>e.key==="Enter"&&!e.shiftKey&&send()}
          placeholder="Frage stellen oder Aufgabe beschreiben... (Enter)"
        />
        <Btn onClick={()=>send()} disabled={loading} style={{ fontSize:18, padding:"10px 20px" }}>↑</Btn>
      </div>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// LAUNCH GUIDE
// ─────────────────────────────────────────────────────────────────────────────
function LaunchGuide() {
  const [done, setDone] = useState({});
  const steps = [
    { id:"vite",    title:"Vite React Projekt erstellen",
      cmd:"npm create vite@latest chefos -- --template react\ncd chefos\nnpm install",
      info:"Node.js muss installiert sein. Download: nodejs.org" },
    { id:"files",   title:"ChefOS Dateien einfügen",
      cmd:"# App.jsx → src/App.jsx ersetzen\n# index.html → root ersetzen\n# vercel.json → root einfügen",
      info:"Alle Dateien aus dem ZIP in den chefos/ Ordner kopieren" },
    { id:"env",     title:".env Datei anlegen",
      cmd:"# Datei .env im Projektordner erstellen:\nVITE_ANTHROPIC_KEY=sk-ant-api03-DEIN-KEY\nVITE_GOOGLE_CLIENT_ID=DEIN-CLIENT-ID.apps.googleusercontent.com",
      info:"API Key: console.anthropic.com → API Keys → Create Key" },
    { id:"test",    title:"Lokal testen",
      cmd:"npm run dev\n# → http://localhost:5173 im Browser öffnen",
      info:"App sollte jetzt vollständig funktionieren" },
    { id:"vercel",  title:"Auf Vercel deployen",
      cmd:"npm install -g vercel\nvercel login\nvercel deploy --prod\n# → URL wird angezeigt (z.B. chefos.vercel.app)",
      info:"Vercel Account kostenlos erstellen: vercel.com" },
    { id:"envvars", title:"API Keys in Vercel eintragen",
      cmd:"# Vercel Dashboard → Projekt → Settings → Environment Variables\n# VITE_ANTHROPIC_KEY = sk-ant-api03-...\n# VITE_GOOGLE_CLIENT_ID = ...\n# Danach: vercel --prod (neu deployen)",
      info:"Wichtig: Ohne diesen Schritt funktioniert die KI nicht live" },
    { id:"gmail",   title:"Gmail OAuth einrichten",
      cmd:"# 1. console.cloud.google.com → Neues Projekt\n# 2. APIs & Services → Gmail API → Aktivieren\n# 3. Credentials → OAuth 2.0 Client ID → Web App\n# 4. Redirect URI: https://deine-app.vercel.app/auth\n# 5. Client ID in Vercel Env Variables eintragen",
      info:"Einmalig 15 Min. Aufwand. Danach Posteingang direkt in ChefOS" },
    { id:"pwa",     title:"Als App auf iPhone/iPad installieren",
      cmd:"# Safari → chefos.vercel.app öffnen\n# Teilen-Symbol → 'Zum Home-Bildschirm'\n# Team-Mitglieder machen dasselbe",
      info:"Funktioniert wie eine native App – kein App Store nötig" },
  ];

  return (
    <div>
      <SectionTitle title="🚀 Launch Guide" sub="Von 0 zur live App in ~30 Minuten"/>
      <div style={{ display:"grid", gap:12 }}>
        {steps.map(s=>(
          <Card key={s.id} style={{ border:`1px solid ${done[s.id]?T.green+"44":T.gold+"22"}`, background:done[s.id]?T.greenDim:T.card }}>
            <div style={{ display:"flex", gap:14, alignItems:"flex-start" }}>
              <div onClick={()=>setDone(d=>({...d,[s.id]:!d[s.id]}))} style={{
                width:34, height:34, borderRadius:"50%",
                background: done[s.id] ? T.green+"22" : T.goldGlow,
                border:`1px solid ${done[s.id]?T.green+"66":T.gold+"44"}`,
                display:"flex", alignItems:"center", justifyContent:"center",
                color: done[s.id] ? T.green : T.gold,
                fontWeight:700, fontSize:14, cursor:"pointer", flexShrink:0
              }}>{done[s.id]?"✓":steps.indexOf(s)+1}</div>
              <div style={{ flex:1 }}>
                <div style={{ color: done[s.id]?T.green:T.text, fontWeight:600, fontSize:14, marginBottom:6 }}>{s.title}</div>
                {s.info && <div style={{ color:T.muted, fontSize:12, marginBottom:8 }}>{s.info}</div>}
                <pre style={{
                  background:T.surface, border:`1px solid ${T.border}`,
                  borderRadius:8, padding:12, margin:0, color:T.gold,
                  fontSize:12, fontFamily:"'Courier New',Courier,monospace",
                  lineHeight:1.7, overflowX:"auto", whiteSpace:"pre-wrap"
                }}>{s.cmd}</pre>
              </div>
            </div>
          </Card>
        ))}
      </div>
      <Card style={{ marginTop:14, border:`1px solid ${T.green}44`, background:T.greenDim }}>
        <div style={{ color:T.green, fontWeight:700, fontSize:14, marginBottom:6 }}>
          ✓ {Object.values(done).filter(Boolean).length}/{steps.length} Schritte erledigt · Geschätzte Zeit: 20–30 Min.
        </div>
        <div style={{ color:T.muted, fontSize:13, lineHeight:1.7 }}>
          Kein Entwickler-Wissen erforderlich. Alle Befehle sind Copy-Paste. Bei Fragen: den AI Assistenten fragen.
        </div>
      </Card>
    </div>
  );
}

// ─────────────────────────────────────────────────────────────────────────────
// APP SHELL
// ─────────────────────────────────────────────────────────────────────────────
const NAV = [
  { id:"dashboard", icon:"◈",  label:"Dashboard" },
  { id:"catering",  icon:"🍽", label:"Catering"  },
  { id:"emails",    icon:"📧", label:"E-Mails"   },
  { id:"recipes",   icon:"📖", label:"Rezepte"   },
  { id:"tasks",     icon:"✅", label:"Aufgaben"  },
  { id:"finanzen",  icon:"💰", label:"Finanzen"  },
  { id:"personal",  icon:"👥", label:"Personal"  },
  { id:"assistant", icon:"✦",  label:"AI Assistent" },
  { id:"launch",    icon:"🚀", label:"Launch Guide" },
];

export default function App() {
  const [active,      setActive]      = useState("dashboard");
  const [collapsed,   setCollapsed]   = useState(false);
  const [voiceOpen,   setVoiceOpen]   = useState(false);
  const [voiceSender, setVoiceSender] = useState(TEAM[0]);
  const [caterings,   setCaterings]   = useState(INIT_CATERINGS);
  const [tasks,       setTasks]       = useState(INIT_TASKS);
  const [emails,      setEmails]      = useState(INIT_EMAILS);
  const unread = emails.filter(e=>!e.read).length;

  const handleVoiceResult = (transcript, analysis) => {
    setTasks(t=>[...t, {
      id:Date.now(), status:"open", prio:"mid",
      title:`Voice: ${transcript.slice(0,55)}...`,
      person: voiceSender.id, deadline:"", module:"voice"
    }]);
  };

  const renderModule = () => {
    switch(active) {
      case "dashboard": return <Dashboard caterings={caterings} tasks={tasks} emails={emails} setActive={setActive}/>;
      case "catering":  return <CateringModule caterings={caterings} setCaterings={setCaterings}/>;
      case "emails":    return <EmailModule emails={emails} setEmails={setEmails}/>;
      case "recipes":   return <RecipeModule recipes={INIT_RECIPES}/>;
      case "tasks":     return <TaskModule tasks={tasks} setTasks={setTasks}/>;
      case "finanzen":  return <FinanzModule/>;
      case "personal":  return <PersonalModule/>;
      case "assistant": return <AssistantModule/>;
      case "launch":    return <LaunchGuide/>;
      default:          return null;
    }
  };

  return (
    <div style={{ display:"flex", height:"100vh", background:T.bg, color:T.text, fontFamily:"'DM Sans',sans-serif", overflow:"hidden" }}>
      {/* SIDEBAR */}
      <div style={{
        width: collapsed ? 62 : 214,
        background:T.surface, borderRight:`1px solid ${T.border}`,
        display:"flex", flexDirection:"column",
        transition:"width 0.25s ease", overflow:"hidden", flexShrink:0
      }}>
        {/* Logo */}
        <div style={{ padding: collapsed?"16px 12px":"16px 18px", borderBottom:`1px solid ${T.border}`, display:"flex", alignItems:"center", justifyContent:"space-between", minHeight:62 }}>
          {!collapsed && (
            <div>
              <span style={{ color:T.gold, fontFamily:"'Playfair Display',serif", fontSize:21, fontWeight:700, letterSpacing:0.5 }}>Chef</span>
              <span style={{ color:T.goldDim, fontFamily:"'Playfair Display',serif", fontSize:21, fontWeight:400 }}>OS</span>
            </div>
          )}
          <button onClick={()=>setCollapsed(!collapsed)} style={{ background:"transparent", border:"none", color:T.muted, cursor:"pointer", fontSize:13, padding:4, flexShrink:0 }}>
            {collapsed?"▶":"◀"}
          </button>
        </div>

        {/* Voice Note Button */}
        <div style={{ padding: collapsed?"10px 8px":"10px 12px", borderBottom:`1px solid ${T.border}` }}>
          <button onClick={()=>setVoiceOpen(true)} style={{
            width:"100%", background:T.goldGlow, border:`1px solid ${T.gold}44`,
            borderRadius:10, padding: collapsed?"10px 0":"10px 14px",
            cursor:"pointer", display:"flex", alignItems:"center",
            justifyContent: collapsed?"center":"flex-start",
            gap:8, color:T.gold, fontWeight:600, fontSize:13
          }}>
            <span style={{ fontSize:16, flexShrink:0 }}>🎙</span>
            {!collapsed && "Voice Note"}
          </button>
        </div>

        {/* Nav */}
        <nav style={{ flex:1, padding:"6px 0", overflowY:"auto" }}>
          {NAV.map(m=>{
            const isActive = active===m.id;
            const badge = m.id==="emails" ? unread : 0;
            return (
              <div key={m.id} onClick={()=>setActive(m.id)} style={{
                display:"flex", alignItems:"center", gap:10,
                padding: collapsed ? "11px 20px" : "11px 16px",
                cursor:"pointer",
                borderLeft:`2px solid ${isActive?T.gold:"transparent"}`,
                background: isActive ? T.goldGlow : "transparent",
                transition:"all 0.12s"
              }}>
                <span style={{ fontSize:15, minWidth:20, textAlign:"center" }}>{m.icon}</span>
                {!collapsed && (
                  <span style={{ fontSize:13, color:isActive?T.gold:T.muted, fontWeight:isActive?600:400, whiteSpace:"nowrap", flex:1 }}>
                    {m.label}
                  </span>
                )}
                {!collapsed && badge>0 && (
                  <span style={{ background:T.red, color:"#fff", borderRadius:10, fontSize:10, fontWeight:700, padding:"1px 6px", flexShrink:0 }}>
                    {badge}
                  </span>
                )}
              </div>
            );
          })}
        </nav>

        {/* Team Switcher */}
        {!collapsed && (
          <div style={{ padding:"12px 14px", borderTop:`1px solid ${T.border}` }}>
            <div style={{ color:T.muted, fontSize:10, letterSpacing:1, textTransform:"uppercase", marginBottom:8 }}>Voice Note senden als</div>
            <div style={{ display:"flex", gap:4, flexWrap:"wrap" }}>
              {TEAM.map(m=>(
                <div key={m.id} onClick={()=>setVoiceSender(m)} style={{
                  background: voiceSender?.id===m.id ? m.color+"22" : "transparent",
                  border:`1px solid ${voiceSender?.id===m.id?m.color:T.border}`,
                  borderRadius:20, padding:"2px 9px", fontSize:11,
                  color: voiceSender?.id===m.id ? m.color : T.muted,
                  cursor:"pointer", fontWeight:voiceSender?.id===m.id?600:400
                }}>{m.name}</div>
              ))}
            </div>
            <div style={{ color:T.faint, fontSize:10, marginTop:10 }}>ChefOS v1.0</div>
          </div>
        )}
      </div>

      {/* MAIN CONTENT */}
      <div style={{ flex:1, overflowY:"auto", padding:"28px 32px" }}>
        {renderModule()}
      </div>

      {/* VOICE MODAL */}
      {voiceOpen && (
        <VoiceModal
          sender={voiceSender}
          onClose={()=>setVoiceOpen(false)}
          onTaskCreated={handleVoiceResult}
        />
      )}
    </div>
  );
}

import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)

# Anthropic API Key – https://console.anthropic.com
VITE_ANTHROPIC_KEY=sk-ant-api03-DEIN-KEY-HIER

# Google OAuth2 – https://console.cloud.google.com
VITE_GOOGLE_CLIENT_ID=DEINE-CLIENT-ID.apps.googleusercontent.com

node_modules
dist
.env
.env.local

<!doctype html>
<html lang="de">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#0C0C0A" />
    <meta name="apple-mobile-web-app-capable" content="yes" />
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
    <meta name="apple-mobile-web-app-title" content="ChefOS" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600;700&family=DM+Sans:opsz,wght@9..40,300;9..40,400;9..40,500;9..40,600;9..40,700&display=swap" rel="stylesheet" />
    <title>ChefOS</title>
    <style>
      *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
      html, body, #root { height: 100%; }
      body { background: #0C0C0A; color: #EDEAE2; font-family: 'DM Sans', sans-serif; -webkit-font-smoothing: antialiased; }
      ::-webkit-scrollbar { width: 6px; height: 6px; }
      ::-webkit-scrollbar-track { background: transparent; }
      ::-webkit-scrollbar-thumb { background: #2A2A26; border-radius: 3px; }
      textarea, input { font-family: inherit; }
    </style>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>

{
  "name": "chefos",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.4.1"
  }
}

{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" }
      ]
    }
  ]
}
