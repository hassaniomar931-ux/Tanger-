Omar# Tanger-
Tangier chicken
// ========== إعدادات المتجر (عدّلها) ==========
const WHATSAPP_NUMBER = "212600000000"; // رقمك بصيغة دولية بدون + (مثال المغرب: 2126XXXXXXXX)
const CURRENCY = "درهم";

// ========== المنتجات (عدّلها أو زد عليها) ==========
// image: ضع رابط/مسار صورة (مثال: "images/eggs.jpg") أو اتركه فارغا لعرض الإيموجي
const PRODUCTS = [
  { id: 1,  cat: "دواجن",  name: "دجاج بلدي",        desc: "دجاج بلدي حي أو مذبوح حسب الطلب", price: 70,  unit: "للحبة",  emoji: "🐔", image: "" },
  { id: 2,  cat: "دواجن",  name: "ديك رومي",         desc: "ديك رومي سمين وصحي",             price: 350, unit: "للحبة",  emoji: "🦃", image: "" },
  { id: 3,  cat: "بيض",    name: "بيض بلدي",         desc: "بيض طازج من دجاج حر",            price: 30,  unit: "للدزينة", emoji: "🥚", image: "" },
  { id: 4,  cat: "بيض",    name: "بيض المائدة",      desc: "بيض طازج يومي",                  price: 25,  unit: "للدزينة", emoji: "🥚", image: "" },
  { id: 5,  cat: "أرانب",  name: "أرنب للتربية",     desc: "أرانب سليمة ومُلقّحة",            price: 120, unit: "للحبة",  emoji: "🐇", image: "" },
  { id: 6,  cat: "أرانب",  name: "أرنب للذبح",       desc: "أرانب جاهزة للذبح",              price: 90,  unit: "للحبة",  emoji: "🐰", image: "" },
  { id: 7,  cat: "بيبي",   name: "كتاكيت صغار",      desc: "كتاكيت عمر أيام، تُباع بالعدد",   price: 12,  unit: "للحبة",  emoji: "🐥", image: "" },
  { id: 8,  cat: "بيبي",   name: "صغار الأرانب",     desc: "صغار أرانب بعد الفطام",           price: 60,  unit: "للحبة",  emoji: "🐇", image: "" },
  { id: 9,  cat: "خرفان",  name: "خروف العيد",       desc: "خروف سمين، اتصل لمعرفة الأوزان",  price: 2500, unit: "للرأس", emoji: "🐏", image: "" },
  { id: 10, cat: "خرفان",  name: "حولي صغير",        desc: "حولي للتربية أو الذبح",           price: 1500, unit: "للرأس", emoji: "🐑", image: "" },
];

// ========== الكود (لا تحتاج تعديله) ==========
const grid = document.getElementById("grid");
const filtersEl = document.getElementById("filters");
const cartEl = document.getElementById("cart");
const overlay = document.getElementById("overlay");
const cartItemsEl = document.getElementById("cartItems");
const cartCountEl = document.getElementById("cartCount");
const cartTotalEl = document.getElementById("cartTotal");

let cart = loadCart();
let activeCat = "الكل";

function loadCart() {
  try { return JSON.parse(localStorage.getItem("cart")) || {}; } catch { return {}; }
}
function saveCart() {
  try { localStorage.setItem("cart", JSON.stringify(cart)); } catch {}
}

function renderFilters() {
  const cats = ["الكل", ...new Set(PRODUCTS.map(p => p.cat))];
  filtersEl.innerHTML = cats.map(c =>
    `<button class="${c === activeCat ? "active" : ""}" data-cat="${c}">${c}</button>`
  ).join("");
  filtersEl.querySelectorAll("button").forEach(b =>
    b.addEventListener("click", () => { activeCat = b.dataset.cat; renderFilters(); renderProducts(); })
  );
}

function renderProducts() {
  const list = activeCat === "الكل" ? PRODUCTS : PRODUCTS.filter(p => p.cat === activeCat);
  grid.innerHTML = list.map(p => `
    <div class="card">
      <div class="pic">${p.image ? `<img src="${p.image}" alt="${p.name}">` : p.emoji}</div>
      <div class="info">
        <h3>${p.name}</h3>
        <p class="desc">${p.desc}</p>
        <p class="price">${p.price} ${CURRENCY} <small>${p.unit}</small></p>
        <button data-add="${p.id}">أضف إلى السلة</button>
      </div>
    </div>`).join("");
  grid.querySelectorAll("[data-add]").forEach(b =>
    b.addEventListener("click", () => addToCart(Number(b.dataset.add)))
  );
}

function addToCart(id) { cart[id] = (cart[id] || 0) + 1; saveCart(); renderCart(); openCart(); }
function changeQty(id, d) {
  cart[id] = (cart[id] || 0) + d;
  if (cart[id] <= 0) delete cart[id];
  saveCart(); renderCart();
}

function renderCart() {
  const ids = Object.keys(cart);
  let total = 0, count = 0;
  cartItemsEl.innerHTML = ids.length ? ids.map(id => {
    const p = PRODUCTS.find(x => x.id == id);
    if (!p) return "";
    const q = cart[id];
    total += p.price * q; count += q;
    return `<div class="cart-item">
      <span>${p.name}<br><small>${p.price * q} ${CURRENCY}</small></span>
      <span class="qty">
        <button data-d="1" data-id="${id}">+</button>
        <b>${q}</b>
        <button data-d="-1" data-id="${id}">−</button>
      </span>
    </div>`;
  }).join("") : "<p>السلة فارغة</p>";
  cartItemsEl.querySelectorAll("[data-d]").forEach(b =>
    b.addEventListener("click", () => changeQty(b.dataset.id, Number(b.dataset.d)))
  );
  cartCountEl.textContent = count;
  cartTotalEl.textContent = `${total} ${CURRENCY}`;
}

function openCart() { cartEl.classList.add("open"); overlay.classList.add("show"); }
function closeCart() { cartEl.classList.remove("open"); overlay.classList.remove("show"); }

document.getElementById("cartBtn").addEventListener("click", openCart);
document.getElementById("closeCart").addEventListener("click", closeCart);
overlay.addEventListener("click", closeCart);

document.getElementById("orderBtn").addEventListener("click", () => {
  const ids = Object.keys(cart);
  if (!ids.length) return alert("السلة فارغة");
  let total = 0;
  const lines = ids.map(id => {
    const p = PRODUCTS.find(x => x.id == id);
    total += p.price * cart[id];
    return `• ${p.name} × ${cart[id]} = ${p.price * cart[id]} ${CURRENCY}`;
  });
  const msg = `السلام عليكم، أريد طلب:\n${lines.join("\n")}\nالمجموع: ${total} ${CURRENCY}`;
  window.open(`https://wa.me/${WHATSAPP_NUMBER}?text=${encodeURIComponent(msg)}`, "_blank");
});

const phoneLink = document.getElementById("phoneLink");
phoneLink.textContent = "+" + WHATSAPP_NUMBER;
phoneLink.href = "https://wa.me/" + WHATSAPP_NUMBER;
document.getElementById("year").textContent = new Date().getFullYear();

renderFilters();
renderProducts();
renderCart();
