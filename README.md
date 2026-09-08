# dag_5_demo
Dagens labration 2026-09-08 där vi testar end‑to‑end IoT‑pipeline: sensor → MQTT → subscriber → adapter → REST‑API → consumer. Innehåller komplett flöde, validering, felhantering och testprotokoll för ett distribuerat mätsystem.

## **Översikt**
Det här projektet demonstrerar ett komplett IoT‑flöde från sensor → MQTT‑broker → subscriber → adapter → REST‑API → konsument.  
Syftet är att testa både funktionellt flöde och felhantering i ett distribuerat system.

Projektet består av följande komponenter:

- **sensor.exe** – genererar en mätning i JSON‑format  
- **mosquitto_pub** – publicerar mätningen via MQTT  
- **capture.py** – prenumererar på MQTT‑topic och sparar mottagen JSON  
- **bridge.exe** – validerar JSON och skickar den vidare till API:t  
- **api.exe** – REST‑API som lagrar senaste mätningen  
- **consumer.exe** – hämtar och visar senaste mätningen från API:t  

---

## **Systemarkitektur**
Flödet ser ut så här:

```
Sensor → MQTT Broker → Subscriber → Adapter → API → Consumer
```

Varje del är isolerad och kommunicerar via standardiserade protokoll (MQTT & HTTP).  
Det gör systemet robust, skalbart och lätt att felsöka.

---

## **Körning – Steg för steg**

### **1. Bygg projektet**
```
cmake -S . -B build
cmake --build build --config Debug
ctest --test-dir build -C Debug --output-on-failure
```

### **2. Starta MQTT‑broker**
```
mosquitto.exe -c mosquitto.conf -v
```

### **3. Starta API**
```
.\build\Debug\api.exe
```

### **4. Testa API‑hälsa**
```
curl.exe -i http://127.0.0.1:8085/health
```

### **5. Skapa mätning**
```
.\build\Debug\sensor.exe reading.json
```

### **6. Starta MQTT‑mottagare**
```
python capture.py "mosquitto_sub.exe" received.json
```

### **7. Publicera mätningen**
```
mosquitto_pub.exe -h 127.0.0.1 -p 1885 -t iot25/dag5/reading -f reading.json
```

### **8. Skicka till API via adapter**
```
.\build\Debug\bridge.exe received.json
```

### **9. Läs värdet via konsumenten**
```
.\build\Debug\consumer.exe
```

---

## **Resultat av huvudflödet**
- MQTT‑mottagaren tog emot JSON korrekt  
- Adaptern validerade och skickade till API  
- API svarade med **201 Created**  
- Konsumenten hämtade värdet och visade:  
  ```
  temp-01: 21.7 C
  ```

---

## **Feltester (gränsfall)**

### **1. Fel datatyp (invalid.json)**
Adapter stoppar direkt:
```
Bridge: expected sensorId:string, value:number, unit:string
```
Konsumenten visar fortfarande senaste giltiga värdet.

### **2. Fel POST direkt till API**
```
HTTP 400 Bad Request
```
API har egen validering och nekar felaktig JSON.

### **3. API avstängt**
```
Bridge: API transport failure: Connection timed out
```
Transportfel hanteras korrekt.

### **4. Fel MQTT‑topic**
Mottagaren får inget meddelande → timeout.

---

## **Tekniska fördelar**
- MQTT ger realtidsflöde och decoupling  
- API ger lagring, validering och historik  
- Adapter ger datakvalitet och skydd mot skräpdata  
- Systemet är modulärt och lätt att skala  

---

## **Sammanfattning**
Det här projektet visar ett komplett IoT‑flöde med robust felhantering, tydlig validering och korrekt kommunikation mellan alla komponenter.  
Alla funktionella och icke‑funktionella tester är genomförda och godkända.
