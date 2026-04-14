# Authentification-802.1x-filaire


```markdown
# 802.1X Filaire - Contrôle d'accès réseau

## 📌 Objectif

Sécuriser l'accès réseau filaire en :
- authentifiant les utilisateurs
- attribuant dynamiquement un VLAN
- différenciant profils (staff, étudiant, invité)

---

## 🧱 Architecture

- Switch : mode authenticator
- Client : supplicant (Windows/Linux)
- Serveur : FreeRADIUS
- Backend : LDAP

---

## 🔄 Fonctionnement

1. Connexion du client
2. Authentification EAP (TTLS)
3. Vérification via LDAP
4. Attribution VLAN via RADIUS

---

## 🔐 Configuration AAA (802.1X)

```bash
aaa group server radius DOT1X-GROUP
 server name radius-dot1x

aaa authentication dot1x default group DOT1X-GROUP
aaa authorization network default group DOT1X-GROUP
aaa accounting identity default start-stop group DOT1X-GROUP
````

---

## 🔌 Configuration interface switch

```bash
interface GigabitEthernet1/0/1
 switchport mode access
 authentication port-control auto
 dot1x pae authenticator
 service-policy type control subscriber DOT1X-POLICY
```

---

## 📜 Policy 802.1X

```bash
policy-map type control subscriber DOT1X-POLICY

 event session-started match-all
  10 authenticate using dot1x

 event authentication-failure match-first
  10 terminate dot1x
  20 authentication-restart 60

 event authentication-success match-all
  10 activate service-template DEFAULT_POLICY
```

---

## 🧠 Détection type d'authentification (FreeRADIUS)

```bash
if (&User-Name =~ /^[0-9a-f]{12}$/i) {
    Auth-Type := mab
}
elsif (&EAP-Message) {
    Auth-Type := DOT1X
}
```

---

## 🏷️ Attribution VLAN dynamique

Exemple :

```bash
if (User-Category == "staff") {
    Tunnel-Private-Group-Id := **
}
elseif (User-Category == "student") {
    Tunnel-Private-Group-Id := **
}
else {
    Tunnel-Private-Group-Id := **
}
```

---

## 🔗 Mapping LDAP → RADIUS

```bash
reply:User-Category = eduPersonPrimaryAffiliation
reply:Group-Name    = supannEntiteAffectationPrincipale
```

---

## 🔐 Configuration EAP

```bash
eap {
  default_eap_type = ttls

  ttls {
    default_eap_type = pap
  }
}
```



## 📊 Logs utiles

* RADIUS : authentification OK/KO
* LDAP : bind + recherche utilisateur
* Client : EAP success
* DHCP : attribution IP

---

## ✅ Résultat

* Accès réseau sécurisé
* VLAN dynamique par profil
* Support utilisateurs + équipements (MAB)

<img width="1408" height="768" alt="Gemini_Generated_Image_dm2ebadm2ebadm2e" src="https://github.com/user-attachments/assets/6f9664aa-16ae-475e-b27f-a8e874d4f3bb" />



