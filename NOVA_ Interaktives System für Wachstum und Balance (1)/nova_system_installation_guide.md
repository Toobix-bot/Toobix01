# Detaillierte Installationsanleitung für das NOVA System

Diese Anleitung führt Sie durch die Installation und Konfiguration des NOVA Systems, einem interaktiven Universum für persönliche Entwicklung mit Gamification-Elementen.

## Voraussetzungen

Bevor Sie beginnen, stellen Sie sicher, dass folgende Software installiert ist:

- Node.js (Version 14 oder höher)
- npm (normalerweise mit Node.js installiert)
- MongoDB (Version 4.4 oder höher)

## Schritt 1: Projektdateien extrahieren

1. Laden Sie die NOVA System-Dateien herunter oder klonen Sie das Repository
2. Extrahieren Sie die Dateien in ein Verzeichnis Ihrer Wahl

```bash
mkdir -p nova_system
unzip nova_system_prototype.zip -d nova_system
cd nova_system
```

## Schritt 2: Abhängigkeiten installieren

Installieren Sie alle erforderlichen npm-Pakete:

```bash
npm install
```

## Schritt 3: MongoDB installieren und konfigurieren

Wenn MongoDB noch nicht installiert ist, folgen Sie diesen Schritten:

### Für Ubuntu/Debian:

```bash
# MongoDB-Repository hinzufügen
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Pakete aktualisieren und MongoDB installieren
sudo apt-get update
sudo apt-get install -y mongodb-org

# Datenverzeichnis erstellen und Berechtigungen setzen
sudo mkdir -p /data/db
sudo chown -R mongodb:mongodb /data/db

# MongoDB starten
sudo mongod --dbpath /data/db --fork --logpath /var/log/mongodb.log
```

### Für andere Betriebssysteme:

Besuchen Sie die [offizielle MongoDB-Installationsanleitung](https://docs.mongodb.com/manual/installation/).

## Schritt 4: Umgebungsvariablen konfigurieren

Erstellen Sie eine `.env`-Datei im Hauptverzeichnis des Projekts:

```bash
touch .env
```

Fügen Sie folgende Konfiguration in die `.env`-Datei ein:

```
MONGO_URI=mongodb://localhost:27017/nova-system
JWT_SECRET=nova_secure_secret_key_for_development
PORT=5000
```

## Schritt 5: Fehlende Dateien erstellen

Erstellen Sie die Datei `src/routes/users.js` mit folgendem Inhalt:

```javascript
const express = require('express');
const router = express.Router();
const { protect } = require('../middleware/authMiddleware');
const User = require('../models/User');

// @desc    Get all users (admin only)
// @route   GET /api/users
// @access  Private
router.get('/', protect, async (req, res) => {
  try {
    // In a production app, this would be restricted to admin users
    const users = await User.find({}).select('-password');
    res.json(users);
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

// @desc    Get user by ID
// @route   GET /api/users/:id
// @access  Private
router.get('/:id', protect, async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (user) {
      res.json(user);
    } else {
      res.status(404).json({ message: 'User not found' });
    }
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});

module.exports = router;
```

## Schritt 6: Server-Konfiguration anpassen

Bearbeiten Sie die Datei `src/server.js` und ändern Sie die `app.listen`-Zeile, damit der Server auf allen Netzwerkschnittstellen hört:

```javascript
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Schritt 7: Frontend-Konfiguration anpassen

Bearbeiten Sie die Datei `pages/_app.js`, um die korrekte API-Basis-URL zu setzen:

```javascript
import '../styles/globals.css';
import { AuthProvider } from '../components/AuthContext';
import axios from 'axios';

// Set base URL for API requests - Passen Sie diese URL an Ihre Umgebung an
axios.defaults.baseURL = 'http://localhost:5000';

function MyApp({ Component, pageProps }) {
  return (
    <AuthProvider>
      <Component {...pageProps} />
    </AuthProvider>
  );
}

export default MyApp;
```

**Wichtig:** Wenn Sie das System in einer Netzwerkumgebung oder über das Internet zugänglich machen, ersetzen Sie `localhost:5000` durch die entsprechende öffentliche URL oder IP-Adresse.

## Schritt 8: Backend-Server starten

Starten Sie den Backend-Server:

```bash
node src/server.js
```

Sie sollten folgende Ausgabe sehen:
```
Server running on port 5000
MongoDB Connected: localhost
```

## Schritt 9: Frontend-Entwicklungsserver starten

Öffnen Sie ein neues Terminal-Fenster und starten Sie den Frontend-Entwicklungsserver:

```bash
npm run dev
```

Sie sollten folgende Ausgabe sehen:
```
> nova-system@1.0.0 dev
> next dev
  ▲ Next.js 14.x.x
  - Local:        http://localhost:3000
```

## Schritt 10: System testen

Öffnen Sie einen Webbrowser und navigieren Sie zu:

```
http://localhost:3000/register
```

Erstellen Sie ein Benutzerkonto, um zu überprüfen, ob die Installation erfolgreich war.

## Fehlerbehebung

### Problem: MongoDB startet nicht
- Überprüfen Sie, ob das Datenverzeichnis `/data/db` existiert und die richtigen Berechtigungen hat
- Prüfen Sie die MongoDB-Logs auf Fehler: `cat /var/log/mongodb.log`

### Problem: Server startet nicht
- Überprüfen Sie, ob MongoDB läuft
- Stellen Sie sicher, dass die .env-Datei korrekt konfiguriert ist
- Prüfen Sie, ob Port 5000 bereits verwendet wird: `lsof -i :5000`

### Problem: Registrierung funktioniert nicht
- Überprüfen Sie die Browser-Konsole auf Fehler
- Stellen Sie sicher, dass die API-Basis-URL in `_app.js` korrekt ist
- Überprüfen Sie, ob der Server auf allen Netzwerkschnittstellen hört (0.0.0.0)

## Nächste Schritte

Nach erfolgreicher Installation können Sie:

1. Das Dashboard erkunden
2. Skills erstellen und verwalten
3. Quests erstellen und abschließen
4. Reflexionen durchführen
5. Die NOVA-Stadt erkunden

Viel Erfolg mit Ihrem NOVA System!
