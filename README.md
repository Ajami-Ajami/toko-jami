<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Toko Jami</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f9f9f9;
      font-size: 16px;
    }

    header {
      background: #ff69b4;
      color: white;
      padding: 15px;
      text-align: center;
    }

    .container {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;
      padding: 15px;
    }

    .product {
      background: white;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      width: 220px;
      text-align: center;
      padding: 14px;
    }

    .product img {
      width: 100%;
      height: 230px;
      object-fit: cover;
      border-radius: 6px;
    }

    .product h4 {
      margin: 10px 0 5px;
      font-size: 18px;
    }

    .product p {
      margin: 0 0 10px;
      font-size: 16px;
    }

    button {
      padding: 6px 10px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .add-btn {
      background-color: #28a745;
      color: white;
      margin-top: 5px;
    }

    .cart-btn {
      position: fixed;
      top: 15px;
      right: 15px;
      background: #ff69b4;
      color: white;
      z-index: 11;
    }

    #notif {
      position: fixed;
      top: 60px;
      right: 20px;
      background: #28a745;
      color: white;
      padding: 10px;
      border-radius: 6px;
      display: none;
      z-index: 10;
    }

    #cart-panel {
      position: fixed;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      width: 80%;
      max-width: 400px;
      height: 100%;
      background: white;
      padding: 15px;
      display: none;
      z-index: 20;
      overflow-y: auto;
      box-shadow: 0 0 20px rgba(0,0,0,0.2);
    }

    .cart-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
      gap: 8px;
    }

    .remove-btn {
      background: #dc3545;
      color: white;
      font-size: 12px;
      padding: 3px 6px;
    }

    form input, form textarea {
      width: 100%;
      margin-bottom: 10px;
      padding: 6px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 14px;
    }

    .checkout-btn {
      background: #007bff;
      color: white;
      margin-top: 8px;
      font-size: 14px;
      padding: 6px 12px;
    }

    .back-btn {
      background: #555;
      color: white;
      margin-bottom: 10px;
      font-size: 14px;
      padding: 6px 12px;
    }

    @media (max-width: 500px) {
      .product {
        width: 95%;
      }
      #cart-panel {
        width: 90%;
      }
    }
  </style>
</head>
<body>
  <header>
    <h1>Toko Jami</h1>
  </header>

  <div id="notif"></div>

  <button class="cart-btn" onclick="showCart()">🛒 Keranjang</button>

  <div class="container" id="product-container"></div>

  <div id="cart-panel">
    <button class="back-btn" onclick="hideCart()">← Kembali</button>
    <h3>Keranjang Belanja</h3>
    <div id="cart-items"></div>
    <p><strong>Total: Rp <span id="cart-total">0</span></strong></p>
    <h4>Checkout</h4>
    <form id="checkout-form" onsubmit="checkoutWA(event)">
      <input type="text" id="nama" placeholder="Nama Lengkap" required />
      <textarea id="alamat" placeholder="Alamat Lengkap" required></textarea>
      <p><strong>Metode Pembayaran:</strong> COD</p>
      <button class="checkout-btn" type="submit">Checkout via WhatsApp</button>
    </form>
  </div>

  <script>
    const produk = [
      { nama: "Pensil", harga: 2000, gambar: "https://i.ibb.co/30D6jXZ/pensil.jpg" },
      { nama: "Pulpen", harga: 6000, gambar: "https://i.ibb.co/ZGcYHBD/pulpen.jpg" },
      { nama: "Penghapus", harga: 1000, gambar: "https://i.ibb.co/xgKcqfG/penghapus.jpg" },
      { nama: "Correction Tip X", harga: 15000, gambar: "https://i.ibb.co/z5XhNph/tipx.jpg" },
      { nama: "Penggaris", harga: 5000, gambar: "https://i.ibb.co/3mvnYqh/penggaris.jpg" },
      { nama: "Rautan", harga: 3000, gambar: "https://i.ibb.co/nLqH4Dg/rautan.jpg" },
      { nama: "Sticky Note 100 Lembar", harga: 5000, gambar: "https://i.ibb.co/zPbnT3K/stickynote.jpg" }
    ];

    let keranjang = [];

    function renderProduk() {
      const container = document.getElementById("product-container");
      container.innerHTML = "";
      produk.forEach((p, i) => {
        container.innerHTML += `
          <div class="product">
            <img src="${p.gambar}" alt="${p.nama}">
            <h4>${p.nama}</h4>
            <p>Rp ${p.harga.toLocaleString()}</p>
            <button class="add-btn" onclick="tambahKeKeranjang(${i})">+ Keranjang</button>
          </div>
        `;
      });
    }

    function tambahKeKeranjang(i) {
      const item = produk[i];
      const index = keranjang.findIndex(k => k.nama === item.nama);
      if (index > -1) {
        keranjang[index].jumlah++;
      } else {
        keranjang.push({ ...item, jumlah: 1 });
      }
      tampilNotif(`${item.nama} ditambahkan ke keranjang`);
    }

    function tampilNotif(text) {
      const notif = document.getElementById("notif");
      notif.innerText = text;
      notif.style.display = "block";
      setTimeout(() => notif.style.display = "none", 2000);
    }

    function showCart() {
      document.getElementById("cart-panel").style.display = "block";
      renderKeranjang();
    }

    function hideCart() {
      document.getElementById("cart-panel").style.display = "none";
    }

    function renderKeranjang() {
      const container = document.getElementById("cart-items");
      const totalEl = document.getElementById("cart-total");
      container.innerHTML = "";
      let total = 0;

      keranjang.forEach((item, i) => {
        total += item.harga * item.jumlah;
        container.innerHTML += `
          <div class="cart-item">
            <span>${item.nama} (${item.jumlah}x)</span>
            <span>Rp ${(item.harga * item.jumlah).toLocaleString()}</span>
            <button class="remove-btn" onclick="hapusItem(${i})">Hapus</button>
          </div>
        `;
      });

      totalEl.innerText = total.toLocaleString();
    }

    function hapusItem(i) {
      keranjang.splice(i, 1);
      renderKeranjang();
    }

    function checkoutWA(e) {
      e.preventDefault();
      const nama = document.getElementById("nama").value;
      const alamat = document.getElementById("alamat").value;
      if (keranjang.length === 0) return alert("Keranjang kosong!");

      let pesan = `Halo, saya ${nama} ingin memesan:\n\n`;
      let total = 0;
      keranjang.forEach(item => {
        pesan += `- ${item.nama} (${item.jumlah}x) - Rp ${(item.harga * item.jumlah).toLocaleString()}\n`;
        total += item.harga * item.jumlah;
      });
      pesan += `\nAlamat: ${alamat}\nMetode: COD\nTotal: Rp ${total.toLocaleString()}`;
      const url = `https://wa.me/6285212690525?text=${encodeURIComponent(pesan)}`;
      window.open(url, "_blank");
    }

    renderProduk();
  </script>
</body>
</html>
