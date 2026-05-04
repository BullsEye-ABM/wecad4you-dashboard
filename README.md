# Dashboard SDR · weCAD4you

Dashboard ejecutivo de actividad SDR conectado a HubSpot en vivo (caché 15 min).

## Filtros disponibles

- **Periodo** (9 opciones): Esta semana, Semana pasada, Este mes, Mes pasado, Este trimestre, Trimestre pasado, Este semestre, Este año, Año pasado.
- **Usuario (SDR)**: cualquier owner activo de HubSpot, mostrado como `Nombre · ID`.
- **Refresco**: botón "🔄 Forzar actualización" en sidebar para limpiar caché manualmente.

## Métricas

- KPIs: llamadas totales, conectadas, tasa de conexión, duración promedio, contactos y empresas del periodo (más totales históricos).
- Gráficos: barras apiladas por día, área por semana ISO, donut por status, % conexión por día.
- Tablas: detalle de llamadas, transcripciones/notas de las conectadas, contactos y empresas creados.

---

## 1) Crear el HubSpot Private App Token

Hazlo una sola vez en el portal de **weCAD4you (6257770)**:

1. Ve a HubSpot → ⚙️ **Settings** → **Integrations** → **Private Apps**.
2. Haz clic en **Create a private app**.
3. Tab **Basic Info**: nombre `weCAD4you SDR Dashboard`.
4. Tab **Scopes** → **CRM** → marca solo lo necesario (lectura):
   - `crm.objects.contacts.read`
   - `crm.objects.companies.read`
   - `crm.objects.deals.read` *(opcional, para extender luego)*
   - `crm.objects.owners.read`
   - `crm.objects.calls.read` *(suele estar bajo `sales-email-read` o `engagements`; busca "calls")*
   - `engagements` (lectura) *(si tu portal lo expone como single scope)*
5. Click **Create app** → copia el token que empieza con `pat-na1-...`.

> ⚠️ El token es secreto. No lo pegues en chats ni en GitHub.

## 2) Configurar el token localmente

```bash
cd ~/Desktop/weCAD4you-Dashboard
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
# Edita el archivo y pega tu token real
```

## 3) Correr en local

```bash
cd ~/Desktop/weCAD4you-Dashboard
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```

Se abre en `http://localhost:8501`. Cierra con Ctrl+C.

## 4) Deploy a Streamlit Cloud (recomendado)

Te da una URL pública, **siempre online**, gratis.

### a. Subir el código a GitHub

```bash
cd ~/Desktop/weCAD4you-Dashboard
git init
git add .
git commit -m "weCAD4you SDR dashboard"
# Crea un repo privado en github.com (Sin README ni gitignore)
git remote add origin git@github.com:<tu-usuario>/wecad4you-dashboard.git
git push -u origin main
```

### b. Conectar Streamlit Cloud

1. Ve a https://share.streamlit.io e ingresa con tu cuenta GitHub.
2. Click **New app** → selecciona el repo y rama `main` → main file `app.py`.
3. **Advanced settings → Secrets** → pega:

   ```toml
   [hubspot]
   private_app_token = "pat-na1-xxxxxxxxxxxx"
   ```

4. Deploy. La app queda en `https://<tu-usuario>-wecad4you-dashboard.streamlit.app`.

### c. Frescura de datos

- La caché se invalida cada **15 minutos**, por lo que cualquier persona que abra la URL ve datos casi en tiempo real.
- Streamlit Cloud no apaga la app si recibe tráfico regular. Si lleva días sin uso, "duerme" y se despierta en ~10s al primer hit.
- Para martes de directorio: abre el link 1 minuto antes para que arranque caliente.

## 5) Mantenimiento

- **Agregar otro SDR**: nada que hacer, el filtro de usuario lista todos los owners en vivo.
- **Cambiar TTL del caché**: edita `CACHE_TTL` en `utils/hubspot.py` (en segundos).
- **Filtros de números 2FA**: agrega/quita números en `KNOWN_2FA_NUMBERS` en `utils/hubspot.py`.
- **Rotar el token**: créalo nuevo en HubSpot, reemplaza en `secrets.toml` (local) y en Streamlit Cloud → Settings → Secrets.

## Stack

- Streamlit 1.39 · Pandas · Plotly · Requests
- Python 3.11
- HubSpot CRM API v3
