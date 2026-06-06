# 🧱 CryptoStuds — Ecosistema de Criptomoneda

**Proyecto Final — Criptografía | Ingeniería en Ciberseguridad | UABCS**  
**Opción A: Blockchain Core desde Cero (Python)**

Integrantes: Carlitos · Emilio · Carlos · Gael

---

## ¿Qué es CryptoStuds?

CryptoStuds es una blockchain P2P simplificada implementada en Python puro, que demuestra los principios fundamentales de las criptomonedas modernas:

- **Prueba de Trabajo (PoW)** con dificultad de 4 ceros hexadecimales
- **Árbol de Merkle** para consolidar e integridad de transacciones
- **ECDSA sobre secp256k1** para firmas digitales (igual que Bitcoin)
- **Red P2P** con propagación de bloques y consenso de cadena más larga
- **Prevención de doble gasto** verificada antes de aceptar transacciones

---

## Estructura del proyecto

```
cryptostuds/
├── backend/
│   ├── block.py          # Bloque con Merkle Root y PoW
│   ├── blockchain.py     # Cadena, validación, anti doble-gasto
│   ├── transaction.py    # TX con firma ECDSA y verificación
│   ├── wallet.py         # Par de claves secp256k1
│   └── node.py           # Nodo P2P, propagación, consenso
├── api/
│   └── node_app.py       # API REST Flask (9 endpoints)
├── docs/
│   └── index.html        # Interfaz web (servida por GitHub Pages)
├── demo_final.py         # Demo completa de 8 pasos
└── README.md
```

---

## Requisitos

```bash
pip install flask flask-cors requests cryptography
```

---

## Cómo ejecutar

### 1. Levantar nodos

Abre dos terminales y ejecuta:

```bash
# Terminal 1 — Nodo A
python api/node_app.py --port 5000

# Terminal 2 — Nodo B
python api/node_app.py --port 5001
```

### 2. Abrir la interfaz web

Abre `docs/index.html` en tu navegador. Conecta al nodo en `http://localhost:5000`.

### 3. Demo automática completa

```bash
python demo_final.py
```

Ejecuta los 8 pasos de forma automática: crea wallets, envía transacciones firmadas, mina bloques, sincroniza nodos y verifica consistencia.

---

## Despliegue público (usar desde cualquier dispositivo)

El proyecto se compone de dos piezas que se publican por separado.

### Frontend (estático)

La carpeta `docs/` (el `index.html` y las imágenes) es estática. Está nombrada `docs/` para que
**GitHub Pages** la pueda servir directamente: en *Settings → Pages*, elige la rama `main` y la
carpeta `/docs`. GitHub entrega una URL pública del tipo `https://<usuario>.github.io/<repo>/`.

### Backend (nodo Flask)

`localhost:5000` solo es accesible desde la propia máquina, por eso la interfaz no funciona en
otros dispositivos hasta exponer el nodo con una URL pública.

- **Túnel (rápido, para la demo):** con el nodo corriendo localmente, abre un túnel con ngrok:

  ```bash
  ngrok http 5000
  ```

  Esto entrega una URL `https://xxxx.ngrok-free.app`. Pégala en el campo *NODO* de la web en
  lugar de `http://localhost:5000`. La API ya envía el header `ngrok-skip-browser-warning`.

- **Servicio en la nube (permanente):** despliega el nodo en una plataforma como Render o
  Railway. En producción se recomienda servir la app con un servidor WSGI (gunicorn) en lugar
  del servidor de desarrollo de Flask:

  ```bash
  gunicorn -w 1 -b 0.0.0.0:5000 api.node_app:app
  ```

> Nota de seguridad: el nodo habilita `CORS(*)` y minería abierta para facilitar la demo. En un
> despliegue real conviene restringir orígenes y autenticar los endpoints de escritura.

---

## Endpoints de la API

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/chain` | Longitud + cadena completa serializada |
| GET | `/chain/valid` | Verifica integridad (PoW + hashes) |
| GET | `/mine?miner=<addr>` | Mina bloque y propaga a peers |
| POST | `/nodes/register` | Registra peers `{ "nodes": ["http://..."] }` |
| GET | `/nodes/resolve` | Consenso: adopta la cadena más larga válida |
| POST | `/block/receive` | Recibe bloque de un peer |
| GET | `/wallet/new` | Genera par de claves secp256k1 |
| POST | `/transaction/new` | Envía TX (valida firma y saldo) |
| GET | `/balance/<address>` | Saldo confirmado de una dirección |

---

## Decisiones de diseño

### Criptografía
- **secp256k1 + ECDSA**: La misma curva elíptica que Bitcoin. Genera claves compactas y firmas verificables sin necesidad de un servidor central.
- **SHA-256**: Usado para el hash de bloques, de transacciones (tx_id) y para derivar la dirección del wallet desde la clave pública comprimida.

### Árbol de Merkle
El `merkle_root` se calcula a partir de los `tx_id` de todas las transacciones del bloque. Si se altera una sola transacción, la raíz cambia, lo que invalida el hash del bloque y rompe el encadenamiento.

### Prevención de doble gasto
Antes de agregar una TX al pool, `blockchain.add_tx()` verifica que el saldo disponible (saldo confirmado menos lo ya comprometido en TXs pendientes) sea suficiente.

### Consenso
El algoritmo de Nakamoto: ante una bifurcación, el nodo adopta la cadena más larga que supere la validación completa (encadenamiento + PoW + Merkle Root por bloque).
