# hortal---appimport { useState, useEffect, useRef } from "react";
import {
  Zap, Droplet, Battery, Box, Wind, Shield, Wrench, Tag,
  Circle, Package, Pen, Shirt, Flame, Layers, CheckCircle2,
  Clock, User, Plus, X, ChevronDown, AlertTriangle, Cpu,
  Trash2, Send, Activity, Hash, RotateCcw
} from "lucide-react";

const ITEMS = [
  { id: "izolirband", label: "איזולירבנד", icon: Layers, category: "חשמל" },
  { id: "teflon", label: "טפלון", icon: Wind, category: "חשמל" },
  { id: "azikun_small", label: "אזיקונים קטנים", icon: Hash, category: "חיבורים" },
  { id: "azikun_med", label: "אזיקונים בינוניים", icon: Hash, category: "חיבורים" },
  { id: "azikun_large", label: "אזיקונים גדולים", icon: Hash, category: "חיבורים" },
  { id: "super_glue", label: "דבק מהיר", icon: Droplet, category: "דבקים" },
  { id: "silicon_rtv", label: "סיליקון RTV", icon: Droplet, category: "דבקים" },
  { id: "glove_m", label: "כפפות M", icon: Shirt, category: "ציוד מגן" },
  { id: "glove_l", label: "כפפות L", icon: Shirt, category: "ציוד מגן" },
  { id: "glove_xl", label: "כפפות XL", icon: Shirt, category: "ציוד מגן" },
  { id: "battery_mrj", label: "סוללות MRJ", icon: Battery, category: "חשמל" },
  { id: "battery_mds", label: "סוללות MDS", icon: Battery, category: "חשמל" },
  { id: "solder_gas", label: "גז למלחם", icon: Flame, category: "לחמה" },
  { id: "shrink_small", label: "שרוול מתכווץ קטן", icon: Zap, category: "חשמל" },
  { id: "shrink_med", label: "שרוול מתכווץ בינוני", icon: Zap, category: "חשמל" },
  { id: "shrink_large", label: "שרוול מתכווץ גדול", icon: Zap, category: "חשמל" },
  { id: "tin", label: "בדיל", icon: Circle, category: "לחמה" },
  { id: "brake_spray", label: "ספריי ברקסים", icon: Wind, category: "ספריי" },
  { id: "lube_spray", label: "ספריי שימון", icon: Wind, category: "ספריי" },
  { id: "contact_spray", label: "ספריי ניקוי מגעים", icon: Wind, category: "ספריי" },
  { id: "chem_glue", label: "דבק כימי להתקנות", icon: Droplet, category: "דבקים" },
  { id: "nuts_washers", label: "אומים ושייבות", icon: Wrench, category: "מכני" },
  { id: "threaded_rods", label: "מוטות הברגה", icon: Wrench, category: "מכני" },
  { id: "marker", label: "טוש סימון", icon: Pen, category: "כלים" },
  { id: "rags", label: "סמרטוטים", icon: Package, category: "כלים" },
  { id: "oring_p12", label: "O-Ring P12", icon: Circle, category: "אטמים" },
  { id: "oring_p125", label: "O-Ring P12.5", icon: Circle, category: "אטמים" },
  { id: "oring_p15", label: "O-Ring P15", icon: Circle, category: "אטמים" },
  { id: "oring_p10a", label: "O-Ring P10A", icon: Circle, category: "אטמים" },
  { id: "grease_bearings", label: "גריז מיסבים", icon: Droplet, category: "גריז" },
  { id: "grease_springs", label: "גריז קפיצים", icon: Droplet, category: "גריז" },
  { id: "grease_seals", label: "גריז אטמים", icon: Droplet, category: "גריז" },
];

const CATEGORIES = [...new Set(ITEMS.map(i => i.category))];

const STATUS_CONFIG = {
  pending: { label: "ממתין", color: "#f59e0b", bg: "rgba(245,158,11,0.15)" },
  brought: { label: "הובא ✓", color: "#22c55e", bg: "rgba(34,197,94,0.15)" },
};

function generateId() {
  return Math.random().toString(36).substr(2, 9);
}

function timeAgo(ts) {
  const diff = Math.floor((Date.now() - ts) / 1000);
  if (diff < 60) return `לפני ${diff} שניות`;
  if (diff < 3600) return `לפני ${Math.floor(diff / 60)} דקות`;
  return `לפני ${Math.floor(diff / 3600)} שעות`;
}

export default function App() {
  const [screen, setScreen] = useState("login");
  const [techName, setTechName] = useState("");
  const [requests, setRequests] = useState([]);
  const [showNewRequest, setShowNewRequest] = useState(false);
  const [selectedItems, setSelectedItems] = useState({});
  const [otherText, setOtherText] = useState("");
  const [activeCategory, setActiveCategory] = useState("הכל");
  const [ticker, setTicker] = useState(0);
  const nameRef = useRef();

  useEffect(() => {
    const interval = setInterval(() => setTicker(t => t + 1), 30000);
    return () => clearInterval(interval);
  }, []);

  const handleLogin = () => {
    if (techName.trim().length < 2) return;
    setScreen("dashboard");
  };

  const toggleItem = (id) => {
    setSelectedItems(prev => {
      const next = { ...prev };
      if (next[id]) delete next[id];
      else next[id] = 1;
      return next;
    });
  };

  const updateQty = (id, delta) => {
    setSelectedItems(prev => {
      const current = prev[id] || 1;
      const next = Math.max(1, current + delta);
      return { ...prev, [id]: next };
    });
  };

  const submitRequest = () => {
    const items = Object.entries(selectedItems).map(([id, qty]) => {
      const item = ITEMS.find(i => i.id === id);
      return { id, label: item.label, qty };
    });
    if (otherText.trim()) items.push({ id: "other", label: otherText.trim(), qty: 1 });
    if (!items.length) return;

    const req = {
      id: generateId(),
      techName,
      items,
      status: "pending",
      createdAt: Date.now(),
    };
    setRequests(prev => [req, ...prev]);
    setSelectedItems({});
    setOtherText("");
    setShowNewRequest(false);
  };

  const markDone = (reqId) => {
    setRequests(prev => prev.map(r =>
      r.id === reqId ? { ...r, status: "brought" } : r
    ));
  };

  const deleteReq = (reqId) => {
    setRequests(prev => prev.filter(r => r.id !== reqId));
  };

  const filteredItems = activeCategory === "הכל"
    ? ITEMS
    : ITEMS.filter(i => i.category === activeCategory);

  const pendingCount = requests.filter(r => r.status === "pending").length;

  // ─── Login Screen ───────────────────────────────────────────────
  if (screen === "login") {
    return (
      <div style={{
        minHeight: "100vh", background: "#0a0f1e",
        display: "flex", flexDirection: "column", alignItems: "center",
        justifyContent: "center", fontFamily: "'Segoe UI', Arial, sans-serif",
        direction: "rtl", padding: "24px",
        backgroundImage: "radial-gradient(ellipse at 20% 50%, rgba(30,58,138,0.3) 0%, transparent 60%), radial-gradient(ellipse at 80% 20%, rgba(234,88,12,0.15) 0%, transparent 50%)"
      }}>
        {/* Grid overlay */}
        <div style={{
          position: "fixed", inset: 0, pointerEvents: "none",
          backgroundImage: "linear-gradient(rgba(30,58,138,0.08) 1px, transparent 1px), linear-gradient(90deg, rgba(30,58,138,0.08) 1px, transparent 1px)",
          backgroundSize: "40px 40px"
        }} />

        <div style={{ position: "relative", width: "100%", maxWidth: 380 }}>
          {/* Logo */}
          <div style={{ textAlign: "center", marginBottom: 40 }}>
            <div style={{
              display: "inline-flex", alignItems: "center", justifyContent: "center",
              width: 72, height: 72, borderRadius: 18,
              background: "linear-gradient(135deg, #ea580c, #f97316)",
              boxShadow: "0 0 40px rgba(234,88,12,0.4)",
              marginBottom: 20
            }}>
              <Cpu size={36} color="#fff" />
            </div>
            <div style={{ color: "#f97316", fontSize: 13, fontWeight: 700, letterSpacing: 4, textTransform: "uppercase", marginBottom: 6 }}>
              HORTAL MACHINES
            </div>
            <div style={{ color: "#fff", fontSize: 24, fontWeight: 800 }}>
              הורטל מכונות בע"מ
            </div>
            <div style={{ color: "#64748b", fontSize: 13, marginTop: 6 }}>
              מערכת ניהול בקשות חומרים
            </div>
          </div>

          {/* Card */}
          <div style={{
            background: "rgba(15,23,42,0.8)", border: "1px solid rgba(30,58,138,0.4)",
            borderRadius: 20, padding: "32px 28px",
            backdropFilter: "blur(20px)",
            boxShadow: "0 24px 60px rgba(0,0,0,0.5)"
          }}>
            <div style={{ color: "#94a3b8", fontSize: 14, marginBottom: 8 }}>שם הטכנאי</div>
            <input
              ref={nameRef}
              value={techName}
              onChange={e => setTechName(e.target.value)}
              onKeyDown={e => e.key === "Enter" && handleLogin()}
              placeholder="הזן את שמך..."
              style={{
                width: "100%", padding: "14px 16px", borderRadius: 12,
                border: "1.5px solid rgba(30,58,138,0.5)",
                background: "rgba(15,23,42,0.9)", color: "#f8fafc",
                fontSize: 16, outline: "none", boxSizing: "border-box",
                transition: "border-color 0.2s"
              }}
              onFocus={e => e.target.style.borderColor = "#f97316"}
              onBlur={e => e.target.style.borderColor = "rgba(30,58,138,0.5)"}
            />
            <button
              onClick={handleLogin}
              disabled={techName.trim().length < 2}
              style={{
                width: "100%", marginTop: 16, padding: "15px",
                borderRadius: 12, border: "none", cursor: "pointer",
                background: techName.trim().length >= 2
                  ? "linear-gradient(135deg, #ea580c, #f97316)"
                  : "rgba(30,58,138,0.3)",
                color: "#fff", fontSize: 16, fontWeight: 700,
                transition: "all 0.2s",
                boxShadow: techName.trim().length >= 2 ? "0 0 24px rgba(234,88,12,0.35)" : "none"
              }}
            >
              כניסה למערכת
            </button>
          </div>

          <div style={{ textAlign: "center", marginTop: 20, color: "#334155", fontSize: 12 }}>
            <Activity size={12} style={{ display: "inline", marginLeft: 4 }} />
            Live Dashboard • Field Technician Portal
          </div>
        </div>
      </div>
    );
  }

  // ─── Dashboard ───────────────────────────────────────────────────
  return (
    <div style={{
      minHeight: "100vh", background: "#080e1c",
      fontFamily: "'Segoe UI', Arial, sans-serif", direction: "rtl",
      backgroundImage: "radial-gradient(ellipse at 0% 0%, rgba(30,58,138,0.2) 0%, transparent 50%)"
    }}>
      {/* Header */}
      <div style={{
        background: "rgba(10,15,30,0.95)", borderBottom: "1px solid rgba(30,58,138,0.3)",
        padding: "14px 20px", position: "sticky", top: 0, zIndex: 50,
        backdropFilter: "blur(20px)"
      }}>
        <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between" }}>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <div style={{
              width: 36, height: 36, borderRadius: 10,
              background: "linear-gradient(135deg, #ea580c, #f97316)",
              display: "flex", alignItems: "center", justifyContent: "center"
            }}>
              <Cpu size={18} color="#fff" />
            </div>
            <div>
              <div style={{ color: "#f97316", fontSize: 10, fontWeight: 700, letterSpacing: 3 }}>HORTAL</div>
              <div style={{ color: "#e2e8f0", fontSize: 13, fontWeight: 700, lineHeight: 1 }}>לוח בקשות חי</div>
            </div>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
            {pendingCount > 0 && (
              <div style={{
                background: "rgba(245,158,11,0.2)", border: "1px solid rgba(245,158,11,0.4)",
                borderRadius: 20, padding: "4px 10px",
                color: "#f59e0b", fontSize: 12, fontWeight: 700
              }}>
                {pendingCount} ממתינות
              </div>
            )}
            <div style={{ display: "flex", alignItems: "center", gap: 6, color: "#94a3b8", fontSize: 12 }}>
              <User size={14} />
              {techName}
            </div>
          </div>
        </div>
      </div>

      {/* Live indicator */}
      <div style={{ padding: "10px 20px", display: "flex", alignItems: "center", gap: 8 }}>
        <div style={{
          width: 8, height: 8, borderRadius: "50%", background: "#22c55e",
          boxShadow: "0 0 8px #22c55e", animation: "pulse 2s infinite"
        }} />
        <span style={{ color: "#64748b", fontSize: 12 }}>LIVE • {requests.length} בקשות פעילות</span>
      </div>

      {/* Requests List */}
      <div style={{ padding: "0 16px 100px" }}>
        {requests.length === 0 && (
          <div style={{
            textAlign: "center", padding: "60px 20px",
            color: "#334155"
          }}>
            <Box size={48} style={{ margin: "0 auto 16px", opacity: 0.3 }} />
            <div style={{ fontSize: 16, fontWeight: 600, color: "#475569" }}>אין בקשות עדיין</div>
            <div style={{ fontSize: 13, marginTop: 6 }}>לחץ על + כדי להוסיף בקשה חדשה</div>
          </div>
        )}

        {requests.map((req, idx) => {
          const cfg = STATUS_CONFIG[req.status];
          const isMe = req.techName === techName;
          return (
            <div key={req.id} style={{
              background: "rgba(15,23,42,0.8)", border: `1px solid ${req.status === "brought" ? "rgba(34,197,94,0.2)" : "rgba(30,58,138,0.3)"}`,
              borderRadius: 16, marginBottom: 12, overflow: "hidden",
              boxShadow: "0 4px 24px rgba(0,0,0,0.3)",
              animation: "slideIn 0.3s ease"
            }}>
              {/* Card Header */}
              <div style={{
                padding: "14px 16px", display: "flex",
                alignItems: "center", justifyContent: "space-between",
                borderBottom: "1px solid rgba(30,58,138,0.2)"
              }}>
                <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                  <div style={{
                    width: 32, height: 32, borderRadius: 8,
                    background: isMe ? "rgba(234,88,12,0.2)" : "rgba(30,58,138,0.3)",
                    border: `1px solid ${isMe ? "rgba(234,88,12,0.4)" : "rgba(30,58,138,0.4)"}`,
                    display: "flex", alignItems: "center", justifyContent: "center"
                  }}>
                    <User size={15} color={isMe ? "#f97316" : "#60a5fa"} />
                  </div>
                  <div>
                    <div style={{ color: "#e2e8f0", fontWeight: 700, fontSize: 14 }}>{req.techName}</div>
                    <div style={{ color: "#475569", fontSize: 11, display: "flex", alignItems: "center", gap: 4 }}>
                      <Clock size={10} /> {timeAgo(req.createdAt)}
                    </div>
                  </div>
                </div>
                <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                  <div style={{
                    padding: "4px 10px", borderRadius: 20, fontSize: 11, fontWeight: 700,
                    color: cfg.color, background: cfg.bg, border: `1px solid ${cfg.color}40`
                  }}>
                    {cfg.label}
                  </div>
                  {isMe && (
                    <button onClick={() => deleteReq(req.id)} style={{
                      background: "rgba(239,68,68,0.1)", border: "1px solid rgba(239,68,68,0.3)",
                      borderRadius: 8, padding: "4px", cursor: "pointer",
                      display: "flex", alignItems: "center", color: "#ef4444"
                    }}>
                      <Trash2 size={13} />
                    </button>
                  )}
                </div>
              </div>

              {/* Items */}
              <div style={{ padding: "12px 16px" }}>
                <div style={{ display: "flex", flexWrap: "wrap", gap: 6 }}>
                  {req.items.map((item, i) => (
                    <div key={i} style={{
                      background: "rgba(30,58,138,0.2)", border: "1px solid rgba(30,58,138,0.35)",
                      borderRadius: 8, padding: "5px 10px",
                      color: "#93c5fd", fontSize: 12, fontWeight: 600,
                      display: "flex", alignItems: "center", gap: 5
                    }}>
                      <span>{item.label}</span>
                      {item.qty > 1 && (
                        <span style={{
                          background: "#f97316", color: "#fff",
                          borderRadius: 10, padding: "1px 6px", fontSize: 10, fontWeight: 800
                        }}>×{item.qty}</span>
                      )}
                    </div>
                  ))}
                </div>
              </div>

              {/* Action */}
              {req.status === "pending" && (
                <div style={{ padding: "0 16px 14px" }}>
                  <button onClick={() => markDone(req.id)} style={{
                    width: "100%", padding: "10px", borderRadius: 10,
                    border: "1px solid rgba(34,197,94,0.4)",
                    background: "rgba(34,197,94,0.1)", color: "#4ade80",
                    fontSize: 13, fontWeight: 700, cursor: "pointer",
                    display: "flex", alignItems: "center", justifyContent: "center", gap: 6,
                    transition: "all 0.2s"
                  }}
                    onMouseEnter={e => e.currentTarget.style.background = "rgba(34,197,94,0.2)"}
                    onMouseLeave={e => e.currentTarget.style.background = "rgba(34,197,94,0.1)"}
                  >
                    <CheckCircle2 size={15} /> סמן כהובא
                  </button>
                </div>
              )}
            </div>
          );
        })}
      </div>

      {/* FAB */}
      <button
        onClick={() => setShowNewRequest(true)}
        style={{
          position: "fixed", bottom: 28, left: "50%", transform: "translateX(-50%)",
          background: "linear-gradient(135deg, #ea580c, #f97316)",
          border: "none", borderRadius: 50, padding: "16px 32px",
          color: "#fff", fontSize: 15, fontWeight: 800, cursor: "pointer",
          display: "flex", alignItems: "center", gap: 8,
          boxShadow: "0 8px 32px rgba(234,88,12,0.5)",
          whiteSpace: "nowrap"
        }}
      >
        <Plus size={20} /> בקשה חדשה
      </button>

      {/* New Request Modal */}
      {showNewRequest && (
        <div style={{
          position: "fixed", inset: 0, background: "rgba(0,0,0,0.85)",
          zIndex: 100, display: "flex", alignItems: "flex-end",
          backdropFilter: "blur(8px)"
        }}>
          <div style={{
            background: "#0a0f1e", width: "100%",
            height: "92vh",
            display: "flex", flexDirection: "column",
            borderTop: "1px solid rgba(30,58,138,0.4)",
            borderRadius: "24px 24px 0 0",
            overflow: "hidden"
          }}>
            {/* Modal Header */}
            <div style={{
              padding: "20px 20px 16px",
              borderBottom: "1px solid rgba(30,58,138,0.3)",
              display: "flex", alignItems: "center", justifyContent: "space-between"
            }}>
              <div>
                <div style={{ color: "#f97316", fontSize: 11, fontWeight: 700, letterSpacing: 3 }}>NEW REQUEST</div>
                <div style={{ color: "#e2e8f0", fontSize: 18, fontWeight: 800 }}>בקשת חומרים חדשה</div>
              </div>
              <button onClick={() => setShowNewRequest(false)} style={{
                background: "rgba(239,68,68,0.1)", border: "1px solid rgba(239,68,68,0.3)",
                borderRadius: 10, padding: "8px", cursor: "pointer", color: "#ef4444",
                display: "flex"
              }}>
                <X size={18} />
              </button>
            </div>

            {/* Selected Count */}
            {Object.keys(selectedItems).length > 0 && (
              <div style={{
                padding: "10px 20px", background: "rgba(234,88,12,0.1)",
                borderBottom: "1px solid rgba(234,88,12,0.2)",
                display: "flex", alignItems: "center", gap: 8
              }}>
                <AlertTriangle size={14} color="#f97316" />
                <span style={{ color: "#f97316", fontSize: 13, fontWeight: 600 }}>
                  {Object.keys(selectedItems).length + (otherText.trim() ? 1 : 0)} פריטים נבחרו
                </span>
              </div>
            )}

            {/* Category Tabs */}
            <div style={{
              display: "flex", gap: 8, padding: "12px 16px",
              overflowX: "auto", borderBottom: "1px solid rgba(30,58,138,0.2)"
            }}>
              {["הכל", ...CATEGORIES].map(cat => (
                <button key={cat} onClick={() => setActiveCategory(cat)} style={{
                  padding: "6px 14px", borderRadius: 20, border: "none",
                  background: activeCategory === cat ? "#f97316" : "rgba(30,58,138,0.3)",
                  color: activeCategory === cat ? "#fff" : "#64748b",
                  fontSize: 12, fontWeight: 700, cursor: "pointer", whiteSpace: "nowrap",
                  transition: "all 0.2s"
                }}>
                  {cat}
                </button>
              ))}
            </div>

            {/* Items Grid */}
            <div style={{ flex: 1, overflowY: "auto", padding: "16px" }}>
              <div style={{
                display: "grid", gridTemplateColumns: "1fr 1fr",
                gap: 10
              }}>
                {filteredItems.map(item => {
                  const Icon = item.icon;
                  const selected = !!selectedItems[item.id];
                  const qty = selectedItems[item.id] || 1;
                  return (
                    <div key={item.id} style={{
                      background: selected ? "rgba(234,88,12,0.15)" : "rgba(15,23,42,0.8)",
                      border: `1.5px solid ${selected ? "#f97316" : "rgba(30,58,138,0.3)"}`,
                      borderRadius: 12, padding: "12px",
                      cursor: "pointer", transition: "all 0.15s",
                      boxShadow: selected ? "0 0 16px rgba(234,88,12,0.2)" : "none"
                    }}
                      onClick={() => toggleItem(item.id)}
                    >
                      <div style={{ display: "flex", alignItems: "flex-start", gap: 8 }}>
                        <div style={{
                          width: 32, height: 32, borderRadius: 8, flexShrink: 0,
                          background: selected ? "rgba(234,88,12,0.3)" : "rgba(30,58,138,0.3)",
                          display: "flex", alignItems: "center", justifyContent: "center"
                        }}>
                          <Icon size={15} color={selected ? "#f97316" : "#60a5fa"} />
                        </div>
                        <div style={{ flex: 1, minWidth: 0 }}>
                          <div style={{
                            color: selected ? "#fed7aa" : "#cbd5e1",
                            fontSize: 12, fontWeight: 600, lineHeight: 1.3
                          }}>
                            {item.label}
                          </div>
                          <div style={{ color: "#475569", fontSize: 10 }}>{item.category}</div>
                        </div>
                      </div>
                      {selected && (
                        <div style={{
                          display: "flex", alignItems: "center", justifyContent: "center",
                          gap: 12, marginTop: 10
                        }} onClick={e => e.stopPropagation()}>
                          <button onClick={() => updateQty(item.id, -1)} style={{
                            width: 26, height: 26, borderRadius: 6,
                            background: "rgba(234,88,12,0.3)", border: "1px solid rgba(234,88,12,0.5)",
                            color: "#f97316", cursor: "pointer", fontSize: 16, fontWeight: 800,
                            display: "flex", alignItems: "center", justifyContent: "center"
                          }}>−</button>
                          <span style={{ color: "#fff", fontWeight: 800, fontSize: 15, minWidth: 20, textAlign: "center" }}>{qty}</span>
                          <button onClick={() => updateQty(item.id, 1)} style={{
                            width: 26, height: 26, borderRadius: 6,
                            background: "rgba(234,88,12,0.3)", border: "1px solid rgba(234,88,12,0.5)",
                            color: "#f97316", cursor: "pointer", fontSize: 16, fontWeight: 800,
                            display: "flex", alignItems: "center", justifyContent: "center"
                          }}>+</button>
                        </div>
                      )}
                    </div>
                  );
                })}
              </div>

              {/* Other */}
              <div style={{
                marginTop: 12, background: "rgba(15,23,42,0.8)",
                border: "1.5px solid rgba(30,58,138,0.3)", borderRadius: 12, padding: 12
              }}>
                <div style={{ color: "#94a3b8", fontSize: 12, fontWeight: 700, marginBottom: 8, display: "flex", alignItems: "center", gap: 6 }}>
                  <Tag size={13} /> אחר (הזן בחופשיות)
                </div>
                <textarea
                  value={otherText}
                  onChange={e => setOtherText(e.target.value)}
                  placeholder="פריט שאינו ברשימה..."
                  rows={2}
                  style={{
                    width: "100%", background: "rgba(10,15,30,0.8)",
                    border: "1px solid rgba(30,58,138,0.4)", borderRadius: 8,
                    color: "#e2e8f0", fontSize: 13, padding: "8px 10px",
                    outline: "none", resize: "none", boxSizing: "border-box",
                    fontFamily: "'Segoe UI', Arial, sans-serif"
                  }}
                />
              </div>
            </div>

            {/* Submit — always visible */}
            <div style={{ padding: "14px 16px", borderTop: "1px solid rgba(30,58,138,0.3)", background: "#0a0f1e", flexShrink: 0 }}>
              <button
                onClick={submitRequest}
                style={{
                  width: "100%", padding: "16px",
                  borderRadius: 14, border: "none", cursor: "pointer",
                  background: (Object.keys(selectedItems).length > 0 || otherText.trim())
                    ? "linear-gradient(135deg, #ea580c, #f97316)"
                    : "rgba(100,116,139,0.35)",
                  color: "#fff", fontSize: 16, fontWeight: 800,
                  display: "flex", alignItems: "center", justifyContent: "center", gap: 8,
                  boxShadow: (Object.keys(selectedItems).length > 0 || otherText.trim()) ? "0 0 28px rgba(234,88,12,0.45)" : "none",
                  transition: "all 0.2s"
                }}
              >
                <Send size={18} />
                {(Object.keys(selectedItems).length > 0 || otherText.trim())
                  ? `שלח בקשה (${Object.keys(selectedItems).length + (otherText.trim() ? 1 : 0)} פריטים)`
                  : "בחר פריטים תחילה"}
              </button>
            </div>
          </div>
        </div>
      )}

      <style>{`
        @keyframes slideIn {
          from { opacity: 0; transform: translateY(-10px); }
          to { opacity: 1; transform: translateY(0); }
        }
        @keyframes pulse {
          0%, 100% { opacity: 1; }
          50% { opacity: 0.4; }
        }
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(30,58,138,0.5); border-radius: 4px; }
        * { -webkit-tap-highlight-color: transparent; }
      `}</style>
    </div>
  );
}
