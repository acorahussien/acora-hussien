/*
ACORA AI — Single-file React component (Tailwind CSS)

How to use:
1. Create a React app (Vite / Create React App). Ensure Tailwind CSS is configured.
2. Save this file as `ACORAAI.jsx` inside `src/components/` (or `src/`).
3. Import and render <ACORAAI /> in your App.
4. To connect to a real AI backend, implement an API endpoint (e.g. /api/chat) that accepts POST { message } and returns { reply }.
   - For production with OpenAI, keep your API key on the server and forward requests securely.
   - This component includes a mock mode for local testing.

Features included:
- Modern, responsive chat UI
- Message history and simple local persistence (localStorage)
- Typing indicator and streaming-style reply (simulated)
- System/preset prompts, model selector placeholder
- Clear, export, and copy buttons

Notes:
- Styling uses Tailwind utility classes. No external CSS file required if Tailwind is configured.
- Replace the `sendToApi` function with your real API request.

*/

import React, { useEffect, useRef, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Send, Trash2, RefreshCw, Copy, Bot, User } from "lucide-react";

export default function ACORAAI() {
  const [messages, setMessages] = useState(() => {
    try {
      const raw = localStorage.getItem("acora_messages_v1");
      return raw ? JSON.parse(raw) : [
        { id: 1, role: "system", text: "أنت المساعد ACORA AI — لطيف ومفيد." },
        { id: 2, role: "assistant", text: "أهلاً! أنا ACORA AI. إزيك؟ إزي أقدر أساعدك النهاردة؟" },
      ];
    } catch (e) {
      return [];
    }
  });
  const [input, setInput] = useState("");
  const [isSending, setIsSending] = useState(false);
  const [mockMode, setMockMode] = useState(true); // change to false to call real API
  const [model, setModel] = useState("acora-lite");
  const messagesEndRef = useRef(null);
  const inputRef = useRef(null);

  useEffect(() => {
    localStorage.setItem("acora_messages_v1", JSON.stringify(messages));
    scrollToBottom();
  }, [messages]);

  function scrollToBottom() {
    if (messagesEndRef.current) messagesEndRef.current.scrollIntoView({ behavior: "smooth" });
  }

  function addMessage(role, text) {
    setMessages((m) => [...m, { id: Date.now(), role, text }]);
  }

  async function handleSend(e) {
    e?.preventDefault();
    const trimmed = input.trim();
    if (!trimmed) return;
    addMessage("user", trimmed);
    setInput("");
    setIsSending(true);

    try {
      const reply = await sendToApi(trimmed, { model, mock: mockMode });
      addMessage("assistant", reply);
    } catch (err) {
      addMessage("assistant", "حصل خطأ في الاتصال. حاول بعد شوية.");
      console.error(err);
    } finally {
      setIsSending(false);
      inputRef.current?.focus();
    }
  }

  // Replace this with your real API call.
  async function sendToApi(message, { model, mock }) {
    if (mock) {
      // simulate streaming by delaying and returning a crafted response
      await new Promise((r) => setTimeout(r, 800 + Math.random() * 700));
      const canned = `فهمت: "${message}". دي إجابة تجريبية من ACORA AI (موديل ${model}).`;
      return canned;
    }

    // Example fetch to your backend endpoint. Uncomment and adjust.
    // const res = await fetch('/api/chat', {
    //   method: 'POST',
    //   headers: { 'Content-Type': 'application/json' },
    //   body: JSON.stringify({ message, model })
    // });
    // if (!res.ok) throw new Error('API error');
    // const j = await res.json();
    // return j.reply;

    throw new Error('No API configured');
  }

  function clearConversation() {
    if (!confirm("هل أنت متأكد إنك عايز تمسح المحادثة؟")) return;
    const sys = messages.find((m) => m.role === "system");
    setMessages(sys ? [sys] : []);
  }

  function exportConversation() {
    const blob = new Blob([JSON.stringify(messages, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "acora_conversation.json";
    a.click();
    URL.revokeObjectURL(url);
  }

  function copyLastReply() {
    const last = [...messages].reverse().find((m) => m.role === "assistant");
    if (!last) return alert("ما فيش ردنسخة");
    navigator.clipboard.writeText(last.text).then(() => alert("اتنسخت!"));
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-50 to-white flex items-center justify-center p-4">
      <div className="w-full max-w-3xl bg-white shadow-2xl rounded-2xl overflow-hidden grid grid-cols-1 md:grid-cols-3">
        {/* Left: brand & controls */}
        <aside className="md:col-span-1 p-5 bg-gradient-to-b from-indigo-600 to-violet-600 text-white">
          <div className="flex items-center gap-3">
            <div className="w-12 h-12 rounded-lg bg-white/20 flex items-center justify-center text-2xl font-bold">AC</div>
            <div>
              <h1 className="text-lg font-semibold">ACORA AI</h1>
              <p className="text-sm opacity-80">مساعد ذكي عربي — نسخة تجريبية</p>
            </div>
          </div>

          <div className="mt-6 space-y-3">
            <div className="flex items-center justify-between">
              <label className="text-sm">موديل</label>
              <select value={model} onChange={(e) => setModel(e.target.value)} className="rounded-md text-black px-2 py-1">
                <option value="acora-lite">acora-lite</option>
                <option value="acora-pro">acora-pro</option>
              </select>
            </div>

            <div className="flex items-center justify-between">
              <label className="text-sm">وضع تجريبي</label>
              <input type="checkbox" checked={mockMode} onChange={(e) => setMockMode(e.target.checked)} />
            </div>

            <div className="pt-2">
              <button onClick={clearConversation} className="w-full flex gap-2 items-center justify-center bg-white/10 hover:bg-white/20 rounded-md px-3 py-2">
                <Trash2 size={16} /> مسح المحادثة
              </button>
            </div>

            <div className="grid grid-cols-2 gap-2 mt-2">
              <button onClick={exportConversation} className="text-sm bg-white/10 rounded-md px-2 py-1">تصدير</button>
              <button onClick={copyLastReply} className="text-sm bg-white/10 rounded-md px-2 py-1">نسخ آخر رد</button>
            </div>

            <p className="text-xs opacity-80 mt-4">ملحوظة: في الوضع الحقيقي، شغّل رابط الـ API في الخادم واحتفظ بمفتاح الـ API بعيداً عن المتصفح.</p>
          </div>
        </aside>

        {/* Center: chat */}
        <main className="md:col-span-2 flex flex-col">
          <div className="flex-1 overflow-auto p-6" style={{ background: "linear-gradient(180deg,#f8fafc, #fff)" }}>
            <div className="max-w-2xl mx-auto">
              <AnimatePresence initial={false}>
                {messages.map((m) => (
                  <motion.div key={m.id} initial={{ opacity: 0, y: 6 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0 }} className={m.role === 'user' ? 'text-right' : 'text-left'}>
                    <div className={`inline-block my-2 px-4 py-2 rounded-2xl shadow-sm ${m.role === 'user' ? 'bg-indigo-600 text-white rounded-br-none' : 'bg-gray-100 text-black rounded-bl-none'}`}>
                      <div className="text-sm whitespace-pre-wrap">{m.text}</div>
                    </div>
                  </motion.div>
                ))}
              </AnimatePresence>

              <div ref={messagesEndRef} />
            </div>
          </div>

          <form onSubmit={handleSend} className="p-4 border-t bg-white flex items-center gap-3">
            <textarea
              ref={inputRef}
              value={input}
              onChange={(e) => setInput(e.target.value)}
              rows={1}
              className="flex-1 resize-none rounded-xl border px-3 py-2 focus:outline-none focus:ring"
              placeholder="اكتب رسالتك هنا..."
            />

            <div className="flex items-center gap-2">
              <button type="button" onClick={() => { setInput(''); inputRef.current?.focus(); }} className="p-2 rounded-md hover:bg-slate-100">
                <RefreshCw size={16} />
              </button>
              <button type="button" onClick={() => { navigator.clipboard.writeText(messages.map(m=>`${m.role}: ${m.text}`).join('\n\n')) }} className="p-2 rounded-md hover:bg-slate-100">
                <Copy size={16} />
              </button>
              <button type="submit" disabled={isSending} className="flex items-center gap-2 bg-indigo-600 disabled:opacity-60 text-white px-4 py-2 rounded-xl">
                <Send size={16} /> {isSending ? 'جارٍ الإرسال...' : 'إرسال'}
              </button>
            </div>
          </form>
        </main>
      </div>
    </div>
  );
}
