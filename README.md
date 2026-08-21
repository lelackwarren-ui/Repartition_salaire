# R-partition_salaire

Bien sûr. Voici un **README.md complet, clair et professionnel**, adapté à ton projet **Répartition salaire**, en tenant compte des différentes interfaces visibles sur tes captures et des fonctionnalités que nous avons développées ensemble : Login, ajout utilisateur, répartition, vérification, statistiques/diagramme, CSV, e-mail, etc.

Tu peux **copier directement tout le contenu ci-dessous dans `README.md`**.

````markdown
# 💰 Répartition Salaire

Application Windows Forms développée en **C#** permettant de gérer et d'analyser la répartition d'un salaire.

L'application permet notamment de :

- 🔐 Se connecter avec un identifiant et un mot de passe
- 👤 Ajouter des utilisateurs
- 🔑 Modifier les informations d'un utilisateur
- 💰 Enregistrer une répartition de salaire
- 🧮 Calculer automatiquement l'épargne restante
- 📊 Consulter les données enregistrées
- 📈 Visualiser l'évolution des salaires et dépenses avec des graphiques
- 📄 Exporter les données au format CSV
- 📧 Envoyer un rapport par e-mail
- 🗑️ Supprimer des données
- 🚪 Se déconnecter

---

# 📌 Sommaire

1. [Présentation du projet](#-présentation-du-projet)
2. [Technologies utilisées](#-technologies-utilisées)
3. [Architecture de l'application](#-architecture-de-lapplication)
4. [Système de connexion](#-système-de-connexion)
5. [Dashboard / Répartition du salaire](#-dashboard--répartition-du-salaire)
6. [Calcul de l'épargne](#-calcul-de-lépargne)
7. [Enregistrement des données](#-enregistrement-des-données)
8. [Interface Vérification](#-interface-vérification)
9. [Ajout d'utilisateur](#-ajout-dutilisateur)
10. [Diagrammes et statistiques](#-diagrammes-et-statistiques)
11. [Export CSV](#-export-csv)
12. [Envoi d'e-mail](#-envoi-de-mail)
13. [Suppression des données](#-suppression-des-données)
14. [Gestion de la session utilisateur](#-gestion-de-la-session-utilisateur)
15. [Base de données](#-base-de-données)
16. [Installation et configuration](#-installation-et-configuration)
17. [Configuration pour modifier le code](#-configuration-pour-modifier-le-code)
18. [Structure du projet](#-structure-du-projet)
19. [Sécurité](#-sécurité)
20. [Améliorations futures](#-améliorations-futures)
21. [Auteur](#-auteur)
22. [Licence](#-licence)

---

# 📖 Présentation du projet

**Répartition Salaire** est une application de bureau développée avec **C# Windows Forms**.

L'objectif du projet est de permettre à un utilisateur de gérer son salaire mensuel en indiquant différentes catégories de dépenses.

Les catégories principales sont :

- Salaire net
- Loyer
- Dépenses
- Transport
- Nourriture
- Autres dépenses
- Épargne restante

L'application calcule automatiquement le montant restant après déduction des différentes dépenses.

### Formule utilisée

```text
Épargne = Salaire Net
        - Loyer
        - Dépenses
        - Transport
        - Nourriture
        - Autres
````

---

# 🛠️ Technologies utilisées

## Langage

* C#

## Framework

* Windows Forms
* .NET Framework

## Base de données

* SQL Server
* SQL Server LocalDB

## Outils

* Visual Studio
* SQL Server Management Studio / SQL Server
* Git
* GitHub

## Fonctionnalités supplémentaires

* `DataGridView`
* `Chart`
* `SaveFileDialog`
* Export CSV
* `System.Net.Mail`

---

# 🏗️ Architecture de l'application

L'application est organisée autour de plusieurs interfaces Windows Forms.

Le fonctionnement général est :

```text
                    ┌──────────────┐
                    │    Login     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Dashboard   │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
       Répartition     Vérification   Diagramme
        Salaire          Salaire      Statistiques
             │             │             │
             ▼             ▼             ▼
          SQL Server    DataGridView    Chart
```

---

# 🔐 Système de connexion

## Interface Login

L'interface Login constitue la première étape de l'application.

Elle permet à l'utilisateur de saisir :

* Identifiant
* Mot de passe

Les boutons disponibles sont :

* **Login**
* **Reset**
* **Add**
* **Exit**
* **Changer de mot de passe**

### Fonctionnement

Lorsqu'un utilisateur se connecte, l'application vérifie les informations dans la table `Profil`.

Exemple de requête :

```sql
SELECT Idname, Pwd, Name
FROM Profil
WHERE Idname = @Idname
AND Pwd = @Pwd;
```

Les requêtes utilisent des paramètres SQL afin d'éviter de placer directement les valeurs saisies par l'utilisateur dans la requête.

---

# 👤 Gestion de l'utilisateur connecté

Après une connexion réussie, l'identifiant de l'utilisateur est conservé afin que les autres formulaires puissent savoir quel utilisateur est connecté.

Exemple :

```csharp
if (reader.Read())
{
    Login.IdProfilConnecte = reader["Idname"].ToString();

    Dashboard dashboard = new Dashboard();
    dashboard.Show();

    this.Hide();
}
```

La variable :

```csharp
Login.IdProfilConnecte
```

contient alors l'identifiant de l'utilisateur connecté.

Cette valeur est ensuite utilisée dans les autres formulaires.

Par exemple :

```sql
SELECT *
FROM Salaire
WHERE Idname = @Idname;
```

avec :

```csharp
cmd.Parameters.AddWithValue(
    "@Idname",
    Login.IdProfilConnecte
);
```

Cela permet d'éviter d'afficher les données appartenant à d'autres utilisateurs.

---

# 💰 Dashboard / Répartition du salaire

L'interface principale de répartition permet de saisir les informations financières du mois.

Les champs disponibles sont :

```text
Nom
Date
Salaire Net
Loyer
Dépenses
Transport
Nourriture
Autres
Reste (Épargne)
```

Les boutons sont :

```text
Calculer
Enregistrer
Vérifier
Revenir
```

---

# 👤 Affichage automatique du nom

Lorsque le formulaire est chargé, le nom de l'utilisateur connecté est récupéré depuis la base de données.

Exemple :

```csharp
private void RepartitionSalaire_Load(
    object sender,
    EventArgs e)
{
    try
    {
        using (SqlConnection con =
            new SqlConnection(connectionString))
        {
            con.Open();

            string requete = @"
                SELECT Name
                FROM Profil
                WHERE Idname = @Idname";

            using (SqlCommand cmd =
                new SqlCommand(requete, con))
            {
                cmd.Parameters.AddWithValue(
                    "@Idname",
                    Login.IdProfilConnecte);

                object resultat =
                    cmd.ExecuteScalar();

                if (resultat != null)
                {
                    nomtxt.Text =
                        resultat.ToString();
                }
            }
        }
    }
    catch (Exception ex)
    {
        MessageBox.Show(
            "Erreur : " + ex.Message);
    }
}
```

Le champ **Nom** est donc rempli automatiquement.

---

# 🧮 Calcul de l'épargne

Le bouton **Calculer** permet de calculer automatiquement le montant restant.

La formule est :

```text
Reste =
Salaire Net
- Loyer
- Dépenses
- Transport
- Nourriture
- Autres
```

Exemple de code :

```csharp
private void btnCalculer_Click(
    object sender,
    EventArgs e)
{
    try
    {
        decimal salaire =
            decimal.Parse(txtSalaireNet.Text);

        decimal loyer =
            decimal.Parse(txtLoyer.Text);

        decimal depenses =
            decimal.Parse(txtDepenses.Text);

        decimal transport =
            decimal.Parse(txtTransport.Text);

        decimal nourriture =
            decimal.Parse(txtNourriture.Text);

        decimal autres =
            decimal.Parse(txtAutres.Text);

        decimal reste =
            salaire
            - loyer
            - depenses
            - transport
            - nourriture
            - autres;

        txtReste.Text =
            reste.ToString("0.00");
    }
    catch (Exception ex)
    {
        MessageBox.Show(
            "Veuillez vérifier les valeurs saisies.\n\n"
            + ex.Message,
            "Erreur",
            MessageBoxButtons.OK,
            MessageBoxIcon.Error);
    }
}
```

---

# 💾 Enregistrement des données

Le bouton **Enregistrer** permet d'enregistrer les informations dans la table `Salaire`.

Exemple de requête :

```sql
INSERT INTO Salaire
(
    SalaireNet,
    Loyer,
    Depenses,
    Transport,
    Nourriture,
    Autres,
    Reste,
    Date,
    Idname
)
VALUES
(
    @SalaireNet,
    @Loyer,
    @Depenses,
    @Transport,
    @Nourriture,
    @Autres,
    @Reste,
    @Date,
    @Idname
);
```

Les valeurs sont transmises avec des paramètres :

```csharp
cmd.Parameters.AddWithValue(
    "@SalaireNet",
    decimal.Parse(txtSalaireNet.Text));

cmd.Parameters.AddWithValue(
    "@Loyer",
    decimal.Parse(txtLoyer.Text));

cmd.Parameters.AddWithValue(
    "@Depenses",
    decimal.Parse(txtDepenses.Text));

cmd.Parameters.AddWithValue(
    "@Transport",
    decimal.Parse(txtTransport.Text));

cmd.Parameters.AddWithValue(
    "@Nourriture",
    decimal.Parse(txtNourriture.Text));

cmd.Parameters.AddWithValue(
    "@Autres",
    decimal.Parse(txtAutres.Text));

cmd.Parameters.AddWithValue(
    "@Reste",
    decimal.Parse(txtReste.Text));

cmd.Parameters.AddWithValue(
    "@Date",
    DateTime.Parse(txtDate.Text));

cmd.Parameters.AddWithValue(
    "@Idname",
    Login.IdProfilConnecte);
```

---

# 🔎 Interface Vérification

L'interface **Vérification** permet de consulter les données enregistrées.

Les données sont affichées dans un :

```text
DataGridView
```

Exemple :

```text
┌─────────┬────────────┬───────┬──────────┬────────┬─────────┐
│ IdSalaire│ SalaireNet │ Loyer │ Depenses │ Autres │ Reste   │
├─────────┼────────────┼───────┼──────────┼────────┼─────────┤
│    1    │ 17698      │ 6000  │ 2000     │ 1000   │ 4698    │
└─────────┴────────────┴───────┴──────────┴────────┴─────────┘
```

Les données sont filtrées par l'utilisateur connecté :

```sql
SELECT
    IdSalaire,
    SalaireNet,
    Loyer,
    Depenses,
    Transport,
    Nourriture,
    Autres,
    Reste,
    Date,
    Idname
FROM Salaire
WHERE Idname = @Idname;
```

---

# 👤 Interface Ajout d'utilisateur

L'interface **Ajout d'utilisateur** permet de créer un nouveau compte.

Elle contient :

```text
Identifiant
Mot de passe
Nom
```

Les informations sont enregistrées dans la table :

```text
Profil
```

Requête utilisée :

```sql
INSERT INTO Profil
(
    Idname,
    Pwd,
    Name
)
VALUES
(
    @Idname,
    @Pwd,
    @Name
);
```

Le `DataGridView` permet également d'afficher les utilisateurs présents dans la base.

---

# 📊 Diagrammes et statistiques

L'application possède une interface **Diagramme** permettant d'analyser l'évolution des données.

Le contrôle utilisé est :

```text
Chart
```

L'utilisateur peut choisir la donnée à afficher avec des `RadioButton`.

Les choix sont :

```text
○ SalaireNet
○ Épargne
○ Loyer
○ Dépense
○ Nourriture
○ Transport
○ Autres
```

---

# 📈 Fonctionnement du graphique

Le graphique représente l'évolution d'une donnée dans le temps.

Par exemple :

```text
Date          Épargne

22/06/2026    4700 Rs
04/08/2026    1000 Rs
```

Le graphique utilise :

```text
Axe X → Date
Axe Y → Montant
```

Le changement de `RadioButton` permet de modifier automatiquement la colonne affichée.

---

# 🔄 Méthode ChargerGraphique()

La méthode principale utilisée pour afficher les statistiques est :

```csharp
private void ChargerGraphique(string colonne)
{
    // récupération des données SQL
    // création de la série
    // affichage des données
}
```

Le paramètre :

```csharp
colonne
```

permet de déterminer quelle donnée doit être affichée.

Exemples :

```csharp
ChargerGraphique("SalaireNet");
```

ou :

```csharp
ChargerGraphique("Reste");
```

ou :

```csharp
ChargerGraphique("Depenses");
```

---

# 📌 RadioButtons

Chaque RadioButton correspond à une colonne SQL.

| RadioButton    | Colonne SQL  |
| -------------- | ------------ |
| `rbSalaireNet` | `SalaireNet` |
| `rbEpargne`    | `Reste`      |
| `rbLoyer`      | `Loyer`      |
| `rbDepenses`   | `Depenses`   |
| `rbTransport`  | `Transport`  |
| `rbNourriture` | `Nourriture` |
| `rbAutres`     | `Autres`     |

Exemple :

```csharp
private void rbEpargne_CheckedChanged(
    object sender,
    EventArgs e)
{
    if (rbEpargne.Checked)
    {
        ChargerGraphique("Reste");
    }
}
```

---

# 🔄 Bouton Actualiser

Le bouton **Actualiser** permet de recharger les données SQL et de redessiner le graphique.

Exemple :

```csharp
private void btnActualiser_Click(
    object sender,
    EventArgs e)
{
    if (rbSalaireNet.Checked)
        ChargerGraphique("SalaireNet");

    else if (rbEpargne.Checked)
        ChargerGraphique("Reste");

    else if (rbLoyer.Checked)
        ChargerGraphique("Loyer");

    else if (rbDepenses.Checked)
        ChargerGraphique("Depenses");

    else if (rbTransport.Checked)
        ChargerGraphique("Transport");

    else if (rbNourriture.Checked)
        ChargerGraphique("Nourriture");

    else if (rbAutres.Checked)
        ChargerGraphique("Autres");
}
```

---

# 📄 Export CSV

L'application permet d'exporter les données dans un fichier :

```text
Statistiques.csv
```

Le bouton utilisé est :

```text
CSV
```

L'utilisateur peut choisir l'emplacement avec :

```csharp
SaveFileDialog
```

Le fichier contient notamment :

```text
Date
SalaireNet
Loyer
Depenses
Transport
Nourriture
Autres
Reste
```

Exemple :

```text
Date,SalaireNet,Loyer,Depenses,Transport,Nourriture,Autres,Reste
22/06/2026,17698,6000,2000,2000,1000,1000,5698
04/08/2026,17698,6000,2000,2000,1000,1000,5698
```

---

# 📧 Envoi d'e-mail

Le bouton :

```text
Envoyer E-mail
```

permet d'envoyer un rapport par e-mail.

L'adresse du destinataire est saisie dans :

```text
txtEmail
```

Le fichier CSV peut être ajouté comme pièce jointe.

Le projet peut utiliser :

```csharp
System.Net.Mail
```

avec un serveur SMTP.

Exemple de structure :

```csharp
MailMessage mail =
    new MailMessage();

mail.From = new MailAddress(
    "adresse@example.com");

mail.To.Add(txtEmail.Text);

mail.Subject =
    "Rapport des statistiques salariales";

mail.Body =
    "Bonjour,\n\n"
    + "Veuillez trouver ci-joint "
    + "le rapport des statistiques salariales.\n\n"
    + "Cordialement.";

mail.Attachments.Add(
    new Attachment("Statistiques.csv"));
```

Le serveur SMTP est ensuite configuré avec `SmtpClient`.

---

# 🗑️ Suppression des données

L'interface Vérification possède également un bouton :

```text
Supprimer
```

Il permet de supprimer une ligne sélectionnée dans le `DataGridView`.

La suppression doit être réalisée avec une requête paramétrée.

Exemple :

```sql
DELETE FROM Salaire
WHERE IdSalaire = @IdSalaire
AND Idname = @Idname;
```

Le deuxième critère permet d'éviter qu'un utilisateur puisse supprimer les données appartenant à un autre utilisateur.

---

# 🔄 Déconnexion

Le bouton :

```text
Log out
```

permet de revenir à l'interface Login.

Le principe est :

```text
Utilisateur connecté
       ↓
Dashboard
       ↓
Diagramme
       ↓
Log out
       ↓
Login
```

Lors d'une nouvelle connexion, une nouvelle valeur est placée dans :

```csharp
Login.IdProfilConnecte
```

---

# 🗄️ Base de données

L'application utilise SQL Server.

Deux tables principales sont utilisées.

## Table Profil

Cette table contient les utilisateurs.

Exemple :

```sql
CREATE TABLE Profil
(
    IdProfil INT IDENTITY(1,1) PRIMARY KEY,
    Idname VARCHAR(50) NOT NULL,
    Pwd VARCHAR(255) NOT NULL,
    Name VARCHAR(100) NOT NULL
);
```

---

## Table Salaire

Cette table contient les informations financières.

```sql
CREATE TABLE Salaire
(
    IdSalaire INT IDENTITY(1,1) PRIMARY KEY,

    SalaireNet DECIMAL(10,2) NOT NULL,
    Loyer DECIMAL(10,2) NOT NULL,
    Depenses DECIMAL(10,2) NOT NULL,
    Transport DECIMAL(10,2) NOT NULL,
    Nourriture DECIMAL(10,2) NOT NULL,
    Autres DECIMAL(10,2) NOT NULL,

    Reste DECIMAL(10,2) NOT NULL,

    Date DATETIME NOT NULL,

    Idname VARCHAR(50) NOT NULL
);
```

---

# 🔗 Relation utilisateur / salaire

La colonne :

```text
Idname
```

permet d'associer une répartition de salaire à l'utilisateur connecté.

Exemple :

```text
Profil

Idname       Name
-------------------------
user01       Warren Kenji
user02       Alice
```

Table Salaire :

```text
IdSalaire    Idname       SalaireNet
-------------------------------------
1            user01       17698
2            user01       18000
3            user02       25000
```

Lorsque `user01` se connecte, l'application utilise :

```csharp
Login.IdProfilConnecte
```

pour récupérer uniquement :

```text
user01
```

et ses données.

---

# 🔐 Filtrage des données

Toutes les opérations concernant les salaires doivent utiliser :

```sql
WHERE Idname = @Idname
```

avec :

```csharp
cmd.Parameters.AddWithValue(
    "@Idname",
    Login.IdProfilConnecte);
```

Cela permet de séparer les données des utilisateurs.

---

# ⚙️ Installation et configuration

> ⚠️ Important : le projet actuel est principalement le **code source** de l'application. Un projet Setup/Installer n'est pas encore inclus.

Pour exécuter le projet depuis Visual Studio, il faut disposer de l'environnement nécessaire.

## Configuration nécessaire

* Windows
* Visual Studio
* .NET Framework / environnement .NET correspondant au projet
* SQL Server / SQL Server LocalDB
* Base de données configurée

---

# 🧩 Modifier le code source

Pour modifier le projet, il est recommandé d'utiliser :

```text
Visual Studio
```

Le développeur doit disposer du framework utilisé par le projet ainsi que de SQL Server.

La chaîne de connexion doit correspondre à l'emplacement du serveur SQL et/ou de la base de données utilisée.

Exemple pour LocalDB :

```csharp
string connectionString =
@"Data Source=(LocalDB)\MSSQLLocalDB;
AttachDbFilename=C:\Chemin\Database1.mdf;
Integrated Security=True";
```

⚠️ Le chemin :

```text
C:\Chemin\Database1.mdf
```

est spécifique à l'ordinateur.

Il doit donc être adapté lorsque le projet est déplacé sur un autre ordinateur.

---

# 📁 Structure du projet

La structure peut être organisée de cette manière :

```text
Repartition_salaire/
│
├── Login.cs
├── Dashboard.cs
├── RepartitionSalaire.cs
├── Verification.cs
├── AjouteUtilisateur.cs
├── Diagramme.cs
│
├── Properties/
│   └── Resources.resx
│
├── Images/
│   ├── login.png
│   └── autres-images
│
├── Database/
│   └── Database1.mdf
│
├── App.config
│
├── README.md
└── LICENSE
```

Les noms peuvent être différents selon l'organisation du projet Visual Studio.

---

# 🖥️ Interfaces de l'application

## 1. Login

L'écran Login permet de :

* saisir l'identifiant ;
* saisir le mot de passe ;
* se connecter ;
* réinitialiser les champs ;
* ajouter un utilisateur ;
* modifier le mot de passe ;
* quitter l'application.

---

## 2. Répartition salaire

Cette interface permet de saisir :

```text
Nom
Date
Salaire Net
Loyer
Dépenses
Transport
Nourriture
Autres
```

Puis :

```text
Calculer
Enregistrer
Vérifier
Revenir
```

Le nom est récupéré automatiquement depuis le compte connecté.

---

## 3. Vérification

Cette interface permet de consulter les données enregistrées dans `Salaire`.

Elle utilise un :

```text
DataGridView
```

Fonctionnalités :

* consulter les données ;
* exporter les données en CSV ;
* envoyer un rapport par e-mail ;
* supprimer une ligne ;
* quitter ;
* revenir à la connexion.

---

## 4. Ajout d'utilisateur

Cette interface permet d'ajouter un utilisateur avec :

```text
Identifiant
Mot de passe
Nom
```

Les utilisateurs existants sont affichés dans un `DataGridView`.

---

## 5. Diagramme

L'interface Diagramme permet d'analyser les données.

L'utilisateur peut sélectionner :

```text
SalaireNet
Épargne
Loyer
Dépense
Nourriture
Transport
Autres
```

Le graphique affiche l'évolution de la valeur sélectionnée en fonction de la date.

---

# 📊 Exemple de fonctionnement

Un utilisateur enregistre :

```text
Salaire Net : 17 698 Rs

Loyer       : 6 000 Rs
Dépenses    : 2 000 Rs
Transport   : 2 000 Rs
Nourriture  : 1 000 Rs
Autres      : 1 000 Rs
```

L'application calcule :

```text
17 698
- 6 000
- 2 000
- 2 000
- 1 000
- 1 000
--------
 5 698 Rs
```

L'épargne enregistrée est donc :

```text
5 698 Rs
```

Cette valeur peut ensuite être affichée dans le graphique.

---

# 🔒 Sécurité

Les requêtes SQL utilisent des paramètres :

```csharp
cmd.Parameters.AddWithValue(
    "@Idname",
    Login.IdProfilConnecte);
```

au lieu de construire des requêtes avec des chaînes directement.

### Exemple à éviter

```csharp
string requete =
    "SELECT * FROM Profil WHERE Idname='"
    + txtIdentifiant.Text
    + "'";
```

### Exemple recommandé

```csharp
string requete =
    "SELECT * FROM Profil WHERE Idname=@Idname";

cmd.Parameters.AddWithValue(
    "@Idname",
    txtIdentifiant.Text);
```

Cette méthode réduit les risques d'injection SQL.

> ⚠️ Pour une application destinée à une utilisation réelle, les mots de passe ne devraient pas être stockés en clair. Il faudrait utiliser un système de hachage sécurisé.

---

# 🚀 Améliorations futures

Plusieurs fonctionnalités pourraient être ajoutées dans les prochaines versions :

* [ ] Création d'un véritable installateur Windows
* [ ] Support d'une base de données serveur distante
* [ ] Passage complet de LocalDB à SQL Server distant
* [ ] Hachage sécurisé des mots de passe
* [ ] Gestion des rôles administrateur/utilisateur
* [ ] Recherche dans les données
* [ ] Filtrage par mois et année
* [ ] Graphiques supplémentaires
* [ ] Export PDF
* [ ] Impression des rapports
* [ ] Sauvegarde automatique de la base
* [ ] Système de mise à jour automatique
* [ ] Interface plus moderne
* [ ] Version distribuable avec Setup.exe / MSI

---

# 📦 Distribution

Le code source peut être publié sur GitHub afin de permettre à d'autres développeurs de consulter ou modifier le projet.

Cependant, le simple téléchargement du code source ne garantit pas que l'application fonctionnera immédiatement sur un autre ordinateur.

Le poste doit disposer des composants nécessaires :

```text
Application
      │
      ├── .NET / Framework
      │
      ├── SQL Server / LocalDB
      │
      ├── Base de données
      │
      └── Configuration
```

Un véritable installateur pourra être créé ultérieurement avec :

```text
Visual Studio Installer Projects
```

afin de simplifier l'installation.

---

# 🐛 Gestion des erreurs

Les opérations SQL sont protégées par `try/catch`.

Exemple :

```csharp
try
{
    // Opération SQL
}
catch (Exception ex)
{
    MessageBox.Show(
        "Une erreur est survenue :\n"
        + ex.Message,
        "Erreur",
        MessageBoxButtons.OK,
        MessageBoxIcon.Error);
}
```

Cela permet d'afficher un message à l'utilisateur plutôt que de fermer brutalement l'application.

---

# 🧪 Tests

Avant de distribuer l'application, il est recommandé de tester :

### Connexion

* [ ] Identifiant correct
* [ ] Mot de passe correct
* [ ] Identifiant incorrect
* [ ] Mot de passe incorrect

### Répartition

* [ ] Saisie du salaire
* [ ] Saisie des dépenses
* [ ] Calcul de l'épargne
* [ ] Enregistrement
* [ ] Vérification dans SQL Server

### Statistiques

* [ ] SalaireNet
* [ ] Épargne
* [ ] Loyer
* [ ] Dépenses
* [ ] Transport
* [ ] Nourriture
* [ ] Autres

### Export

* [ ] Création du CSV
* [ ] Ouverture du CSV
* [ ] Vérification des données

### E-mail

* [ ] Adresse correcte
* [ ] Envoi
* [ ] Pièce jointe
* [ ] Gestion des erreurs

---

# 📜 Licence

Ce projet est distribué sous la licence :

```text
MIT License
```

Voir le fichier :

```text
LICENSE
```

pour les conditions complètes.

---

# 👨‍💻 Auteur

**Waren Kenji-LWK**

Projet :

**Répartition salaire**

Année :

**2026**

---

# ⭐ Remerciements

Ce projet a été réalisé dans un objectif d'apprentissage et de mise en pratique de :

* C#
* Windows Forms
* SQL Server
* SQL
* ADO.NET
* DataGridView
* Chart
* CSV
* SMTP
* Git
* GitHub

---

# 📌 Statut du projet

🟢 **Projet fonctionnel**

Fonctionnalités principales disponibles :

* ✅ Connexion
* ✅ Gestion des utilisateurs
* ✅ Répartition du salaire
* ✅ Calcul de l'épargne
* ✅ Enregistrement SQL
* ✅ Consultation des données
* ✅ Suppression
* ✅ Graphiques
* ✅ Export CSV
* ✅ Envoi par e-mail
* ✅ Déconnexion

Fonctionnalités encore prévues :

* ⏳ Installateur Windows
* ⏳ Déploiement simplifié
* ⏳ Amélioration de la sécurité
* ⏳ Système de mise à jour

---

Merci!!!

