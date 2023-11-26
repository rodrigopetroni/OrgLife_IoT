#  🥼 **Orglife - 2TDSR**

## 💻 Descrição do Projeto
Projeto feito para a Global Solution, com a proposta de criar os sensores da nossa clínica de órgãos. Utilizamos uma placa ESP32 para realizar o sensor, utilizamos o Node-RED para a realização do gateway e para a realização do dashboard.

## 💎 Fluxos Node-RED 
Os fluxos para o Node-RED podem ser encontrados nos arquivos acima dentro do arquivo **fluxos-nodered.jpg**

## 📝 Instruções
Para utilizar o projeto, siga às seguintes instruções:
- Utilize primeiramente uma plataforma de códigos para a execução do código, no nosso caso utilizamos o Wokwi
- Depois, dentro do Wokwi, inicie a aplicação do projeto
- Digite dentro do seu Wokwi o seguinte código:
- ```
  #include <WiFi.h>
  #include <PubSubClient.h>
  #include <DHTesp.h>

  const int DHT_PIN = 15;
  DHTesp dht; 

  // Configure com os padrões utilizados pela sua rede.

  const char* ssid = "Wokwi-GUEST";
  const char* password = "";
  const char* mqtt_server = "test.mosquitto.org";

  WiFiClient espClient;
  PubSubClient client(espClient);
  unsigned long lastMsg = 0;
  float temp = 0;
  float hum = 0;

  void setup_wifi() { //perintah koneksi wifi
    delay(10);
    // Começamos conectando com uma rede Wi-Fi
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.mode(WIFI_STA); //setting wifi chip sebagai station/client
    WiFi.begin(ssid, password); //koneksi ke jaringan wifi

    while (WiFi.status() != WL_CONNECTED) { //perintah tunggu esp32 sampi terkoneksi ke wifi
      delay(500);
      Serial.print(".");
    }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  }

  void callback(char* topic, byte* payload, unsigned int length) { //perintah untuk menampilkan data ketika esp32 di setting sebagai subscriber
    Serial.print("Message arrived [");
    Serial.print(topic);
    Serial.print("] ");
  for (int i = 0; i < length; i++) { //mengecek jumlah data yang ada di topik mqtt
      Serial.print((char)payload[i]);
    }
    Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(2, LOW);   // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(2, HIGH);  // Turn the LED off by making the voltage HIGH
  }
  }

  void reconnect() { //perintah koneksi esp32 ke mqtt broker baik itu sebagai publusher atau subscriber
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // perintah membuat client id agar mqtt broker mengenali board yang kita gunakan
    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("Connected");
      // Once connected, publish an announcement...
      client.publish("/indobot/p/mqtt", "Indobot"); //perintah publish data ke alamat topik yang di setting
      // ... and resubscribe
      client.subscribe("/indobot/p/mqtt"); //perintah subscribe data ke mqtt broker
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
  }

  void setup() {
    pinMode(2, OUTPUT);     // inisialisasi pin 2 / ledbuiltin sebagai output
    Serial.begin(115200);
    setup_wifi(); //memanggil void setup_wifi untuk dieksekusi
    client.setServer(mqtt_server, 1883); //perintah connecting / koneksi awal ke broker
    client.setCallback(callback); //perintah menghubungkan ke mqtt broker untuk subscribe data
    dht.setup(DHT_PIN, DHTesp::DHT22);//inisialiasi komunikasi dengan sensor dht22
  }

  void loop() {
    if (!client.connected()) {
      reconnect();
    }
    client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) { //perintah publish data
    lastMsg = now;
    TempAndHumidity  data = dht.getTempAndHumidity();

    String temp = String(data.temperature, 2); //membuat variabel temp untuk di publish ke broker mqtt
    client.publish("/indobot/p/temp", temp.c_str()); //publish data dari varibel temp ke broker mqtt
    
    String hum = String(data.humidity, 1); //membuat variabel hum untuk di publish ke broker mqtt
    client.publish("/indobot/p/hum", hum.c_str()); //publish data dari varibel hum ke broker mqtt

    Serial.print("Temperature: ");
    Serial.println(temp);
    Serial.print("Humidity: ");
    Serial.println(hum);
  }
  }
 

- Em seguida, abra o Node-RED e copie os fluxos que estarão disponíveis no arquivo **fluxos.nodered.jpg**
- E por último, insira dentro do Node-RED o seguinte JSON:
```js
   [
    {
        "id": "2f5404c550b1c057",
        "type": "ui_gauge",
        "z": "530b5f4a309c5367",
        "name": "",
        "group": "77aa337cec4d48f3",
        "order": 1,
        "width": 0,
        "height": 0,
        "gtype": "gage",
        "title": "Umidade",
        "label": "%",
        "format": "{{value}}",
        "min": 0,
        "max": "100",
        "colors": [
            "#00b500",
            "#e6e600",
            "#ca3838"
        ],
        "seg1": "",
        "seg2": "",
        "diff": false,
        "className": "",
        "x": 520,
        "y": 360,
        "wires": []
    },
    {
        "id": "77aa337cec4d48f3",
        "type": "ui_group",
        "name": "dados hora e cartao",
        "tab": "b65ecaa6413acd47",
        "order": 1,
        "disp": false,
        "width": "6",
        "collapse": false,
        "className": ""
    },
    {
        "id": "b65ecaa6413acd47",
        "type": "ui_tab",
        "name": "Home",
        "icon": "dashboard",
        "disabled": false,
        "hidden": false
    }
]
```

## 🦺 Colaboradores
Segue abaixo nome dos integrantes que auxiliaram na realização do projeto:
* Rodrigo Costa Petroni / RM: 93872
* Gustavo Ribeiro Maio / RM: 93211
* Lucas Antunes dos Reis Silva / RM: 95781
* Nickolas Kenji Takeda Maldonado / RM: 95281
* Kauan Altino Gianesini Grilo / RM: 94700

## 🛠️ Tecnologias
Utilizamos para realizar os sensores as seguintes tecnologias:
* Arduino
* Plataforma Wokwi
* Node-RED
* Placa ESP32
* Protocolo MQTT
* Placa para sensor de umidade e temperatura DHT22

## 🎬 Vídeo
Link para o vídeo referente ao pitch do projeto com explicação dos temas: 
* https://youtu.be/arG7gNc4fGY

