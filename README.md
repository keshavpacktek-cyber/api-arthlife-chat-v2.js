# api-arthlife-chat-v2.js
// Arthlife — Smart Brand-Scoped Chat (v r6)
const VERSION = "arthlife-chat:r6";

export default async function handler(req, res) {
  // CORS + version header so we can see it in tests
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
  res.setHeader("X-Arth-Version", VERSION);

  if (req.method === "OPTIONS") return res.status(200).end();
  if (req.method !== "POST") {
    return res.status(405).json({ error: "POST only", version: VERSION });
  }

  const { message = "" } = req.body || {};
  const raw = String(message || "").trim();
  if (!raw) return res.status(400).json({ error: "message required", version: VERSION });

  const lower = raw.toLowerCase();
  const has = (...keys) => keys.some(k => lower.includes(k));
  const take = (rgx) => { const m = raw.match(rgx); return m ? (m[1] || m[0]) : null; };
  const re  = (r) => r.test(lower);

  const pincode = take(/\b([1-9][0-9]{5})\b/);
  const orderId = take(/\border(?:\s*id|#)?\s*[:\-]?\s*([a-z0-9\-]{4,18})\b/i) || take(/\b(\d{4,12})\b/);

  // brand guard
  const BRAND = [
    "arthlife","bracelet","gemstone","crystal","product","order","delivery","shipping","pincode","cod",
    "size","fit","return","exchange","replace","refund","payment","checkout","authentic","cleanse",
    "charge","care","warranty","packaging","gift","availability","stock","bundle","policy","threshold",
    "whatsapp","email","support","invoice","tracking","awb","dispatch","cart"
  ];
  if (!BRAND.some(w => lower.includes(w))) {
    return res.json({
      version: VERSION,
      reply:
`Main **Arthlife** ke products, orders, delivery, size/fit, returns, payments aur care se related madad karta/kartee hoon 🌿
Examples: “Where is my order”, “Return/Exchange”, “Pincode 560001 delivery time”, “Size guide”, “Is COD available?”.`
    });
  }

  const ORDER_ID_HELP =
`**Order ID/Tracking ID kaise milega**  
• Email: “Arthlife – Order Confirmation / Order # …” (Inbox/Spam)  
• SMS/WhatsApp: confirmation/AWB message  
• “My Account → Orders” (agar account bana tha)
Mil jaaye to yahin likh dein: “Order ID 1004” ya “AWB 12345…”.`;

  const CONTACT = `WhatsApp: **+91 97177 09426**  •  Email: **support@arthlife.in**`;

  // intents
  if (has("where is my order","order status","track","tracking","awb","consignment") ||
      re(/\b(where|kahan|kidhar)\b.*\border\b/) || re(/\border\b.*\b(track|status)\b/)) {
    if (orderId) {
      return res.json({
        version: VERSION,
        reply:
`**Order ${orderId} – Tracking help**  
• AWB milte hi WhatsApp/Email aata hai.  
• Agar AWB hai to courier portal par track karein; agar nahi mila to mai fetch kara deta/deti hoon.  
${CONTACT}`
      });
    }
    return res.json({
      version: VERSION,
      reply:
`Order tracking me madad karta/kartee hoon 🙏  
Aap **Order ID** share karein to mai status check kara dunga/dungi.  
${ORDER_ID_HELP}`
    });
  }

  if (has("return","exchange","replace","size change","replacement","refund","badalna")) {
    return res.json({
      version: VERSION,
      reply:
`**7-day Easy Returns/Exchange/Replace**  
• Condition: **unused**, full packing & tags intact  
• Pickup arrange hota hai; refund/exchange initiate  
${orderId ? `**Order ${orderId}** mila — kripya 1 line reason likh dein, mai pickup start kara deta/deti hoon.` : ORDER_ID_HELP}`
    });
  }

  if (has("delivery","pincode","pin code","eta","kitne din","kab tak aa","shipping time","reach")) {
    if (pincode) {
      return res.json({
        version: VERSION,
        reply:
`**Pincode ${pincode}**: 92% orders **2–5 din**; remote/ODF: **5–7 din**.  
Dispatch 24–48 hrs me.`
      });
    }
    return res.json({
      version: VERSION,
      reply: `Typical delivery **2–5 din** (remote **5–7 din**). Apna **pincode** bhejenge to exact ETA bata dunga/dungi.`
    });
  }

  if (has("cod","cash on delivery","cash on del")) {
    return res.json({ version: VERSION, reply: `**COD available** ✅ (kuch pincodes par verification/limit ho sakti hai). Checkout me select kar sakte hain.` });
  }

  if (has("size","fit","guide","measure","wrist","kaise naap","kaise measure")) {
    return res.json({
      version: VERSION,
      reply:
`**Size / Fit Guide**  
1) Dhaaga wrist par lapet kar mark karein → scale se **cm** me naap lein  
2) Choose: **Snug** = wrist, **Comfort** = wrist + 0.5–1cm, **Loose** = wrist + 1–1.5cm`
    });
  }

  if (has("authentic","original","genuine","real","fake","nakli","lab","certificate")) {
    return res.json({ version: VERSION, reply: `Hamare gemstones **lab-verified & natural** hote hain 🌿  Colour/pattern me halka natural variation normal hai.` });
  }

  if (has("cleanse","charge","energ","ritual","how to use","kaise use","kaise wear")) {
    return res.json({
      version: VERSION,
      reply:
`**Cleanse & Charge**  
• Mist/salt se cleanse  • Intention set karke daily pehnein  
• Weekly **2–3 min sunlight** me recharge karein`
    });
  }

  if (has("payment","checkout","upi","card","netbanking","order place","failed","error","decline")) {
    return res.json({
      version: VERSION,
      reply:
`**Payment/Checkout help**  
• Cache clear ya **incognito** try karein • UPI/Card/NetBanking available  
• Error ka screenshot bhej dein — mai guide kar dunga/dungi  
${CONTACT}`
    });
  }

  if (has("free shipping","threshold","shipping charge","delivery charges","minimum order")) {
    return res.json({ version: VERSION, reply: `**Free Shipping threshold** site par same hai (e.g., ₹999). Cart ke top goal-bar se status dekh sakte hain.` });
  }

  if (has("available","availability","stock","out of stock","restock","kab aayega")) {
    return res.json({ version: VERSION, reply: `Agar product “Out of stock” ho to page par **Notify me** use karein — restock hote hi message aa jayega.` });
  }

  if (has("custom","personalise","engrave","gift","gift wrap","packaging","hamper")) {
    return res.json({ version: VERSION, reply: `Gift packaging available 🎁 — ritual pouch & message card. Custom request ke liye WhatsApp par ping karein.` });
  }

  if (has("care","maintain","clean","perfume","warranty","guarantee","damage")) {
    return res.json({ version: VERSION, reply: `Care: harsh soaps/perfumes se bachayein, soft cloth se wipe karein, ritual pouch me store karein. Warranty: defects ko case-by-case resolve karte hain — pics share karein.` });
  }

  if (has("contact","support","help","whatsapp","email","phone","call")) {
    return res.json({ version: VERSION, reply: CONTACT });
  }

  if (orderId) {
    return res.json({ version: VERSION, reply: `**Order ${orderId}** mila 🙏  Bataye: **track / return / exchange / replace** me se kis me help chahiye?` });
  }

  return res.json({
    version: VERSION,
    reply:
`Main madad ke liye yahan hoon. Aap bata sakte hain: **Track order • Return/Exchange/Replace • Delivery time (pincode) • Size/fit • COD/Payment • Authenticity • Care**  
Agar Order ID/AWB yaad nahi hai:  
${ORDER_ID_HELP}`
  });
}
