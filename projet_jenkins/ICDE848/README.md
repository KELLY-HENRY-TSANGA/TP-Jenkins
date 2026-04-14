
# TP Jenkins – Boutique en ligne
## ICDE848 – Intégration Continue : Serveur, Tests & Métriques

---

## 📁 Structure du projet

Basé sur la structure réelle de votre dépôt GitHub :

```text
TP-Jenkins/ (Racine du dépôt)
└── projet_jenkins/
    └── ICDE848/             <-- Dossier racine du projet Maven
        ├── pom.xml          <-- Configuration Maven complète
        ├── checkstyle.xml   <-- Règles de style de code
        ├── Jenkinsfile      <-- Pipeline CI complète
        ├── README.md        <-- Ce fichier
        └── src/
            ├── main/java/fr/epsi/FGF
            │   ├── model/
            │   │   ├── Article.java
            │   │   └── Panier.java
            │   └── service/
            │       └── CommandeService.java
            └── test/java/fr/epsi/service/
                ├── CommandeServiceTest.java  <-- Tests unitaires
                └── CommandeServiceIT.java    <-- Tests d'intégration
