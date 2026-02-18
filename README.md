# 🧩 TP VLAN & Masques — Comprendre la segmentation réseau

> Atelier pratique pour comprendre le rôle des VLAN et des masques IP dans la segmentation d’un réseau.

---

# 🎯 Objectifs pédagogiques

À la fin de ce TP, vous devrez savoir :

✅ Expliquer le rôle d’un VLAN  
✅ Comprendre le lien VLAN ↔ sous-réseau IP  
✅ Calculer et utiliser des masques IP  
✅ Mettre en place un trunk 802.1Q  
✅ Tester la communication intra/inter-VLAN

---

# 🧠 Rappel théorique simple

## 📌 VLAN
Un VLAN est un **réseau logique** sur un même switch physique.

👉 Chaque VLAN = un domaine de broadcast distinct  
👉 Chaque VLAN = un sous-réseau IP différent

---

## 📌 Masque de sous-réseau
Le masque définit :
- La partie réseau  
- La partie hôte  

| Masque | Nb hôtes |
|------|---------|
   /24 | 254 |
   /25 | 126 |
   /26 | 62 |

---

# 🗺️ Topologie du TP

```
         VLAN 10         VLAN 20
        192.168.10.0/24 192.168.20.0/24

PC1 -------- SW1 -------- R1 -------- PC3
              |  (trunk)
              |
             PC2
```

---

# 🧩 Matériel Packet Tracer

- 1 routeur (2911)  
- 1 switch (2960)  
- 3 PC  

---

# 🌐 Plan d’adressage

## VLAN 10

| Équipement | IP |
|----------|------|
PC1 | 192.168.10.10/24 |
PC2 | 192.168.10.20/24 |
GW | 192.168.10.1 |

---

## VLAN 20

| Équipement | IP |
|----------|------|
PC3 | 192.168.20.10/24 |
GW | 192.168.20.1 |

---

# 🔹 PARTIE 1 — Création des VLAN

Sur le switch :

```
enable
conf t
vlan 10
name ADMIN
vlan 20
name USERS
```

---

# 🔹 PARTIE 2 — Affectation des ports

```
interface f0/1
switchport mode access
switchport access vlan 10

interface f0/2
switchport mode access
switchport access vlan 10

interface f0/3
switchport mode access
switchport access vlan 20
```

---

# 🔹 PARTIE 3 — Trunk vers le routeur

```
interface f0/24
switchport mode trunk
```

---

# 🔹 PARTIE 4 — Router-on-a-stick

Sur R1 :

```
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface g0/0
no shutdown
```

---

# 🔹 PARTIE 5 — Configuration IP des PC

Configurer IP + passerelle selon le plan d’adressage.

---

# 🧪 TESTS

## Test 1 — Intra-VLAN
PC1 → PC2  
👉 Doit fonctionner

![Screenshot Actions](Capture/PC1-PC2.png)

---

## Test 2 — Inter-VLAN
PC1 → PC3  
👉 Fonctionne uniquement grâce au routeur

![Screenshot Actions](Capture/PC1-PC3.png)
  
---

# ❓ Questions de réflexion

1. Pourquoi PC1 ne voit-il pas PC3 sans routeur ?

Comme le routeur héberge les deux passerelles (gateways) des deux VLAN, les PC ne sauraient pas où envoyer les paquets pour joindre l'autre VLAN s'il n'y avait pas de routeur.

2. Quel rôle joue le masque /24 ?

Le masque définit la taille du réseau, ça permet au PC de savoir si l'adresse qu'il veut ping est locale ou s'il doit envoyer le paquet à la gateway pour sortir.

3. Que se passe-t-il si VLAN 10 et VLAN 20 ont le même réseau IP ?

S'ils étaient sur le même réseau, les PC penseraient qu'ils sont voisins et essaieraient de communiquer en direct sans passer par la gateway, mais le switch va tout bloquer car ils ne sont pas dans le même VLAN.

4. Pourquoi un trunk est-il nécessaire ?

Le trunk permet de faire passer le trafic des 2 VLAN dans le seul câble qui est branché au routeur, sinon il faudrait tirer un câble physique pour chaque VLAN.

---

# ⭐ Travail sur les Masques

Changer VLAN 10 en :

```
192.168.10.0/25
```

Questions :
- Combien d’hôtes max ?

Le masque /25 laisse 7 bits pour les hôtes, Cela fait 126 adresses utilisables pour les PC (128 moins l'adresse réseau et le broadcast). 

- Quelle plage IP valide ?

De 192.168.10.1 à 192.168.10.126

- Peut-on encore communiquer avec VLAN 20 ?

Oui, car le routeur fait le lien entre les réseaux peu importe leur taille, mais si on ne change pas le masque sur la gateway (interface du routeur) et sur les PC, le ping ne marchera pas.

---

# 🚀 Extensions

- Ajouter VLAN 30  
- Mettre un DHCP par VLAN  
  
---

# 📝 Évaluation (/20)

| Critère | Points |
|--------|-------|
VLAN créés correctement | 4 |
Ports bien affectés | 2 |
Trunk opérationnel | 4 |
Inter-VLAN fonctionnel | 4 |
Travail sur les masques | 4 |  
Extention | 2 |  
  
# ✅ Fin du TP

Si vous savez expliquer :
> "Pourquoi deux VLAN ne communiquent pas sans routeur ?"

Alors vous avez compris la segmentation réseau 👍
