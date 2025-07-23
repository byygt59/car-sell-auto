<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <title>Car Sell Auto</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f4f4; margin: 0; padding: 0; }
    header { background: #333; color: white; padding: 10px; text-align: center; }
    #game { max-width: 600px; margin: 20px auto; padding: 10px; background: white; border-radius: 8px; }
    .bakiye { font-size: 1.2em; margin-bottom: 10px; }
    .buton { display: block; width: 100%; padding: 10px; margin: 5px 0; font-size: 1em; border: none; border-radius: 5px; cursor: pointer; }
    .buy { background: #28a745; color: white; }
    .repair { background: #17a2b8; color: white; }
    .sell { background: #ffc107; color: #333; }
    .night { background: #6f42c1; color: white; }
    .customer { background: #007bff; color: white; }
    .log { margin-top: 15px; padding: 10px; background: #fafafa; height: 200px; overflow-y: auto; border: 1px solid #ddd; border-radius: 5px; }
    .list { margin: 15px 0; max-height: 150px; overflow-y: auto; }
    li { margin: 5px 0; }
  </style>
</head>
<body>
  <header><h1>Car Sell Auto</h1></header>
  <div id="game">
    <div class="bakiye">💰 Bakiye: <span id="bakiye">200000</span> ₺</div>
    <button class="buton buy" onclick="showCars()">1. Araba Satın Al</button>
    <button class="buton repair" onclick="repairCar()">2. Araba Tamir Et</button>
    <button class="buton sell" onclick="sellCar()">3. Araba Sat</button>
    <button class="buton night" onclick="nightSale()">4. Gece Kaçak Satış</button>
    <button class="buton customer" onclick="serveCustomers()">5. Günlük Müşteriler</button>

    <div id="details">
      <ul class="list" id="listArea"></ul>
    </div>
    <div class="log" id="logArea"></div>
  </div>

  <script>
    const tumArabalar = [
      "BMW F10","Mercedes E60","Honda S2000","Mercedes C250d","Renault Megane 2",
      "Toyota Supra","Nissan 350Z","Audi A4","Volkswagen Golf GTI","Ford Mustang",
      "Chevrolet Camaro","Dodge Charger","Tesla Model 3","Hyundai i20","Opel Astra",
      "Peugeot 308","Skoda Superb","Fiat Egea","Volvo S60","BMW M5","Lexus IS300",
      "Mazda RX-8","Porsche Cayman","Mini Cooper S","Citroen C5"
    ];
    const renkler = ["Kırmızı","Siyah","Gri","Beyaz","Mavi"];
    let bakiye = 200000, envanter = [], arabalar = [];

    class Araba {
      constructor(model) {
        this.model = model;
        this.fiyat = randInt(60000,180000);
        this.bozukluk = randInt(0,2);
        this.km = randInt(50000,200000);
        this.yil = randInt(2005,2022);
        this.renk = renkler[randInt(0, renkler.length-1)];
      }
      tamirMaliyeti() { return this.bozukluk===0 ? 0 : this.fiyat*(this.bozukluk===1?0.1:0.25); }
      satFiyati(){ return this.fiyat*(this.bozukluk===0?1.6:1.3); }
      detay(){ return ${this.model} | ${this.renk} | ${this.yil} | ${this.km}km | Durum: ${["Sağlam","Az Bozuk","Çok Bozuk"][this.bozukluk]} | Fiyat: ${Math.floor(this.fiyat)}₺; }
    }

    function randInt(min,max){ return Math.floor(Math.random()*(max-min+1))+min; }
    function log(msg){ const a=document.getElementById("logArea"); a.innerHTML+= msg+"<br>"; a.scrollTop = a.scrollHeight; }
    function updateBalance(){ document.getElementById("bakiye").innerText = Math.floor(bakiye); }

    function fillCars(){
      arabalar = [];
      for(let i=0; i<5; i++) arabalar.push(new Araba(tumArabalar[randInt(0,tumArabalar.length-1)]));
    }

    fillCars(); updateBalance();

    function showCars(){
      const ul = document.getElementById("listArea");
      ul.innerHTML = "";
      arabalar.forEach((c,i)=>{
        const li = document.createElement("li");
        li.textContent = ${i+1}. ${c.detay()};
        li.onclick = ()=>{
          if(bakiye>=c.fiyat){
            bakiye-=c.fiyat; envanter.push(c);
            log(✅ ${c.model} satın alındı!);
            arabalar.splice(i,1);
            arabalar.push(new Araba(tumArabalar[randInt(0,tumArabalar.length-1)]));
            updateBalance(); ul.innerHTML="";
          }else log("❌ Yetersiz bakiye!");
        };
        ul.appendChild(li);
      });
      log("Arabaları seçmek için listeye dokun.");
    }

    function repairCar(){
      const ul = document.getElementById("listArea");
      ul.innerHTML="";
      envanter.forEach((c,i)=>{
        const cost = c.tamirMaliyeti();
        const li = document.createElement("li");
        li.textContent = ${i+1}. ${c.detay()} | Tamir Bedeli: ${Math.floor(cost)}₺;
        li.onclick = ()=>{
          if(c.bozukluk===0){ log("✅ Bu araç zaten sağlam."); }
          else if(bakiye>=cost){
            bakiye-=cost; c.bozukluk=0;
            log(🛠 ${c.model} tamir edildi!);
            updateBalance(); ul.innerHTML="";
          } else log("❌ Tamir için para yok!");
        };
        ul.appendChild(li);
      });
      log("Tamir etmek için envanterden seç.");
    }

    function sellCar(){
      const ul=document.getElementById("listArea"); ul.innerHTML="";
      envanter.forEach((c,i)=>{
        const sellPrice = c.satFiyati();
        const li = document.createElement("li");
        li.textContent = ${i+1}. ${c.detay()} | Satış Fiyatı: ${Math.floor(sellPrice)}₺;
        li.onclick = ()=>{
          const price=Math.floor(c.satFiyati());
          envanter.splice(i,1); bakiye+=price; updateBalance();
          log(💵 ${c.model} satıldı! Kazanç: ${price}₺);
          if(randInt(1,100)<=15){
            log("🚨 Polis kontrolü!");
            if(c.bozukluk>0){
              const ceza=randInt(10000,30000); bakiye-=ceza; updateBalance();
              log(❌ Eksik belgeler nedeniyle ceza: ${ceza}₺);
            } else log("✅ Sorunsuz geçti.");
          }
          ul.innerHTML="";
        };
        ul.appendChild(li);
      });
      log("Satmak için envanterden bir araç seç.");
    }

    function nightSale(){
      const ul=document.getElementById("listArea"); ul.innerHTML="";
      envanter.forEach((c,i)=>{
        const nightPrice = c.satFiyati()*2;
        const li = document.createElement("li");
        li.textContent = ${i+1}. ${c.detay()} | Kaçak Satış Fiyatı: ${Math.floor(nightPrice)}₺;
        li.onclick = ()=>{
          const risk=randInt(1,100);
          if(risk<=30){
            const ceza=Math.floor(c.fiyat/2); bakiye-=ceza; updateBalance();
            envanter.splice(i,1);
            log(🚨 Kaçak yaparken yakalandın! Ceza: ${ceza}₺);
          } else {
            const gelir=Math.floor(c.satFiyati()*2);
            envanter.splice(i,1); bakiye+=gelir; updateBalance();
            log(✅ Kaçak satışta ${gelir}₺ kazandın!);
          }
          ul.innerHTML="";
        };
        ul.appendChild(li);
      });
      log("Kaçak satış için bir araç seç.");
    }

    function serveCustomers(){
      const ul=document.getElementById("listArea"); ul.innerHTML="";
      for(let i=0; i<3; i++){
        const aranan = tumArabalar[randInt(0,tumArabalar.length-1)];
        const butce = randInt(60000,200000);
        log(👤 Müşteri ${i+1}: ${aranan} istiyor, bütçe: ${butce}₺);
        let satildi=false;
        envanter.forEach((c,j)=>{
          if(!satildi && c.model===aranan && c.satFiyati()<=butce){
            const fiyat=Math.floor(c.satFiyati());
            envanter.splice(j,1); bakiye+=fiyat; updateBalance();
            log(✅ ${c.model} satıldı! Kazanç: ${fiyat}₺);
            satildi=true;
          }
        });
        if(!satildi) log("❌ Uygun araç bulunamadı.");
      }
      ul.innerHTML="";
    }
  </script>
</body>
</html>
