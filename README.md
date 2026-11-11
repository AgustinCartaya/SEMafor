# 🚦 SEMàfor

**Sistema de suport a la decisió per al Sistema d'Emergències Mèdiques (SEM) de Catalunya que tria el millor hospital de destinació per a cada pacient, en temps real.**

🔗 **Demo interactiva (mockup):** https://agustincartaya.github.io/SEMafor/

---

## 📌 El problema

Quan una ambulància del SEM atén una emergència (un ictus, un infart, un politraumatisme…), cal decidir **a quin hospital portar el pacient**. Avui aquesta decisió depèn en gran part de **trucades telefòniques** des de la central del SEM als hospitals per preguntar:

- Hi ha TAC / sala d'hemodinàmica / quiròfan disponible?
- Hi ha de guàrdia l'especialista necessari (neurologia, cardiologia, cirurgia…)?
- Queden llits d'UCI lliures?
- Com de saturat està el servei d'urgències?

Aquestes trucades **consumeixen minuts crítics**, ocupen el personal de la central i dels hospitals, i la informació pot quedar desactualitzada de seguida. A més, l'hospital més proper no sempre és el millor: si no pot tractar la patologia, el pacient necessitarà un **trasllat secundari**, i es perd encara més temps.

## 💡 La solució

SEMàfor **redueix (o elimina) aquestes trucades** obtenint automàticament l'estat dels recursos de cada hospital, i fa servir **Intel·ligència Artificial** per recomanar la destinació que minimitza el **temps fins al tractament definitiu** (*"Fins al tractament"*), i no només el temps d'arribada.

> ⏱️ **Fins al tractament** = trajecte amb trànsit real + temps habitual del centre des de la porta fins al tractament (+ possible trasllat secundari).

La recomanació es mostra a l'operador del SEM en una interfície gràfica clara: l'operador **sempre té l'última paraula**; SEMàfor li dona la informació i una proposta raonada.

## 🏗️ Arquitectura

SEMàfor es basa en **dos blocs** i s'**acobla al sistema actual del SEM** (gestió d'incidents, ambulàncies, localització de recursos…) sense reinventar la roda.

```
 ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
 │  Hospital A  │   │  Hospital B  │   │  Hospital N  │   Sistemes propis de cada
 │ (HIS, llits, │   │ (HIS, llits, │   │ (HIS, llits, │   hospital, cadascun amb el
 │  quiròfans)  │   │  quiròfans)  │   │  quiròfans)  │   seu format
 └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
        │ ETL              │ ETL              │ ETL        ① Extracció i normalització
        ▼                  ▼                  ▼
 ┌─────────────────────────────────────────────────────┐
 │       Dades unificades de recursos hospitalaris      │
 └──────────────────────────┬──────────────────────────┘
                            │
   Sistema actual del SEM ──┤  (incident, ubicació, ambulàncies,
   Trànsit en temps real ───┤   patologia, gravetat…)
                            ▼
 ┌─────────────────────────────────────────────────────┐
 │  Motor de decisió amb IA (Graph Neural Networks)     │   ② Decisió intel·ligent
 └──────────────────────────┬──────────────────────────┘
                            ▼
 ┌─────────────────────────────────────────────────────┐
 │       Interfície gràfica per a l'operador del SEM    │   Recomanació + explicació
 └─────────────────────────────────────────────────────┘
```

### ① Extracció d'informació (ETL)

Cada hospital té els seus propis sistemes i formats. SEMàfor col·loca **una ETL per hospital** que fa de "traductor":

- **Extract**: llegeix l'estat dels recursos de l'hospital (llits, UCI, TAC, RM, hemodinàmica, quiròfans, especialistes de guàrdia, saturació d'urgències…).
- **Transform**: converteix aquestes dades a un **format comú** i validat.
- **Load**: les publica de manera contínua en un repositori unificat accessible per al SEM.

Així el SEM disposa d'una **visió actualitzada de tota la xarxa hospitalària** sense haver de trucar.

### ② Decisió amb IA (Graph Neural Networks)

La xarxa sanitària es modela de manera natural com un **graf**: hospitals, ubicació de l'incident i ambulàncies són nodes; les carreteres i les rutes de trasllat entre centres són arestes. Una **GNN** aprofita aquesta estructura per puntuar cada hospital candidat tenint en compte:

| Paràmetre | Exemple |
|---|---|
| 🏥 Disponibilitat dels recursos necessaris | TAC + neurologia de guàrdia per a un codi ictus |
| 📍 Distància | Km reals per carretera |
| 🚗 Trànsit | Temps de trajecte estimat en temps real |
| 🔁 Trasllats secundaris | Probabilitat / cost d'haver de derivar després |
| ⏳ Temps porta-tractament | Rendiment habitual de cada centre |
| 📈 Saturació | Càrrega actual d'urgències i UCI |
| 🩺 Patologia i gravetat | Codis d'activació (ictus, IAM, politrauma, sèpsia…) |

El resultat és un **rànquing d'hospitals** amb l'opció recomanada i el motiu, que l'operador pot acceptar o modificar.

### 🔌 Integració amb el SEM

SEMàfor **no substitueix** les eines actuals del SEM: s'hi connecta per reutilitzar el que ja existeix (gestió d'incidents, flota d'ambulàncies, geolocalització) i hi afegeix una capa d'informació hospitalària i de recomanació intel·ligent.

## 🖥️ Mockup

Aquest repositori conté un **mockup interactiu** ([`SEMafor.html`](SEMafor.html)) que simula el flux complet: una emergència, l'extracció de dades dels hospitals via ETL i la recomanació de destinació de SEMàfor segons el temps fins al tractament.

- **En línia:** https://agustincartaya.github.io/SEMafor/
- **En local:** descarrega el repositori i obre `SEMafor.html` en qualsevol navegador (no cal instal·lació ni servidor).

> ⚠️ És una demo conceptual amb dades simulades; no està connectada a sistemes reals del SEM ni dels hospitals.

## 📂 Estructura del repositori

```
SEMafor/
├── SEMafor.html   # Mockup interactiu (autocontingut)
├── index.html     # Redirigeix a SEMafor.html (pàgina d'inici de GitHub Pages)
├── .nojekyll      # Serveix els fitxers tal qual a GitHub Pages
└── README.md
```

## 🗺️ Propers passos

- [ ] Definir l'esquema comú de dades hospitalàries.
- [ ] Prototip d'ETL amb dades d'un hospital pilot.
- [ ] Entrenar i validar el model GNN amb dades històriques de trasllats.
- [ ] Integració amb l'API del sistema de gestió del SEM.
- [ ] Validació amb operadors del SEM.

## 👥 Equip

Projecte desenvolupat en el marc de **HackIDIBGI** (VICOROB – Universitat de Girona).
