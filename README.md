import { useState } from "react";

const COLORS = {
  navy: "#0A1628",
  navyLight: "#112240",
  blue: "#1565C0",
  blueAccent: "#1E88E5",
  cyan: "#00B4D8",
  cyanLight: "#90E0EF",
  gold: "#FFB300",
  white: "#F0F4FF",
  gray: "#8892A4",
  grayLight: "#C8D0E0",
  success: "#00C896",
  danger: "#FF4757",
  card: "rgba(17, 34, 64, 0.85)",
};

// ─── PRICING ──────────────────────────────────────────────────────────────────
// N&B: < 20 pages = 1.00 MAD/page | >= 20 pages = 0.50 MAD/page
// Couleur = double du prix N&B
const calcPrice = (pages, color, copies) => {
  const nbRate = pages < 20 ? 1.00 : 0.50;
  const rate = color ? nbRate * 2 : nbRate;
  return parseFloat((pages * copies * rate).toFixed(2));
};

const getPriceLabel = (pages, color) => {
  const nbRate = pages < 20 ? 1.00 : 0.50;
  const rate = color ? nbRate * 2 : nbRate;
  return `${rate.toFixed(2)} MAD/page`;
};

// ─── MOCK DATA ────────────────────────────────────────────────────────────────
const mockUser = {
  name: "Yassine El Mansouri",
  email: "y.elmansouri@cmc.ac.ma",
  id: "CMC-2024-0347",
  balance: 45.50,
  totalPrints: 128,
  pendingJobs: 2,
};

const mockJobs = [
  { id: "JOB-001", ticket: "TKT-8823", file: "Rapport_Stage_Final.pdf", pages: 24, color: false, copies: 2, cost: calcPrice(24,false,2), status: "pending", time: "08:30", date: "Aujourd'hui", desk: "Bureau 3" },
  { id: "JOB-002", ticket: "TKT-8824", file: "Presentation_PFE.pdf", pages: 15, color: true, copies: 1, cost: calcPrice(15,true,1), status: "pending", time: "09:00", date: "Aujourd'hui", desk: "Bureau 1" },
  { id: "JOB-003", ticket: "TKT-8801", file: "CV_Yassine.pdf", pages: 2, color: false, copies: 3, cost: calcPrice(2,false,3), status: "done", time: "14:20", date: "Hier", desk: "Bureau 2" },
  { id: "JOB-004", ticket: "TKT-8800", file: "Lettre_Motivation.pdf", pages: 1, color: false, copies: 1, cost: calcPrice(1,false,1), status: "done", time: "10:05", date: "Hier", desk: "Bureau 1" },
  { id: "JOB-005", ticket: "TKT-8790", file: "Notes_Cours_S6.pdf", pages: 45, color: false, copies: 1, cost: calcPrice(45,false,1), status: "done", time: "16:00", date: "20/04", desk: "Bureau 3" },
];

const mockTransactions = [
  { id: "TXN-001", type: "charge", amount: 50.00, method: "Carte Bancaire", date: "22/04/2025" },
  { id: "TXN-002", type: "print", amount: -calcPrice(24,false,2), method: "Impression", date: "23/04/2025" },
  { id: "TXN-003", type: "print", amount: -calcPrice(15,true,1), method: "Impression", date: "23/04/2025" },
  { id: "TXN-004", type: "charge", amount: 20.00, method: "Kiosque", date: "18/04/2025" },
  { id: "TXN-005", type: "print", amount: -calcPrice(2,false,3), method: "Impression", date: "17/04/2025" },
];

// ─── QR CODE SVG (simple visual) ─────────────────────────────────────────────
const QRCodeSVG = ({ value, size = 120 }) => {
  // Simple deterministic pattern based on value string
  const cells = 10;
  const cellSize = size / cells;
  const hash = value.split("").reduce((acc, c) => acc + c.charCodeAt(0), 0);
  const grid = Array.from({ length: cells }, (_, row) =>
    Array.from({ length: cells }, (_, col) => {
      if (row < 3 && col < 3) return true;
      if (row < 3 && col >= cells - 3) return true;
      if (row >= cells - 3 && col < 3) return true;
      return (hash * (row + 1) * (col + 1)) % 3 === 0;
    })
  );
  return (
    <svg width={size} height={size} viewBox={`0 0 ${size} ${size}`}>
      <rect width={size} height={size} fill="white" rx="4" />
      {grid.map((row, r) =>
        row.map((filled, c) =>
          filled ? <rect key={`${r}-${c}`} x={c * cellSize + 1} y={r * cellSize + 1} width={cellSize - 1} height={cellSize - 1} fill="#0A1628" /> : null
        )
      )}
    </svg>
  );
};

// ─── ICONS ────────────────────────────────────────────────────────────────────
const Icon = ({ name, size = 20, color = "currentColor" }) => {
  const icons = {
    printer: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect x="6" y="14" width="12" height="8"/></svg>,
    upload: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>,
    wallet: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M20 12V22H4a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v2"/><path d="M20 12a2 2 0 0 0-2-2H4"/><circle cx="16" cy="12" r="2"/></svg>,
    history: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>,
    dashboard: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>,
    file: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M13 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"/><polyline points="13 2 13 9 20 9"/></svg>,
    check: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round"><polyline points="20 6 9 17 4 12"/></svg>,
    qr: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="3" y="3" width="5" height="5"/><rect x="16" y="3" width="5" height="5"/><rect x="3" y="16" width="5" height="5"/><path d="M21 16h-3a2 2 0 0 0-2 2v3"/><line x1="21" y1="21" x2="21" y2="21"/><path d="M3 11h3a2 2 0 0 0 2-2V3"/><line x1="11" y1="3" x2="11" y2="5"/><path d="M11 11h5a2 2 0 0 1 2 2v1"/><line x1="11" y1="16" x2="11" y2="21"/><line x1="16" y1="21" x2="21" y2="21"/></svg>,
    card: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="1" y="4" width="22" height="16" rx="2" ry="2"/><line x1="1" y1="10" x2="23" y2="10"/></svg>,
    logout: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>,
    plus: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2.5" strokeLinecap="round"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>,
    arrow: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>,
    ticket: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M2 9a3 3 0 0 1 0 6v2a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-2a3 3 0 0 1 0-6V7a2 2 0 0 0-2-2H4a2 2 0 0 0-2 2z"/><line x1="12" y1="5" x2="12" y2="19" strokeDasharray="2 3"/></svg>,
    copy: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>,
    desk: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect x="2" y="7" width="20" height="14" rx="2"/><path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"/></svg>,
  };
  return icons[name] || null;
};

// ─── PRINT TICKET MODAL ───────────────────────────────────────────────────────
const PrintTicket = ({ job, onClose }) => {
  const [copied, setCopied] = useState(false);
  const handleCopy = () => {
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <div style={{
      position: "fixed", inset: 0, zIndex: 1000,
      background: "rgba(0,0,0,0.7)", backdropFilter: "blur(8px)",
      display: "flex", alignItems: "center", justifyContent: "center",
      padding: 20,
    }}>
      <div style={{
        background: COLORS.navyLight,
        border: "1px solid rgba(255,255,255,0.1)",
        borderRadius: 24, width: "100%", maxWidth: 420,
        overflow: "hidden",
        boxShadow: "0 40px 80px rgba(0,0,0,0.6)",
        animation: "fadeIn 0.3s ease",
      }}>
        {/* Header */}
        <div style={{
          background: `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.cyan}33)`,
          padding: "20px 24px",
          borderBottom: "1px solid rgba(255,255,255,0.08)",
          display: "flex", alignItems: "center", justifyContent: "space-between",
        }}>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <Icon name="ticket" size={20} color={COLORS.cyanLight} />
            <span style={{ color: COLORS.white, fontWeight: 700, fontSize: 16 }}>Ticket d'impression</span>
          </div>
          <button onClick={onClose} style={{ background: "rgba(255,255,255,0.1)", border: "none", borderRadius: 8, color: COLORS.gray, width: 32, height: 32, cursor: "pointer", fontSize: 18, display: "flex", alignItems: "center", justifyContent: "center" }}>×</button>
        </div>

        <div style={{ padding: "24px" }}>
          {/* QR Code */}
          <div style={{ textAlign: "center", marginBottom: 24 }}>
            <div style={{
              display: "inline-block", padding: 12,
              background: "white", borderRadius: 16,
              boxShadow: `0 0 30px ${COLORS.cyan}33`,
            }}>
              <QRCodeSVG value={job.ticket} size={140} />
            </div>
            <p style={{ color: COLORS.gray, fontSize: 12, marginTop: 10 }}>
              Montrez ce QR code à la bibliothèque
            </p>
          </div>

          {/* Ticket number */}
          <div style={{
            background: `${COLORS.blue}22`,
            border: `1px solid ${COLORS.blue}44`,
            borderRadius: 14, padding: "16px 20px", marginBottom: 16,
            display: "flex", alignItems: "center", justifyContent: "space-between",
          }}>
            <div>
              <div style={{ color: COLORS.gray, fontSize: 12, marginBottom: 4 }}>Numéro de ticket</div>
              <div style={{ color: COLORS.cyanLight, fontSize: 22, fontWeight: 800, letterSpacing: 2 }}>{job.ticket}</div>
            </div>
            <button onClick={handleCopy} style={{
              background: copied ? `${COLORS.success}33` : "rgba(255,255,255,0.06)",
              border: `1px solid ${copied ? COLORS.success : "rgba(255,255,255,0.1)"}`,
              borderRadius: 10, padding: "8px 14px", cursor: "pointer",
              display: "flex", alignItems: "center", gap: 6,
              color: copied ? COLORS.success : COLORS.gray,
              fontSize: 13, fontFamily: "'Sora', sans-serif",
              transition: "all 0.2s",
            }}>
              <Icon name={copied ? "check" : "copy"} size={14} color={copied ? COLORS.success : COLORS.gray} />
              {copied ? "Copié!" : "Copier"}
            </button>
          </div>

          {/* Bureau info */}
          <div style={{
            background: `${COLORS.gold}15`,
            border: `1px solid ${COLORS.gold}33`,
            borderRadius: 14, padding: "14px 18px", marginBottom: 16,
            display: "flex", alignItems: "center", gap: 12,
          }}>
            <div style={{ width: 38, height: 38, borderRadius: 10, background: `${COLORS.gold}22`, display: "flex", alignItems: "center", justifyContent: "center" }}>
              <Icon name="desk" size={18} color={COLORS.gold} />
            </div>
            <div>
              <div style={{ color: COLORS.gray, fontSize: 12 }}>Bureau d'impression assigné</div>
              <div style={{ color: COLORS.gold, fontSize: 16, fontWeight: 700 }}>{job.desk} — CMC Imprimerie</div>
            </div>
          </div>

          {/* Job details */}
          <div style={{
            background: "rgba(255,255,255,0.03)",
            border: "1px solid rgba(255,255,255,0.06)",
            borderRadius: 14, padding: "16px 18px",
            marginBottom: 20,
          }}>
            <div style={{ color: COLORS.gray, fontSize: 12, marginBottom: 12, textTransform: "uppercase", letterSpacing: "0.5px", fontWeight: 600 }}>Détails de la commande</div>
            {[
              { label: "Fichier", value: job.file },
              { label: "Pages", value: `${job.pages} pages` },
              { label: "Copies", value: `${job.copies} copies` },
              { label: "Mode", value: job.color ? "Couleur" : "Noir & Blanc" },
              { label: "Heure de retrait", value: job.time },
              { label: "Date", value: job.date },
              { label: "Montant débité", value: `${job.cost.toFixed(2)} MAD` },
            ].map((row, i) => (
              <div key={i} style={{
                display: "flex", justifyContent: "space-between", alignItems: "center",
                padding: "7px 0",
                borderBottom: i < 6 ? "1px solid rgba(255,255,255,0.04)" : "none",
              }}>
                <span style={{ color: COLORS.gray, fontSize: 13 }}>{row.label}</span>
                <span style={{ color: row.label === "Montant débité" ? COLORS.danger : COLORS.white, fontSize: 13, fontWeight: row.label === "Montant débité" ? 700 : 500 }}>{row.value}</span>
              </div>
            ))}
          </div>

          {/* Instructions */}
          <div style={{
            background: `${COLORS.success}10`,
            border: `1px solid ${COLORS.success}30`,
            borderRadius: 12, padding: "14px 16px",
          }}>
            <div style={{ color: COLORS.success, fontSize: 13, fontWeight: 600, marginBottom: 6 }}>📋 Instructions pour la bibliothèque</div>
            <ol style={{ color: COLORS.gray, fontSize: 12, paddingLeft: 18, lineHeight: 1.8, margin: 0 }}>
              <li>Présentez ce ticket (QR code ou numéro) à l'agent</li>
              <li>L'agent scanne le QR ou entre le numéro de ticket</li>
              <li>Le système confirme automatiquement la commande</li>
              <li>Récupérez vos impressions à l'heure indiquée</li>
            </ol>
          </div>
        </div>
      </div>
    </div>
  );
};

// ─── LOGIN PAGE ───────────────────────────────────────────────────────────────
const LoginPage = ({ onLogin }) => {
  const [email, setEmail] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [step, setStep] = useState("email");
  const [code, setCode] = useState("");
  const [authMethod, setAuthMethod] = useState(2);

  const handleEmailSubmit = () => {
    if (!email.endsWith("@cmc.ac.ma")) {
      setError("Utilisez votre email universitaire CMC (@cmc.ac.ma)");
      return;
    }
    setLoading(true);
    setTimeout(() => { setLoading(false); setStep("code"); setError(""); }, 1500);
  };

  const handleCodeSubmit = () => {
    setLoading(true);
    setTimeout(() => { setLoading(false); onLogin(); }, 1200);
  };

  return (
    <div style={{
      minHeight: "100vh",
      background: `linear-gradient(135deg, ${COLORS.navy} 0%, #0d1f3c 50%, #091525 100%)`,
      display: "flex", alignItems: "center", justifyContent: "center",
      fontFamily: "'Sora', sans-serif", position: "relative", overflow: "hidden",
    }}>
      <div style={{ position: "absolute", inset: 0, overflow: "hidden", pointerEvents: "none" }}>
        <div style={{ position: "absolute", top: "-20%", right: "-10%", width: 600, height: 600, borderRadius: "50%", background: `radial-gradient(circle, ${COLORS.blue}22 0%, transparent 70%)` }} />
        <div style={{ position: "absolute", bottom: "-20%", left: "-10%", width: 500, height: 500, borderRadius: "50%", background: `radial-gradient(circle, ${COLORS.cyan}18 0%, transparent 70%)` }} />
      </div>
      <div style={{
        background: COLORS.card, backdropFilter: "blur(20px)",
        border: "1px solid rgba(255,255,255,0.08)", borderRadius: 24,
        padding: "48px 44px", width: "100%", maxWidth: 440,
        position: "relative", zIndex: 1, boxShadow: "0 40px 80px rgba(0,0,0,0.5)",
      }}>
        <div style={{ textAlign: "center", marginBottom: 36 }}>
          <div style={{ width: 68, height: 68, borderRadius: 18, background: `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.cyan})`, display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 16px", boxShadow: `0 8px 24px ${COLORS.blue}55` }}>
            <Icon name="printer" size={32} color="white" />
          </div>
          <h1 style={{ margin: 0, fontSize: 26, fontWeight: 700, color: COLORS.white, letterSpacing: "-0.5px" }}>CMC Print</h1>
          <p style={{ margin: "6px 0 0", color: COLORS.gray, fontSize: 14 }}>Système d'impression stagiaires</p>
        </div>

        {step === "email" ? (
          <>
            <div style={{ marginBottom: 12 }}>
              <label style={{ display: "block", color: COLORS.grayLight, fontSize: 13, marginBottom: 8, fontWeight: 500 }}>Email universitaire</label>
              <input type="email" placeholder="prenom.nom@cmc.ac.ma" value={email}
                onChange={e => { setEmail(e.target.value); setError(""); }}
                onKeyDown={e => e.key === "Enter" && handleEmailSubmit()}
                style={{ width: "100%", padding: "14px 16px", background: "rgba(255,255,255,0.05)", border: `1.5px solid ${error ? COLORS.danger : "rgba(255,255,255,0.1)"}`, borderRadius: 12, color: COLORS.white, fontSize: 15, outline: "none", boxSizing: "border-box", fontFamily: "'Sora', sans-serif" }}
              />
              {error && <p style={{ margin: "8px 0 0", color: COLORS.danger, fontSize: 13 }}>{error}</p>}
            </div>
            <div style={{ marginBottom: 24 }}>
              <label style={{ display: "block", color: COLORS.grayLight, fontSize: 13, marginBottom: 8, fontWeight: 500 }}>Méthode de vérification</label>
              <div style={{ display: "flex", gap: 10 }}>
                {["QR Code", "CIN", "Email OTP"].map((m, i) => (
                  <div key={i} onClick={() => setAuthMethod(i)} style={{ flex: 1, padding: "10px 8px", textAlign: "center", background: authMethod === i ? `${COLORS.blue}33` : "rgba(255,255,255,0.04)", border: `1.5px solid ${authMethod === i ? COLORS.blue : "rgba(255,255,255,0.08)"}`, borderRadius: 10, cursor: "pointer", fontSize: 12, color: authMethod === i ? COLORS.cyanLight : COLORS.gray, fontWeight: authMethod === i ? 600 : 400 }}>{m}</div>
                ))}
              </div>
            </div>
            <button onClick={handleEmailSubmit} disabled={loading} style={{ width: "100%", padding: "15px", background: loading ? COLORS.navyLight : `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.blueAccent})`, border: "none", borderRadius: 12, color: "white", fontSize: 15, fontWeight: 600, cursor: loading ? "not-allowed" : "pointer", fontFamily: "'Sora', sans-serif", display: "flex", alignItems: "center", justifyContent: "center", gap: 8, boxShadow: loading ? "none" : `0 4px 20px ${COLORS.blue}55` }}>
              {loading ? <div style={{ width: 20, height: 20, border: "2px solid rgba(255,255,255,0.3)", borderTopColor: "white", borderRadius: "50%", animation: "spin 0.8s linear infinite" }} /> : <><span>Envoyer le code</span><Icon name="arrow" size={16} /></>}
            </button>
          </>
        ) : (
          <>
            <p style={{ color: COLORS.gray, fontSize: 14, textAlign: "center", marginBottom: 24 }}>Code envoyé à <span style={{ color: COLORS.cyan }}>{email}</span></p>
            <input type="text" placeholder="• • • • • •" value={code} onChange={e => setCode(e.target.value)} maxLength={6}
              style={{ width: "100%", padding: "18px 16px", textAlign: "center", background: "rgba(255,255,255,0.05)", border: "1.5px solid rgba(255,255,255,0.12)", borderRadius: 12, color: COLORS.white, fontSize: 22, outline: "none", boxSizing: "border-box", letterSpacing: 12, fontFamily: "monospace", marginBottom: 20 }}
            />
            <button onClick={handleCodeSubmit} disabled={loading} style={{ width: "100%", padding: "15px", background: `linear-gradient(135deg, ${COLORS.success}cc, ${COLORS.cyan})`, border: "none", borderRadius: 12, color: "white", fontSize: 15, fontWeight: 600, cursor: "pointer", fontFamily: "'Sora', sans-serif", display: "flex", alignItems: "center", justifyContent: "center", gap: 8 }}>
              {loading ? <div style={{ width: 20, height: 20, border: "2px solid rgba(255,255,255,0.3)", borderTopColor: "white", borderRadius: "50%", animation: "spin 0.8s linear infinite" }} /> : <><Icon name="check" size={18} /><span>Connexion</span></>}
            </button>
            <button onClick={() => setStep("email")} style={{ width: "100%", marginTop: 12, padding: "12px", background: "transparent", border: "none", color: COLORS.gray, fontSize: 14, cursor: "pointer", fontFamily: "'Sora', sans-serif" }}>← Retour</button>
          </>
        )}
        <p style={{ textAlign: "center", marginTop: 24, color: COLORS.gray, fontSize: 12 }}>CMC — Centre des Métiers du Commerce · 2025</p>
      </div>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Sora:wght@300;400;500;600;700&display=swap');
        @keyframes spin { to { transform: rotate(360deg); } }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(12px); } to { opacity: 1; transform: translateY(0); } }
        * { box-sizing: border-box; margin: 0; padding: 0; }
      `}</style>
    </div>
  );
};

// ─── SIDEBAR ──────────────────────────────────────────────────────────────────
const Sidebar = ({ active, setActive, onLogout }) => {
  const items = [
    { id: "dashboard", label: "Tableau de bord", icon: "dashboard" },
    { id: "upload", label: "Envoyer PDF", icon: "upload" },
    { id: "jobs", label: "Mes impressions", icon: "printer" },
    { id: "wallet", label: "Mon solde", icon: "wallet" },
    { id: "history", label: "Historique", icon: "history" },
  ];
  return (
    <div style={{ width: 240, minHeight: "100vh", background: COLORS.navyLight, borderRight: "1px solid rgba(255,255,255,0.06)", display: "flex", flexDirection: "column", padding: "24px 0", position: "fixed", left: 0, top: 0 }}>
      <div style={{ padding: "0 20px 28px", borderBottom: "1px solid rgba(255,255,255,0.06)" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <div style={{ width: 40, height: 40, borderRadius: 10, background: `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.cyan})`, display: "flex", alignItems: "center", justifyContent: "center" }}>
            <Icon name="printer" size={20} color="white" />
          </div>
          <div>
            <div style={{ color: COLORS.white, fontWeight: 700, fontSize: 16 }}>CMC Print</div>
            <div style={{ color: COLORS.gray, fontSize: 11 }}>v2.0 · 2025</div>
          </div>
        </div>
      </div>
      <nav style={{ flex: 1, padding: "16px 12px" }}>
        {items.map(item => (
          <button key={item.id} onClick={() => setActive(item.id)} style={{ width: "100%", display: "flex", alignItems: "center", gap: 12, padding: "12px 14px", borderRadius: 10, border: "none", cursor: "pointer", background: active === item.id ? `linear-gradient(135deg, ${COLORS.blue}33, ${COLORS.cyan}18)` : "transparent", color: active === item.id ? COLORS.cyanLight : COLORS.gray, fontSize: 14, fontWeight: active === item.id ? 600 : 400, textAlign: "left", marginBottom: 4, borderLeft: active === item.id ? `3px solid ${COLORS.cyan}` : "3px solid transparent", fontFamily: "'Sora', sans-serif" }}>
            <Icon name={item.icon} size={18} color={active === item.id ? COLORS.cyan : COLORS.gray} />
            {item.label}
          </button>
        ))}
      </nav>
      <div style={{ padding: "16px 12px", borderTop: "1px solid rgba(255,255,255,0.06)" }}>
        <div style={{ padding: "12px 14px", borderRadius: 10, background: "rgba(255,255,255,0.03)", marginBottom: 8 }}>
          <div style={{ color: COLORS.white, fontSize: 13, fontWeight: 600, marginBottom: 2 }}>{mockUser.name.split(" ")[0]} {mockUser.name.split(" ")[1]?.[0]}.</div>
          <div style={{ color: COLORS.gray, fontSize: 11 }}>{mockUser.email}</div>
          <div style={{ marginTop: 8, display: "flex", alignItems: "center", justifyContent: "space-between" }}>
            <span style={{ color: COLORS.gray, fontSize: 11 }}>Solde</span>
            <span style={{ color: COLORS.success, fontSize: 14, fontWeight: 700 }}>{mockUser.balance.toFixed(2)} MAD</span>
          </div>
        </div>
        <button onClick={onLogout} style={{ width: "100%", display: "flex", alignItems: "center", justifyContent: "center", gap: 8, padding: "10px", borderRadius: 10, border: "1px solid rgba(255,255,255,0.06)", background: "transparent", color: COLORS.gray, cursor: "pointer", fontSize: 13, fontFamily: "'Sora', sans-serif" }}>
          <Icon name="logout" size={16} /><span>Déconnexion</span>
        </button>
      </div>
    </div>
  );
};

// ─── DASHBOARD ────────────────────────────────────────────────────────────────
const DashboardPage = ({ setActive }) => {
  const [selectedJob, setSelectedJob] = useState(null);
  const stats = [
    { label: "Solde disponible", value: `${mockUser.balance.toFixed(2)} MAD`, sub: "+50.00 MAD il y a 2j", color: COLORS.success, icon: "wallet" },
    { label: "Impressions totales", value: mockUser.totalPrints, sub: "Ce mois: 23 pages", color: COLORS.cyan, icon: "printer" },
    { label: "En attente", value: mockUser.pendingJobs, sub: "2 jobs pour demain", color: COLORS.gold, icon: "history" },
    { label: "Dépensé ce mois", value: "24.60 MAD", sub: "Moyenne: 5.2 MAD/j", color: COLORS.blueAccent, icon: "card" },
  ];
  return (
    <div style={{ animation: "fadeIn 0.4s ease" }}>
      {selectedJob && <PrintTicket job={selectedJob} onClose={() => setSelectedJob(null)} />}
      <div style={{ marginBottom: 32 }}>
        <h2 style={{ margin: 0, color: COLORS.white, fontSize: 24, fontWeight: 700 }}>Bonjour, {mockUser.name.split(" ")[0]} 👋</h2>
        <p style={{ margin: "6px 0 0", color: COLORS.gray, fontSize: 14 }}>Mercredi 23 avril 2025 · ID: {mockUser.id}</p>
      </div>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(2, 1fr)", gap: 16, marginBottom: 28 }}>
        {stats.map((s, i) => (
          <div key={i} style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 16, padding: "20px 22px" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 12 }}>
              <span style={{ color: COLORS.gray, fontSize: 13 }}>{s.label}</span>
              <div style={{ width: 36, height: 36, borderRadius: 10, background: `${s.color}22`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                <Icon name={s.icon} size={16} color={s.color} />
              </div>
            </div>
            <div style={{ color: s.color, fontSize: 26, fontWeight: 700 }}>{s.value}</div>
            <div style={{ color: COLORS.gray, fontSize: 12, marginTop: 4 }}>{s.sub}</div>
          </div>
        ))}
      </div>
      <div style={{ marginBottom: 28 }}>
        <h3 style={{ color: COLORS.grayLight, fontSize: 14, fontWeight: 600, marginBottom: 14, textTransform: "uppercase", letterSpacing: "0.5px" }}>Actions rapides</h3>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 12 }}>
          {[
            { label: "Envoyer PDF", icon: "upload", color: COLORS.blue, page: "upload" },
            { label: "Charger solde", icon: "plus", color: COLORS.success, page: "wallet" },
            { label: "Mes jobs", icon: "printer", color: COLORS.cyan, page: "jobs" },
          ].map((a, i) => (
            <button key={i} onClick={() => setActive(a.page)} style={{ padding: "18px 12px", borderRadius: 14, border: "none", background: `${a.color}18`, cursor: "pointer", textAlign: "center", fontFamily: "'Sora', sans-serif" }}>
              <div style={{ width: 42, height: 42, borderRadius: 12, background: `${a.color}28`, display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 10px" }}>
                <Icon name={a.icon} size={20} color={a.color} />
              </div>
              <div style={{ color: COLORS.grayLight, fontSize: 13, fontWeight: 500 }}>{a.label}</div>
            </button>
          ))}
        </div>
      </div>
      <div>
        <h3 style={{ color: COLORS.grayLight, fontSize: 14, fontWeight: 600, marginBottom: 14, textTransform: "uppercase", letterSpacing: "0.5px" }}>Dernières impressions</h3>
        {mockJobs.slice(0, 3).map((job, i) => (
          <div key={i} onClick={() => setSelectedJob(job)} style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 12, padding: "14px 18px", marginBottom: 10, display: "flex", alignItems: "center", justifyContent: "space-between", cursor: "pointer" }}>
            <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
              <div style={{ width: 36, height: 36, borderRadius: 8, background: job.status === "pending" ? `${COLORS.gold}22` : `${COLORS.success}22`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                <Icon name="file" size={16} color={job.status === "pending" ? COLORS.gold : COLORS.success} />
              </div>
              <div>
                <div style={{ color: COLORS.white, fontSize: 13, fontWeight: 500 }}>{job.file}</div>
                <div style={{ color: COLORS.gray, fontSize: 12 }}>{job.pages} pages · {job.date} à {job.time} · <span style={{ color: COLORS.cyan }}>{job.ticket}</span></div>
              </div>
            </div>
            <div style={{ textAlign: "right" }}>
              <div style={{ color: COLORS.danger, fontSize: 14, fontWeight: 600 }}>-{job.cost.toFixed(2)} MAD</div>
              <div style={{ fontSize: 11, marginTop: 2, padding: "2px 8px", borderRadius: 6, background: job.status === "pending" ? `${COLORS.gold}22` : `${COLORS.success}22`, color: job.status === "pending" ? COLORS.gold : COLORS.success }}>
                {job.status === "pending" ? "En attente" : "Terminé"}
              </div>
            </div>
          </div>
        ))}
        <p style={{ color: COLORS.gray, fontSize: 12, textAlign: "center", marginTop: 8 }}>Cliquez sur un job pour voir le ticket 🎫</p>
      </div>
    </div>
  );
};

// ─── UPLOAD PAGE ──────────────────────────────────────────────────────────────
const UploadPage = () => {
  const [file, setFile] = useState(null);
  const [step, setStep] = useState("upload");
  const [settings, setSettings] = useState({ copies: 1, color: false, recto: true, time: "08:30", pages: 12 });
  const [ticket, setTicket] = useState(null);

  const cost = calcPrice(settings.pages, settings.color, settings.copies);
  const priceLabel = getPriceLabel(settings.pages, settings.color);
  const desk = ["Bureau 1", "Bureau 2", "Bureau 3"][Math.floor(Math.random() * 3)];

  const handleConfirm = () => {
    const newTicket = {
      ticket: `TKT-${8800 + Math.floor(Math.random() * 99)}`,
      file: file?.name,
      pages: settings.pages,
      color: settings.color,
      copies: settings.copies,
      cost,
      time: settings.time,
      date: "Aujourd'hui",
      desk,
    };
    setTicket(newTicket);
    setStep("ticket");
  };

  if (step === "ticket" && ticket) return (
    <div style={{ animation: "fadeIn 0.4s ease" }}>
      <h2 style={{ margin: "0 0 6px", color: COLORS.white, fontSize: 22, fontWeight: 700 }}>Job confirmé! 🎉</h2>
      <p style={{ margin: "0 0 24px", color: COLORS.gray, fontSize: 14 }}>Voici votre ticket d'impression</p>

      {/* QR */}
      <div style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.08)", borderRadius: 20, padding: 24, textAlign: "center", marginBottom: 20 }}>
        <div style={{ display: "inline-block", padding: 14, background: "white", borderRadius: 16, boxShadow: `0 0 40px ${COLORS.cyan}44`, marginBottom: 16 }}>
          <QRCodeSVG value={ticket.ticket} size={160} />
        </div>
        <div style={{ color: COLORS.gray, fontSize: 12, marginBottom: 8 }}>Numéro de ticket</div>
        <div style={{ color: COLORS.cyanLight, fontSize: 28, fontWeight: 800, letterSpacing: 3 }}>{ticket.ticket}</div>
        <div style={{ marginTop: 8, color: COLORS.gray, fontSize: 13 }}>Montrez ce QR à l'agent de la bibliothèque</div>
      </div>

      {/* Bureau */}
      <div style={{ background: `${COLORS.gold}15`, border: `1px solid ${COLORS.gold}33`, borderRadius: 14, padding: "16px 20px", marginBottom: 20, display: "flex", alignItems: "center", gap: 14 }}>
        <div style={{ width: 44, height: 44, borderRadius: 12, background: `${COLORS.gold}22`, display: "flex", alignItems: "center", justifyContent: "center" }}>
          <Icon name="desk" size={22} color={COLORS.gold} />
        </div>
        <div>
          <div style={{ color: COLORS.gray, fontSize: 12 }}>Bureau assigné</div>
          <div style={{ color: COLORS.gold, fontSize: 18, fontWeight: 700 }}>{ticket.desk} — CMC Imprimerie</div>
          <div style={{ color: COLORS.gray, fontSize: 12 }}>Retrait prévu à {ticket.time}</div>
        </div>
      </div>

      {/* Details */}
      <div style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 14, padding: "18px 20px", marginBottom: 20 }}>
        <div style={{ color: COLORS.gray, fontSize: 12, marginBottom: 14, textTransform: "uppercase", letterSpacing: "0.5px", fontWeight: 600 }}>Récapitulatif</div>
        {[
          { label: "Fichier", value: ticket.file },
          { label: "Pages", value: `${ticket.pages} pages (${ticket.color ? "Couleur" : "N&B"})` },
          { label: "Copies", value: `${ticket.copies} copies` },
          { label: "Prix unitaire", value: priceLabel },
          { label: "Total débité", value: `${ticket.cost.toFixed(2)} MAD` },
        ].map((row, i) => (
          <div key={i} style={{ display: "flex", justifyContent: "space-between", padding: "8px 0", borderBottom: i < 4 ? "1px solid rgba(255,255,255,0.04)" : "none" }}>
            <span style={{ color: COLORS.gray, fontSize: 13 }}>{row.label}</span>
            <span style={{ color: row.label === "Total débité" ? COLORS.danger : COLORS.white, fontSize: 13, fontWeight: row.label === "Total débité" ? 700 : 500 }}>{row.value}</span>
          </div>
        ))}
      </div>

      {/* Instructions */}
      <div style={{ background: `${COLORS.success}10`, border: `1px solid ${COLORS.success}30`, borderRadius: 12, padding: "14px 18px", marginBottom: 20 }}>
        <div style={{ color: COLORS.success, fontSize: 13, fontWeight: 600, marginBottom: 8 }}>📋 Instructions pour la bibliothèque</div>
        <ol style={{ color: COLORS.gray, fontSize: 12, paddingLeft: 18, lineHeight: 2, margin: 0 }}>
          <li>Présentez le QR code ou le numéro <strong style={{ color: COLORS.cyanLight }}>{ticket.ticket}</strong> à l'agent</li>
          <li>L'agent scanne et confirme la commande</li>
          <li>Récupérez vos impressions à <strong style={{ color: COLORS.white }}>{ticket.time}</strong></li>
        </ol>
      </div>

      <button onClick={() => { setStep("upload"); setFile(null); setTicket(null); }} style={{ width: "100%", padding: "14px", background: `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.blueAccent})`, border: "none", borderRadius: 12, color: "white", fontSize: 15, fontWeight: 600, cursor: "pointer", fontFamily: "'Sora', sans-serif" }}>
        Envoyer un autre PDF
      </button>
    </div>
  );

  return (
    <div style={{ animation: "fadeIn 0.4s ease" }}>
      <h2 style={{ margin: "0 0 6px", color: COLORS.white, fontSize: 22, fontWeight: 700 }}>Envoyer un PDF</h2>
      <p style={{ margin: "0 0 28px", color: COLORS.gray, fontSize: 14 }}>Planifiez votre impression à l'avance</p>

      {/* Pricing info */}
      <div style={{ background: `${COLORS.cyan}10`, border: `1px solid ${COLORS.cyan}25`, borderRadius: 12, padding: "12px 16px", marginBottom: 24, display: "flex", gap: 20, flexWrap: "wrap" }}>
        <div style={{ fontSize: 13 }}><span style={{ color: COLORS.gray }}>N&B (−20p): </span><span style={{ color: COLORS.white, fontWeight: 600 }}>1.00 MAD/page</span></div>
        <div style={{ fontSize: 13 }}><span style={{ color: COLORS.gray }}>N&B (+20p): </span><span style={{ color: COLORS.white, fontWeight: 600 }}>0.50 MAD/page</span></div>
        <div style={{ fontSize: 13 }}><span style={{ color: COLORS.gray }}>Couleur: </span><span style={{ color: COLORS.gold, fontWeight: 600 }}>×2 du prix N&B</span></div>
      </div>

      {step === "upload" ? (
        <div onClick={() => { setFile({ name: "Mon_Document.pdf" }); setStep("settings"); }} style={{ border: "2px dashed rgba(255,255,255,0.12)", borderRadius: 20, padding: "60px 40px", textAlign: "center", background: "rgba(255,255,255,0.02)", cursor: "pointer" }}>
          <div style={{ width: 64, height: 64, borderRadius: 16, background: `${COLORS.blue}28`, display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 16px" }}>
            <Icon name="upload" size={28} color={COLORS.cyan} />
          </div>
          <p style={{ color: COLORS.white, fontSize: 16, fontWeight: 600, margin: "0 0 8px" }}>Glissez votre PDF ici</p>
          <p style={{ color: COLORS.gray, fontSize: 14, margin: 0 }}>ou cliquez pour sélectionner</p>
          <p style={{ color: COLORS.gray, fontSize: 12, marginTop: 12 }}>PDF uniquement · Max 50MB</p>
        </div>
      ) : (
        <div>
          <div style={{ background: `${COLORS.success}12`, border: `1px solid ${COLORS.success}33`, borderRadius: 12, padding: "14px 18px", marginBottom: 24, display: "flex", alignItems: "center", gap: 12 }}>
            <Icon name="check" size={18} color={COLORS.success} />
            <span style={{ color: COLORS.white, fontSize: 14, fontWeight: 500 }}>{file?.name}</span>
            <span style={{ color: COLORS.gray, fontSize: 13, marginLeft: "auto" }}>{settings.pages} pages détectées</span>
          </div>

          <div style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 16, padding: "22px", marginBottom: 20 }}>
            <h3 style={{ margin: "0 0 18px", color: COLORS.grayLight, fontSize: 14, textTransform: "uppercase", letterSpacing: "0.5px", fontWeight: 600 }}>Paramètres</h3>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16 }}>
              <div>
                <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 8 }}>Copies</label>
                <div style={{ display: "flex", gap: 8 }}>
                  {[1, 2, 3, 5].map(n => (
                    <button key={n} onClick={() => setSettings({...settings, copies: n})} style={{ flex: 1, padding: "10px 0", borderRadius: 8, border: "none", cursor: "pointer", background: settings.copies === n ? COLORS.blue : "rgba(255,255,255,0.05)", color: settings.copies === n ? "white" : COLORS.gray, fontFamily: "'Sora', sans-serif", fontSize: 14, fontWeight: 600 }}>{n}</button>
                  ))}
                </div>
              </div>
              <div>
                <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 8 }}>Heure de retrait</label>
                <select value={settings.time} onChange={e => setSettings({...settings, time: e.target.value})} style={{ width: "100%", padding: "10px 12px", borderRadius: 8, background: "rgba(255,255,255,0.05)", border: "1px solid rgba(255,255,255,0.1)", color: COLORS.white, fontSize: 14, fontFamily: "'Sora', sans-serif", outline: "none" }}>
                  {["07:30","08:00","08:30","09:00","10:00","12:00","14:00"].map(t => <option key={t} value={t} style={{ background: COLORS.navyLight }}>{t}</option>)}
                </select>
              </div>
              <div>
                <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 8 }}>Mode</label>
                <div style={{ display: "flex", gap: 8 }}>
                  {[{ label: "N&B", val: false }, { label: "Couleur", val: true }].map(m => (
                    <button key={m.label} onClick={() => setSettings({...settings, color: m.val})} style={{ flex: 1, padding: "10px", borderRadius: 8, border: "none", cursor: "pointer", background: settings.color === m.val ? COLORS.blue : "rgba(255,255,255,0.05)", color: settings.color === m.val ? "white" : COLORS.gray, fontFamily: "'Sora', sans-serif", fontSize: 13 }}>{m.label}</button>
                  ))}
                </div>
              </div>
              <div>
                <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 8 }}>Impression</label>
                <div style={{ display: "flex", gap: 8 }}>
                  {[{ label: "Recto", val: true }, { label: "R/V", val: false }].map(m => (
                    <button key={m.label} onClick={() => setSettings({...settings, recto: m.val})} style={{ flex: 1, padding: "10px", borderRadius: 8, border: "none", cursor: "pointer", background: settings.recto === m.val ? COLORS.blue : "rgba(255,255,255,0.05)", color: settings.recto === m.val ? "white" : COLORS.gray, fontFamily: "'Sora', sans-serif", fontSize: 13 }}>{m.label}</button>
                  ))}
                </div>
              </div>
            </div>
          </div>

          {/* Cost */}
          <div style={{ background: `${COLORS.blue}18`, border: `1px solid ${COLORS.blue}33`, borderRadius: 14, padding: "18px 22px", marginBottom: 20, display: "flex", alignItems: "center", justifyContent: "space-between" }}>
            <div>
              <div style={{ color: COLORS.gray, fontSize: 13 }}>Coût estimé</div>
              <div style={{ color: COLORS.white, fontSize: 28, fontWeight: 700 }}>{cost.toFixed(2)} MAD</div>
              <div style={{ color: COLORS.gray, fontSize: 12 }}>Solde restant: {(mockUser.balance - cost).toFixed(2)} MAD</div>
            </div>
            <div style={{ textAlign: "right", color: COLORS.gray, fontSize: 13, lineHeight: 1.8 }}>
              <div>{settings.pages} pages × {settings.copies} copies</div>
              <div style={{ color: settings.color ? COLORS.gold : COLORS.cyanLight, fontWeight: 600 }}>{priceLabel}</div>
              <div>Retrait à {settings.time}</div>
            </div>
          </div>

          <div style={{ display: "flex", gap: 12 }}>
            <button onClick={() => { setStep("upload"); setFile(null); }} style={{ padding: "14px 20px", borderRadius: 12, border: "1px solid rgba(255,255,255,0.1)", background: "transparent", color: COLORS.gray, cursor: "pointer", fontFamily: "'Sora', sans-serif", fontSize: 14 }}>Annuler</button>
            <button onClick={handleConfirm} style={{ flex: 1, padding: "14px", background: `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.blueAccent})`, border: "none", borderRadius: 12, color: "white", fontSize: 15, fontWeight: 600, cursor: "pointer", fontFamily: "'Sora', sans-serif", boxShadow: `0 4px 20px ${COLORS.blue}44` }}>
              Confirmer → Obtenir le ticket
            </button>
          </div>
        </div>
      )}
    </div>
  );
};

// ─── JOBS PAGE ────────────────────────────────────────────────────────────────
const JobsPage = () => {
  const [selectedJob, setSelectedJob] = useState(null);
  return (
    <div style={{ animation: "fadeIn 0.4s ease" }}>
      {selectedJob && <PrintTicket job={selectedJob} onClose={() => setSelectedJob(null)} />}
      <h2 style={{ margin: "0 0 6px", color: COLORS.white, fontSize: 22, fontWeight: 700 }}>Mes Impressions</h2>
      <p style={{ margin: "0 0 28px", color: COLORS.gray, fontSize: 14 }}>Cliquez sur un job pour voir le ticket 🎫</p>
      {mockJobs.map((job, i) => (
        <div key={i} onClick={() => setSelectedJob(job)} style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 14, padding: "18px 20px", marginBottom: 12, cursor: "pointer", transition: "border-color 0.15s" }}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 12 }}>
            <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
              <div style={{ width: 40, height: 40, borderRadius: 10, background: job.status === "pending" ? `${COLORS.gold}22` : `${COLORS.success}22`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                <Icon name="file" size={18} color={job.status === "pending" ? COLORS.gold : COLORS.success} />
              </div>
              <div>
                <div style={{ color: COLORS.white, fontSize: 14, fontWeight: 500 }}>{job.file}</div>
                <div style={{ color: COLORS.cyan, fontSize: 12, fontWeight: 600 }}>{job.ticket} · {job.desk}</div>
              </div>
            </div>
            <div style={{ padding: "4px 12px", borderRadius: 8, fontSize: 12, fontWeight: 600, background: job.status === "pending" ? `${COLORS.gold}22` : `${COLORS.success}22`, color: job.status === "pending" ? COLORS.gold : COLORS.success }}>
              {job.status === "pending" ? "⏳ En attente" : "✓ Terminé"}
            </div>
          </div>
          <div style={{ display: "flex", gap: 16, fontSize: 13, color: COLORS.gray, flexWrap: "wrap" }}>
            <span>📄 {job.pages}p</span>
            <span>📋 {job.copies} copies</span>
            <span>🎨 {job.color ? "Couleur" : "N&B"}</span>
            <span>🕒 {job.date} · {job.time}</span>
            <span style={{ marginLeft: "auto", color: COLORS.danger, fontWeight: 600 }}>-{job.cost.toFixed(2)} MAD</span>
          </div>
        </div>
      ))}
    </div>
  );
};

// ─── WALLET PAGE ──────────────────────────────────────────────────────────────
const WalletPage = () => {
  const [amount, setAmount] = useState(50);
  const [method, setMethod] = useState("card");
  const [success, setSuccess] = useState(false);
  return (
    <div style={{ animation: "fadeIn 0.4s ease" }}>
      <h2 style={{ margin: "0 0 6px", color: COLORS.white, fontSize: 22, fontWeight: 700 }}>Mon Solde</h2>
      <p style={{ margin: "0 0 28px", color: COLORS.gray, fontSize: 14 }}>Rechargez votre compte d'impression</p>
      <div style={{ background: `linear-gradient(135deg, ${COLORS.blue}, #0d2850)`, borderRadius: 20, padding: "28px", marginBottom: 24, position: "relative", overflow: "hidden", boxShadow: `0 20px 50px ${COLORS.blue}33` }}>
        <div style={{ position: "absolute", top: -30, right: -30, width: 150, height: 150, borderRadius: "50%", background: "rgba(255,255,255,0.04)" }} />
        <div style={{ color: "rgba(255,255,255,0.7)", fontSize: 13, marginBottom: 8 }}>Solde disponible</div>
        <div style={{ color: "white", fontSize: 42, fontWeight: 700, letterSpacing: "-1px" }}>{mockUser.balance.toFixed(2)} <span style={{ fontSize: 20, fontWeight: 400, opacity: 0.7 }}>MAD</span></div>
        <div style={{ marginTop: 16, fontSize: 12, color: "rgba(255,255,255,0.6)" }}>ID: {mockUser.id} · Mis à jour: aujourd'hui</div>
      </div>
      {success ? (
        <div style={{ textAlign: "center", padding: 40 }}>
          <div style={{ width: 70, height: 70, borderRadius: "50%", background: `${COLORS.success}22`, border: `2px solid ${COLORS.success}`, display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 16px" }}>
            <Icon name="check" size={32} color={COLORS.success} />
          </div>
          <h3 style={{ color: COLORS.white, margin: "0 0 8px" }}>Rechargement réussi!</h3>
          <p style={{ color: COLORS.gray }}>+{amount.toFixed(2)} MAD ajoutés</p>
          <button onClick={() => setSuccess(false)} style={{ padding: "12px 24px", marginTop: 16, background: `${COLORS.blue}33`, border: `1px solid ${COLORS.blue}44`, borderRadius: 10, color: COLORS.cyanLight, fontSize: 14, cursor: "pointer", fontFamily: "'Sora', sans-serif" }}>Nouveau rechargement</button>
        </div>
      ) : (
        <div style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 16, padding: "24px" }}>
          <h3 style={{ margin: "0 0 18px", color: COLORS.grayLight, fontSize: 14, textTransform: "uppercase", letterSpacing: "0.5px", fontWeight: 600 }}>Recharger le solde</h3>
          <div style={{ marginBottom: 20 }}>
            <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 10 }}>Montant (MAD)</label>
            <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 10 }}>
              {[20, 50, 100, 200].map(a => (
                <button key={a} onClick={() => setAmount(a)} style={{ padding: "12px", borderRadius: 10, border: "none", cursor: "pointer", background: amount === a ? `linear-gradient(135deg, ${COLORS.blue}, ${COLORS.blueAccent})` : "rgba(255,255,255,0.05)", color: amount === a ? "white" : COLORS.gray, fontFamily: "'Sora', sans-serif", fontSize: 15, fontWeight: 600 }}>{a}</button>
              ))}
            </div>
          </div>
          <div style={{ marginBottom: 24 }}>
            <label style={{ color: COLORS.gray, fontSize: 13, display: "block", marginBottom: 10 }}>Méthode de paiement</label>
            {[
              { id: "card", label: "Carte bancaire", sub: "Visa / Mastercard", icon: "card" },
              { id: "qr", label: "QR Code", sub: "Via l'application CMC", icon: "qr" },
              { id: "kiosk", label: "Kiosque", sub: "Paiement espèces en agence", icon: "wallet" },
            ].map(m => (
              <div key={m.id} onClick={() => setMethod(m.id)} style={{ display: "flex", alignItems: "center", gap: 14, padding: "14px 16px", borderRadius: 12, marginBottom: 8, cursor: "pointer", background: method === m.id ? `${COLORS.blue}18` : "rgba(255,255,255,0.03)", border: `1.5px solid ${method === m.id ? COLORS.blue : "rgba(255,255,255,0.06)"}` }}>
                <div style={{ width: 38, height: 38, borderRadius: 9, background: method === m.id ? `${COLORS.blue}33` : "rgba(255,255,255,0.06)", display: "flex", alignItems: "center", justifyContent: "center" }}>
                  <Icon name={m.icon} size={18} color={method === m.id ? COLORS.cyan : COLORS.gray} />
                </div>
                <div>
                  <div style={{ color: COLORS.white, fontSize: 14, fontWeight: 500 }}>{m.label}</div>
                  <div style={{ color: COLORS.gray, fontSize: 12 }}>{m.sub}</div>
                </div>
                {method === m.id && <div style={{ marginLeft: "auto" }}><Icon name="check" size={18} color={COLORS.cyan} /></div>}
              </div>
            ))}
          </div>
          <button onClick={() => setSuccess(true)} style={{ width: "100%", padding: "15px", background: `linear-gradient(135deg, ${COLORS.success}cc, ${COLORS.cyan})`, border: "none", borderRadius: 12, color: "white", fontSize: 15, fontWeight: 600, cursor: "pointer", fontFamily: "'Sora', sans-serif" }}>
            Recharger {amount} MAD
          </button>
        </div>
      )}
    </div>
  );
};

// ─── HISTORY PAGE ─────────────────────────────────────────────────────────────
const HistoryPage = () => (
  <div style={{ animation: "fadeIn 0.4s ease" }}>
    <h2 style={{ margin: "0 0 6px", color: COLORS.white, fontSize: 22, fontWeight: 700 }}>Historique</h2>
    <p style={{ margin: "0 0 28px", color: COLORS.gray, fontSize: 14 }}>Toutes vos transactions</p>
    {mockTransactions.map((t, i) => (
      <div key={i} style={{ background: COLORS.card, border: "1px solid rgba(255,255,255,0.06)", borderRadius: 12, padding: "16px 20px", marginBottom: 10, display: "flex", alignItems: "center", justifyContent: "space-between" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 14 }}>
          <div style={{ width: 40, height: 40, borderRadius: 10, background: t.type === "charge" ? `${COLORS.success}22` : `${COLORS.danger}18`, display: "flex", alignItems: "center", justifyContent: "center" }}>
            <Icon name={t.type === "charge" ? "plus" : "printer"} size={18} color={t.type === "charge" ? COLORS.success : COLORS.danger} />
          </div>
          <div>
            <div style={{ color: COLORS.white, fontSize: 14, fontWeight: 500 }}>{t.method}</div>
            <div style={{ color: COLORS.gray, fontSize: 12 }}>{t.date} · {t.id}</div>
          </div>
        </div>
        <div style={{ color: t.amount > 0 ? COLORS.success : COLORS.danger, fontSize: 16, fontWeight: 700 }}>
          {t.amount > 0 ? "+" : ""}{t.amount.toFixed(2)} MAD
        </div>
      </div>
    ))}
  </div>
);

// ─── MAIN APP ─────────────────────────────────────────────────────────────────
export default function App() {
  const [loggedIn, setLoggedIn] = useState(false);
  const [active, setActive] = useState("dashboard");
  const pages = {
    dashboard: <DashboardPage setActive={setActive} />,
    upload: <UploadPage />,
    jobs: <JobsPage />,
    wallet: <WalletPage />,
    history: <HistoryPage />,
  };
  if (!loggedIn) return <LoginPage onLogin={() => setLoggedIn(true)} />;
  return (
    <div style={{ minHeight: "100vh", background: COLORS.navy, fontFamily: "'Sora', sans-serif", display: "flex" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Sora:wght@300;400;500;600;700&display=swap');
        @keyframes spin { to { transform: rotate(360deg); } }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(12px); } to { opacity: 1; transform: translateY(0); } }
        * { box-sizing: border-box; margin: 0; padding: 0; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.12); border-radius: 4px; }
      `}</style>
      <Sidebar active={active} setActive={setActive} onLogout={() => setLoggedIn(false)} />
      <main style={{ marginLeft: 240, flex: 1, padding: "36px 40px", maxWidth: 800 }}>
        {pages[active]}
      </main>
    </div>
  );
}
