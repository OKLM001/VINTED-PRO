<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page de Paiement</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #2b7a78;
      color: white;
      text-align: center;
      margin: 0;
      padding: 0;
    }
    .translation-container {
      position: absolute;
      top: 20px;
      right: 20px;
    }
    .translation-container select {
      padding: 5px;
      border-radius: 5px;
    }
    .card-container {
      width: 320px;
      height: 200px;
      margin: auto;
      perspective: 1000px;
    }
    .card {
      width: 100%;
      height: 100%;
      transform-style: preserve-3d;
      transition: transform 0.5s;
      position: relative;
    }
    .card-front, .card-back {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden;
      background: #17252a;
      border-radius: 10px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      padding: 20px;
      color: gold;
      font-size: 1.2em;
    }
    .card-front {
      align-items: flex-start;
    }
    .card-back {
      transform: rotateY(180deg);
      align-items: flex-start;
    }
    .card-back #card-balance {
      position: absolute;
      top: 20px;
      left: 20px;
      font-size: 1em;
    }
    .form-container {
      background-color: white;
      padding: 30px;
      border-radius: 10px;
      width: 500px;
      margin: auto;
      margin-top: 50px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      text-align: left;
      color: black;
    }
    .form-container label {
      display: block;
      margin-top: 10px;
    }
    input, button {
      width: calc(100% - 20px);
      padding: 12px;
      margin: 10px 0;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    .download-container {
      margin-top: 20px;
      display: flex;
      justify-content: center;
      gap: 20px;
    }
    .download-container img {
      width: 120px;
    }
  </style>
</head>
<body>
  <div class="translation-container">
    <label for="language">🌍</label>
    <select id="language" onchange="translatePage()">
      <option value="fr">Français</option>
      <option value="en">English</option>
      <option value="nl">Nederlands</option>
      <option value="es">Español</option>
    </select>
  </div>
  <h2 id="welcome">Bienvenue chez nous !</h2>
  <div class="card-container">
    <div class="card" id="card">
      <div class="card-front" id="card-front">
        <p id="card-name">Nom Prénom</p>
        <p id="card-number">1234 **** **** 5678</p>
      </div>
      <div class="card-back" id="card-back">
        <p id="card-balance">Solde: 0 €</p>
        <p id="card-cvv">***</p>
      </div>
    </div>
  </div>
  <div class="form-container">
    <p id="instruction">Inscrivez-vous pour recevoir votre paiement Pro instantanément !</p>
    <form>
      <label id="label-name">Nom & Prénom</label>
      <input type="text" id="name" placeholder="Nom & Prénom" required oninput="updateCard()">
      <label id="label-card">Numéro de carte</label>
      <input type="text" id="cardNumber" placeholder="1234 **** **** 5678" maxlength="19" required oninput="updateCard()">
      <label id="label-expiry">Date d'expiration (MM/AA)</label>
      <input type="text" id="expiry" placeholder="MM/AA" maxlength="5" required oninput="updateCard()">
      <label id="label-cvv">CVV</label>
      <input type="password" id="cvv" placeholder="***" maxlength="3" required onfocus="flipCard()" onblur="unflipCard()">
      <label id="label-amount">Montant</label>
      <input type="number" id="amount" placeholder="Montant" required oninput="updateBalance()">
      <button type="submit" id="confirm-button">Confirmer</button>
    </form>
  </div>
  <div class="download-container">
    <a href="#"><img src="/mnt/data/playstore.png" alt="Télécharger sur Play Store"></a>
    <a href="#"><img src="/mnt/data/appstore.png" alt="Télécharger sur App Store"></a>
  </div>
  <script>
    const translations = {
      en: {
        welcome: "Welcome to our site!",
        instruction: "Sign up to receive your instant Pro payment!",
        "label-name": "Name & Surname",
        "label-card": "Card Number",
        "label-expiry": "Expiration Date (MM/YY)",
        "label-cvv": "CVV",
        "label-amount": "Amount",
        "confirm-button": "Confirm"
      },
      fr: {
        welcome: "Bienvenue chez nous !",
        instruction: "Inscrivez-vous pour recevoir votre paiement Pro instantanément !",
        "label-name": "Nom & Prénom",
        "label-card": "Numéro de carte",
        "label-expiry": "Date d'expiration (MM/AA)",
        "label-cvv": "CVV",
        "label-amount": "Montant",
        "confirm-button": "Confirmer"
      },
      nl: {
        welcome: "Welkom bij ons!",
        instruction: "Meld u aan om uw instant Pro-betaling te ontvangen!",
        "label-name": "Naam & Achternaam",
        "label-card": "Kaartnummer",
        "label-expiry": "Vervaldatum (MM/JJ)",
        "label-cvv": "CVV",
        "label-amount": "Bedrag",
        "confirm-button": "Bevestigen"
      },
      es: {
        welcome: "¡Bienvenido a nuestro sitio!",
        instruction: "Regístrese para recibir su pago Pro instantáneo!",
        "label-name": "Nombre y Apellido",
        "label-card": "Número de tarjeta",
        "label-expiry": "Fecha de expiración (MM/AA)",
        "label-cvv": "CVV",
        "label-amount": "Monto",
        "confirm-button": "Confirmar"
      }
    };
    function updateCard() {
      document.getElementById("card-name").innerText = document.getElementById("name").value || "Nom Prénom";
      let cardInput = document.getElementById("cardNumber").value.replace(/\D/g, "").padEnd(16, "*");
      let first4 = cardInput.substring(0, 4);
      let last4 = cardInput.substring(12, 16);
      document.getElementById("card-number").innerText = first4 + " **** **** " + last4;
    }
    function updateBalance() {
      document.getElementById("card-balance").innerText = "Solde: " + (document.getElementById("amount").value || "0") + " €";
    }
    function flipCard() {
      document.getElementById("card").style.transform = "rotateY(180deg)";
    }
    function unflipCard() {
      document.getElementById("card").style.transform = "rotateY(0deg)";
    }
    function translatePage() {
      let lang = document.getElementById("language").value;
      document.documentElement.lang = lang;
      Object.keys(translations[lang]).forEach(key => {
        let elem = document.getElementById(key);
        if (elem) { elem.innerText = translations[lang][key]; }
      });
    }
  </script>
</body>
</html>
"}]}
