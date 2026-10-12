const products = [
  { id: "cpu-ryzen7900x", name: "AMD Ryzen 9 7900X", category: "cpu", price: 489, tag: "CPU", imageClass: "image-2" },
  { id: "cpu-ryzen7600", name: "AMD Ryzen 5 7600", category: "cpu", price: 229, tag: "CPU", imageClass: "image-2" },
  { id: "gpu-rtx4070", name: "MSI GeForce RTX 4070 Super", category: "gpu", price: 799, tag: "GPU", imageClass: "image-1" },
  { id: "gpu-rtx4080", name: "NVIDIA GeForce RTX 4080", category: "gpu", price: 1199, tag: "GPU", imageClass: "image-1" },
  { id: "gpu-rx7900xt", name: "AMD Radeon RX 7900 XT", category: "gpu", price: 899, tag: "GPU", imageClass: "image-7" },
  { id: "ram-ddr5-32", name: "G.Skill DDR5 32GB 6000MHz", category: "ram", price: 179, tag: "RAM", imageClass: "image-3" },
  { id: "ram-ddr4-16", name: "Corsair DDR4 16GB 3200MHz", category: "ram", price: 79, tag: "RAM", imageClass: "image-3" },
  { id: "ssd-990pro-2tb", name: "Samsung 990 Pro 2TB NVMe", category: "storage", price: 169, tag: "SSD", imageClass: "image-4" },
  { id: "ssd-870qvo-4tb", name: "Samsung 870 QVO 4TB SATA", category: "storage", price: 219, tag: "SSD", imageClass: "image-4" },
  { id: "hdd-western-8tb", name: "WD Blue 8TB HDD", category: "storage", price: 169, tag: "HDD", imageClass: "image-5" },
  { id: "m2-gen4-1tb", name: "Crucial P5 Plus 1TB M.2", category: "storage", price: 89, tag: "M.2", imageClass: "image-4" },
  { id: "cooler-kraken", name: "NZXT Kraken 240 RGB AIO", category: "cooling", price: 199, tag: "Cooling", imageClass: "image-6" },
  { id: "fan-noctua-140", name: "Noctua NF-A14 PWM Fan", category: "cooling", price: 29, tag: "Fan", imageClass: "image-6" },
  { id: "motherboard-b650", name: "MSI B650 Tomahawk WiFi", category: "motherboard", price: 249, tag: "Board", imageClass: "image-8" },
  { id: "motherboard-x670", name: "ASUS ROG X670E Hero", category: "motherboard", price: 499, tag: "Board", imageClass: "image-8" }
];

const storageKey = "voltcore-cart";

function getCart() {
  try {
    return JSON.parse(localStorage.getItem(storageKey)) || [];
  } catch (error) {
    return [];
  }
}

function saveCart(cart) {
  localStorage.setItem(storageKey, JSON.stringify(cart));
}

function updateCartBadge() {
  const badge = document.getElementById("cart-badge");
  const cart = getCart();
  if (badge) badge.textContent = cart.length;
}

function addToCart(productId) {
  const product = products.find((item) => item.id === productId);
  if (!product) return;

  const cart = getCart();
  cart.push(product);
  saveCart(cart);
  updateCartBadge();
  alert(`${product.name} added to cart.`);
}

function removeFromCart(productId) {
  const cart = getCart().filter((item) => item.id !== productId);
  saveCart(cart);
  renderCart();
  updateCartBadge();
}

function renderProducts() {
  const grid = document.getElementById("product-grid");
  if (!grid) return;

  const buttons = document.querySelectorAll(".filter-btn");
  const currentFilter = document.querySelector(".filter-btn.active")?.dataset.filter || "all";

  const filteredProducts = currentFilter === "all"
    ? products
    : products.filter((item) => item.category === currentFilter);

  grid.innerHTML = filteredProducts
    .map(
      (product) => `
        <article class="product-card">
          <div class="product-image ${product.imageClass}"></div>
          <div class="product-body">
            <span class="tag">${product.tag}</span>
            <h3>${product.name}</h3>
            <div class="rating">★★★★★ <span>4.9</span></div>
            <div class="price-row">
              <strong>$${product.price}</strong>
              <button class="add-cart" data-id="${product.id}">Add to cart</button>
            </div>
          </div>
        </article>
      `
    )
    .join("");

  attachProductButtons();
}

function attachProductButtons() {
  document.querySelectorAll(".add-cart").forEach((button) => {
    button.addEventListener("click", () => addToCart(button.dataset.id));
  });

  document.querySelectorAll(".filter-btn").forEach((button) => {
    button.addEventListener("click", () => {
      document.querySelectorAll(".filter-btn").forEach((btn) => btn.classList.remove("active"));
      button.classList.add("active");
      renderProducts();
    });
  });
}

function renderCart() {
  const cartItems = document.getElementById("cart-items");
  if (!cartItems) return;

  const cart = getCart();

  if (!cart.length) {
    cartItems.innerHTML = `
      <div class="empty-state">
        <h3>Your cart is empty</h3>
        <p>Browse the product catalog and add premium PC parts to get started.</p>
        <a href="products.html" class="btn btn-primary" style="margin-top: 1rem;">Shop now</a>
      </div>
    `;
    updateSummary([], true);
    return;
  }

  const grouped = cart.reduce((acc, item) => {
    acc[item.id] = acc[item.id] || { ...item, qty: 0 };
    acc[item.id].qty += 1;
    return acc;
  }, {});

  const array = Object.values(grouped);
  const subtotal = array.reduce((sum, item) => sum + item.price * item.qty, 0);

  cartItems.innerHTML = array
    .map(
      (item) => `
        <div class="cart-item">
          <div class="cart-thumb"></div>
          <div class="cart-item-details">
            <h3>${item.name}</h3>
            <p>Quantity: ${item.qty}</p>
          </div>
          <div class="cart-item-actions">
            <strong>$${(item.price * item.qty).toFixed(2)}</strong>
            <button class="remove-item" data-id="${item.id}">Remove</button>
          </div>
        </div>
      `
    )
    .join("");

  document.querySelectorAll(".remove-item").forEach((button) => {
    button.addEventListener("click", () => removeFromCart(button.dataset.id));
  });

  updateSummary(array, false, subtotal);
}

function updateSummary(items, isEmpty = false, subtotal = 0) {
  const summaryItems = document.getElementById("summary-items");
  const summarySubtotal = document.getElementById("summary-subtotal");
  const summaryTotal = document.getElementById("summary-total");

  if (summaryItems) summaryItems.textContent = isEmpty ? "0" : items.reduce((count, item) => count + item.qty, 0);
  if (summarySubtotal) summarySubtotal.textContent = `$${subtotal.toFixed(2)}`;
  if (summaryTotal) summaryTotal.textContent = `$${subtotal.toFixed(2)}`;
}

function renderCheckoutSummary() {
  const cart = getCart();
  const summaryContainer = document.getElementById("checkout-summary-items");
  const totalEl = document.getElementById("checkout-total");

  if (!summaryContainer || !totalEl) return;

  if (!cart.length) {
    summaryContainer.innerHTML = "<p>Your cart is empty.</p>";
    totalEl.textContent = "$0.00";
    return;
  }

  const grouped = cart.reduce((acc, item) => {
    acc[item.id] = acc[item.id] || { ...item, qty: 0 };
    acc[item.id].qty += 1;
    return acc;
  }, {});

  const items = Object.values(grouped);
  const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0);

  summaryContainer.innerHTML = items
    .map((item) => `<div>${item.name} × ${item.qty} — $${(item.price * item.qty).toFixed(2)}</div>`)
    .join("");

  totalEl.textContent = `$${subtotal.toFixed(2)}`;
}

function setupCheckoutForm() {
  const form = document.getElementById("checkout-form");
  const message = document.getElementById("checkout-message");

  if (!form) return;

  form.addEventListener("submit", (event) => {
    event.preventDefault();
    const cart = getCart();

    if (!cart.length) {
      message.textContent = "Your cart is empty.";
      message.style.color = "#ff9f9f";
      return;
    }

    const formData = new FormData(form);
    const name = formData.get("name");
    message.textContent = `Thank you, ${name}! Your premium PC parts order has been placed successfully.`;
    message.style.color = "#34d399";
    localStorage.removeItem(storageKey);
    updateCartBadge();
    form.reset();
    renderCheckoutSummary();
  });
}

document.addEventListener("DOMContentLoaded", () => {
  updateCartBadge();

  if (document.getElementById("product-grid")) {
    renderProducts();
  }

  if (document.getElementById("cart-items")) {
    renderCart();
  }

  if (document.getElementById("checkout-summary-items")) {
    renderCheckoutSummary();
  }

  setupCheckoutForm();
});
