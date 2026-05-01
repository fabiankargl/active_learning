# Label Studio API Setup & Nutzung (mit Google Colab & ngrok)

## Überblick

In diesem Guide beschreibe ich, wie ich **Label Studio lokal installiert**, über **ngrok zugänglich gemacht** und anschließend über die **API in Google Colab** verwendet habe.

---

## 1. Installation von Label Studio

Zuerst habe ich Label Studio über `pip` installiert, sodass die Anwendung lokal läuft:

```bash
pip install label-studio
```

Starten kann man Label Studio anschließend mit:

```bash
label-studio start
```

Die Anwendung läuft dann standardmäßig unter:

```
http://localhost:8080/
```

> Hinweis: Alternativ könnte man auch eine bereits gehostete Instanz (z. B. von der FH) verwenden, das habe ich jedoch nicht getestet.

---

## 2. Zugriff aus Google Colab mit ngrok

Da Google Colab keinen Zugriff auf den lokalen `localhost` hat, habe ich **ngrok** verwendet, um einen Tunnel zu erstellen.

### Installation von ngrok

```bash
pip install ngrok
```

Danach muss man sich bei ngrok registrieren (z. B. mit Google Account):
https://dashboard.ngrok.com/get-started/setup/windows

Dort findet man den **AuthToken**, den man im Terminal setzt:

```bash
ngrok config add-authtoken YOUR_AUTHTOKEN
```

### ngrok starten

```bash
ngrok http 8080
```

Anschließend erhält man eine **öffentliche HTTPS-URL** unter *Forwarding*, z. B.:

```
https://xxxxx.ngrok-free.app
```

Diese URL wird später in Colab verwendet, um auf Label Studio zuzugreifen.

---

## 3. Label Studio API Token erstellen

Den API Token kann man direkt im lokal laufenden Label Studio generieren und kopieren.

---

## 4. Nutzung in Google Colab

In Colab habe ich den API Key sicher über **Secrets** gespeichert und so darauf zugegriffen:

```python
from google.colab import userdata

LABEL_STUDIO_URL = "YOUR_NGROK_FORWARDING"
LABEL_STUDIO_API_KEY = userdata.get("secretName")
```

### Verbindung zur API herstellen

```python
from label_studio_sdk import LabelStudio

client = LabelStudio(
    base_url=LABEL_STUDIO_URL,
    api_key=LABEL_STUDIO_API_KEY
)

me = client.users.whoami()
print(f"Connected as: {me.username} ({me.email})")
```

---

## 5. Projekt erstellen

Ein Projekt kann entweder über die UI oder direkt per Code erstellt werden.

### Beispiel: MNIST Labeling Projekt

```python
LABEL_CONFIG = """
<View>
  <Image name="image" value="$image"/>
  <Choices name="digit" toName="image" choice="single">
    <Choice value="0"/><Choice value="1"/><Choice value="2"/>
    <Choice value="3"/><Choice value="4"/><Choice value="5"/>
    <Choice value="6"/><Choice value="7"/><Choice value="8"/>
    <Choice value="9"/>
  </Choices>
</View>
"""

project = client.projects.create(
    title="MNIST Active Learning",
    label_config=LABEL_CONFIG,
)

print(f"Project created: ID={project.id}")
```

---

## 6. Setup auf macOS

Für macOS unterscheiden sich die Installationsbefehle leicht:

```bash
brew install ngrok
ngrok config add-authtoken AUTH_TOKEN

brew install humansignal/tap/label-studio
label-studio
```

---
