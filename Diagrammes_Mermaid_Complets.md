# DIAGRAMMES MERMAID - PLATEFORME IMMOBILIÈRE

## 1. DIAGRAMME ENTITÉ-RELATION (ER)

```mermaid
erDiagram
    USERS ||--o{ AGENCIES : owns
    USERS ||--o{ PARTNERS : "is a"
    USERS ||--o{ FAVORITES : creates
    AGENCIES ||--o{ LISTINGS : publishes
    AGENCIES ||--o{ PARTNERS : invites
    LISTINGS ||--o{ PHOTOS : contains
    LISTINGS ||--o{ CONTACTS : receives
    AGENCIES ||--o{ SUBSCRIPTIONS : has
    SUBSCRIPTIONS ||--o{ PAYMENTS : processes
    USERS ||--o{ PAYMENTS : makes

    USERS {
        uuid id PK
        string email UK
        string password_hash
        string firstName
        string lastName
        string phone
        enum role "admin, agency, partner, buyer"
        enum status "active, suspended, deleted"
        boolean email_verified
        timestamp created_at
    }

    AGENCIES {
        uuid id PK
        uuid user_id FK
        string name
        text description
        string logo_url
        string phone
        string email
        string city
        string quartier
        string address
        decimal lat
        decimal lng
        uuid subscription_id FK
        enum status "active, suspended"
        decimal rating
        int total_listings
        timestamp created_at
    }

    LISTINGS {
        uuid id PK
        uuid agency_id FK
        uuid partner_id FK
        string title
        text description
        decimal price
        string currency "XOF"
        string category
        int bedrooms
        int bathrooms
        decimal area
        string city
        string quartier
        string address
        decimal lat
        decimal lng
        enum status "available, reserved, sold, suspended"
        boolean is_premium
        int views
        int favorites
        int contacts
        timestamp featured_until
        timestamp created_at
        timestamp published_at
    }

    PHOTOS {
        uuid id PK
        uuid listing_id FK
        string url
        string cloudinary_id
        boolean is_primary
        int order_index
        timestamp created_at
    }

    CONTACTS {
        uuid id PK
        uuid listing_id FK
        string name
        string email
        string phone
        text message
        enum contact_method "whatsapp, call, form"
        enum status "new, contacted, closed"
        timestamp created_at
    }

    FAVORITES {
        uuid id PK
        uuid user_id FK
        uuid listing_id FK
        timestamp created_at
    }

    SUBSCRIPTIONS {
        uuid id PK
        uuid agency_id FK
        enum plan "free, standard, premium"
        decimal price
        int max_listings
        timestamp renewal_date
        enum status "active, expired, cancelled"
        timestamp created_at
    }

    PAYMENTS {
        uuid id PK
        uuid subscription_id FK
        uuid user_id FK
        decimal amount
        string currency
        enum payment_method "card, wave, orange_money, paypal"
        string transaction_id UK
        enum status "pending, completed, failed"
        timestamp created_at
    }

    PARTNERS {
        uuid id PK
        uuid agency_id FK
        uuid user_id FK
        enum status "invited, active, inactive"
        boolean require_validation
        timestamp invited_at
        timestamp accepted_at
    }
```

---

## 2. DIAGRAMME UML - USE CASES

```mermaid
graph TB
    subgraph Acteurs["Acteurs du Système"]
        Visiteur["👤 Visiteur"]
        Acheteur["👤 Acheteur"]
        Agence["🏢 Agence"]
        Partenaire["🤝 Partenaire"]
        Admin["🔐 Admin"]
    end

    subgraph UseCases["Cas d'Usage Principaux"]
        UC1["Consulter Annonces"]
        UC2["Filtrer & Rechercher"]
        UC3["Voir Détail Annonce"]
        UC4["Ajouter aux Favoris"]
        UC5["Contacter Vendeur"]
        UC6["Créer Compte"]
        UC7["Se Connecter"]
        UC8["Créer Annonce"]
        UC9["Modifier Annonce"]
        UC10["Supprimer Annonce"]
        UC11["Publier Annonce"]
        UC12["Inviter Partenaire"]
        UC13["Gérer Partenaires"]
        UC14["Consulter Dashboard"]
        UC15["Gérer Abonnement"]
        UC16["Effectuer Paiement"]
        UC17["Modérer Contenu"]
        UC18["Gérer Utilisateurs"]
        UC19["Consulter Statistiques"]
        UC20["Gérer Support"]
    end

    Visiteur --> UC1
    Visiteur --> UC2
    Visiteur --> UC3
    Visiteur --> UC5

    Acheteur --> UC6
    Acheteur --> UC7
    Acheteur --> UC1
    Acheteur --> UC2
    Acheteur --> UC3
    Acheteur --> UC4
    Acheteur --> UC5

    Agence --> UC8
    Agence --> UC9
    Agence --> UC10
    Agence --> UC11
    Agence --> UC12
    Agence --> UC13
    Agence --> UC14
    Agence --> UC15
    Agence --> UC16

    Partenaire --> UC8
    Partenaire --> UC9
    Partenaire --> UC10
    Partenaire --> UC14

    Admin --> UC17
    Admin --> UC18
    Admin --> UC19
    Admin --> UC20
```

---

## 3. DIAGRAMME DE FLUX - PUBLICATION D'ANNONCE

```mermaid
sequenceDiagram
    actor User as Agence/Partenaire
    participant App as Application
    participant API as Backend API
    participant DB as PostgreSQL
    participant Cloud as Cloudinary
    participant Email as Email Service

    User->>App: Clique "Créer Annonce"
    App->>User: Affiche formulaire multi-étapes
    User->>App: Rempli formulaire (infos, localisation, caractéristiques)
    User->>App: Upload photos
    App->>Cloud: Envoie photos
    Cloud->>Cloud: Traite et stocke images
    Cloud-->>App: Retourne URLs
    User->>App: Confirme et valide
    App->>API: POST /listings (auth token)
    API->>DB: Insère listing avec statut "pending"
    DB-->>API: Listing créé avec ID
    API->>Email: Envoie notification modérateur
    Email->>Email: Notification envoyée
    API-->>App: 201 Created
    App->>User: Succès! Annonce en attente de validation
    
    Note over API,DB: Modération (Agence ou Admin selon config)
    Admin->>API: Valide annonce
    API->>DB: Update listing status = "published"
    API->>Email: Notifie Agence/Partenaire
    Email-->>User: Email: Annonce approuvée!
    User->>App: Voir annonce en ligne
```

---

## 4. DIAGRAMME DE FLUX - RECHERCHE ET CONTACT

```mermaid
sequenceDiagram
    actor User as Acheteur/Visiteur
    participant App as Application Frontend
    participant API as Backend API
    participant DB as PostgreSQL
    participant Maps as Google Maps API

    User->>App: Accède à la plateforme
    App->>API: GET /listings (défaut: 20 annonces)
    API->>DB: Requête annonces avec filtres
    DB-->>API: Retourne annonces
    API-->>App: JSON annonces
    App->>App: Affiche grille annonces

    User->>App: Filtre: Dakar, 2 chambres, 40-60M XOF
    App->>API: GET /listings?city=Dakar&bedrooms=2&minPrice=40M&maxPrice=60M
    API->>DB: Requête filtrée
    DB-->>API: Retourne résultats
    API-->>App: JSON annonces filtrées
    App->>App: Affiche résultats

    User->>App: Clique sur annonce
    App->>API: GET /listings/:id (détails complets)
    API->>DB: Requête détail + photos
    DB-->>API: Listing complet
    API->>API: Incrémente compteur vues
    API-->>App: JSON + photos
    App->>Maps: Charge carte de localisation
    Maps-->>App: Affiche épingle
    App->>User: Affiche page détail

    User->>App: Clique "Contacter via WhatsApp"
    App->>App: Ouvre WhatsApp Web/Mobile
    User->>User: Envoie message

    Alt Formulaire de Contact
        User->>App: Remplit formulaire
        App->>API: POST /listings/:id/contacts
        API->>DB: Insère contact
        API->>Email: Notifie Agence
        Email-->>User: Confirmation
    End
```

---

## 5. DIAGRAMME DE FLUX - ABONNEMENT & PAIEMENT

```mermaid
sequenceDiagram
    actor Agency as Agence
    participant App as Application
    participant API as Backend API
    participant Stripe as Stripe/Wave API
    participant DB as PostgreSQL
    participant Email as Email Service

    Agency->>App: Accède au Dashboard
    App->>API: GET /subscriptions (auth)
    API->>DB: Récupère abonnement actuel
    DB-->>API: Données abonnement
    API-->>App: JSON abonnement
    App->>Agency: Affiche plan actuel "Gratuit"

    Agency->>App: Clique "Upgrade vers Standard"
    App->>App: Affiche page de paiement
    Agency->>App: Sélectionne "Carte Bancaire"
    Agency->>App: Remplit détails paiement
    App->>API: POST /payments/process
    API->>Stripe: Crée charge avec amount
    Stripe->>Stripe: Valide carte 3D Secure
    Stripe-->>API: Succès ou Erreur
    
    Alt Paiement Réussi
        API->>DB: Update subscription = "standard"
        API->>DB: Insert payment record
        DB-->>API: OK
        API->>Email: Envoie facture + confirmation
        Email-->>Agency: Email reçu
        API-->>App: 201 Payment Created
        App->>Agency: Succès! Upgrade en cours
        App->>API: GET /subscriptions
        API-->>App: Retourne "standard"
        App->>Agency: Dashboard mise à jour (20 annonces max, boost 1x/mois)
    End

    Alt Paiement Échoué
        API-->>App: 400 Payment Failed
        App->>Agency: Erreur - Carte refusée
        Agency->>App: Réessaie avec Wave
        App->>API: POST /payments/process (wave)
        API->>Stripe: Crée charge Wave
        Stripe-->>API: Succès
        API->>DB: Update subscription
        API->>Email: Facture
        Email-->>Agency: Confirmation Wave
    End
```

---

## 6. DIAGRAMME D'ARCHITECTURE SYSTÈME

```mermaid
graph TB
    subgraph Client["Frontend (Client)"]
        Web["🌐 Application Web<br/>Next.js + React<br/>Tailwind CSS"]
        Mobile["📱 Application Mobile<br/>(Future)"]
    end

    subgraph API["API Layer (Backend)"]
        Auth["🔐 Auth Service<br/>JWT + 2FA"]
        Listings["📋 Listings Service<br/>CRUD + Search"]
        Agencies["🏢 Agencies Service<br/>Gestion Agences"]
        Payments["💳 Payments Service<br/>Intégration Stripe/Wave"]
        Notifications["🔔 Notifications Service<br/>Email + In-app"]
        Admin["🔑 Admin Service<br/>Modération"]
    end

    subgraph Data["Data Layer"]
        DB["🗄️ PostgreSQL<br/>Base Principale"]
        Cache["⚡ Redis<br/>Cache + Sessions"]
    end

    subgraph External["Services Externes"]
        Cloud["☁️ Cloudinary<br/>Stockage Images"]
        Maps["🗺️ Google Maps<br/>Géolocalisation"]
        Email["📧 SendGrid<br/>Emails"]
        Payments_Gateway["💰 Stripe/Wave<br/>Paiements"]
    end

    Web -->|HTTPS| Auth
    Web -->|HTTPS| Listings
    Web -->|HTTPS| Agencies
    Web -->|HTTPS| Payments
    Mobile -->|HTTPS| Auth
    Mobile -->|HTTPS| Listings

    Auth -->|Read/Write| DB
    Auth -->|Store Session| Cache
    Listings -->|Read/Write| DB
    Listings -->|Cache| Cache
    Agencies -->|Read/Write| DB
    Payments -->|Read/Write| DB
    Notifications -->|Read| DB
    Admin -->|Read/Write| DB

    Listings -->|Upload| Cloud
    Web -->|Display| Cloud
    Listings -->|Geocoding| Maps
    Notifications -->|Send| Email
    Payments -->|API| Payments_Gateway

    style Web fill:#e1f5ff
    style Mobile fill:#e1f5ff
    style Auth fill:#fff3e0
    style Listings fill:#fff3e0
    style DB fill:#f3e5f5
    style Cache fill:#f3e5f5
    style Cloud fill:#e8f5e9
    style Maps fill:#e8f5e9
```

---

## 7. DIAGRAMME DE RÔLES & PERMISSIONS

```mermaid
graph TD
    subgraph Roles["Rôles et Permissions"]
        Admin["🔐 ADMINISTRATEUR<br/>━━━━━━━━━<br/>✓ Gérer tous utilisateurs<br/>✓ Gérer toutes agences<br/>✓ Modérer annonces<br/>✓ Gérer paiements<br/>✓ Voir statistiques globales<br/>✓ Support client"]
        
        Agency["🏢 AGENCE<br/>━━━━━━━━━<br/>✓ Créer annonces<br/>✓ Modifier ses annonces<br/>✓ Supprimer ses annonces<br/>✓ Inviter partenaires<br/>✓ Gérer partenaires<br/>✓ Voir son dashboard<br/>✓ Gérer abonnement<br/>✓ Voir ses statistiques"]
        
        Partner["🤝 PARTENAIRE<br/>━━━━━━━━━<br/>✓ Créer annonces<br/>✓ Modifier ses annonces<br/>✓ Supprimer ses annonces<br/>✓ Voir ses statistiques<br/>✓ Gérer profil"]
        
        Buyer["👤 ACHETEUR<br/>━━━━━━━━━<br/>✓ Rechercher annonces<br/>✓ Filtrer et trier<br/>✓ Voir détails<br/>✓ Ajouter favoris<br/>✓ Contacter vendeur<br/>✓ Recevoir alertes<br/>✓ Gérer favoris"]
        
        Visitor["👁️ VISITEUR<br/>━━━━━━━━━<br/>✓ Consulter annonces<br/>✓ Utiliser filtres<br/>✓ Voir détails<br/>✓ Contacter agence"]
    end

    style Admin fill:#ffebee
    style Agency fill:#e8f5e9
    style Partner fill:#e3f2fd
    style Buyer fill:#f3e5f5
    style Visitor fill:#fff3e0
```

---

## 8. DIAGRAMME D'ÉTATS - ANNONCE

```mermaid
stateDiagram-v2
    [*] --> Brouillon
    Brouillon --> Pendente: Soumettre pour publication
    Pendente --> Disponible: Approbation admin/agence
    Pendente --> Rejetée: Refus modération
    Rejetée --> Brouillon: Modifier et resoummettre
    Disponible --> Réservée: Acheteur réserve
    Disponible --> Vendue: Transaction complétée
    Disponible --> Suspendue: Admin suspend
    Réservée --> Vendue: Transaction finalisée
    Réservée --> Disponible: Réservation annulée
    Suspendue --> Disponible: Admin réactive
    Vendue --> [*]
    Rejetée --> [*]
```

---

## 9. DIAGRAMME DE CLASSES (Simplifié)

```mermaid
classDiagram
    class User {
        -id: UUID
        -email: String
        -password_hash: String
        -firstName: String
        -lastName: String
        -phone: String
        -role: Enum
        -status: Enum
        +register()
        +login()
        +logout()
        +updateProfile()
    }

    class Agency {
        -id: UUID
        -user_id: UUID
        -name: String
        -logo_url: String
        -subscription_id: UUID
        -rating: Decimal
        +createListing()
        +updateListing()
        +deleteListing()
        +invitePartner()
        +getDashboard()
        +getStatistics()
    }

    class Listing {
        -id: UUID
        -agency_id: UUID
        -title: String
        -description: String
        -price: Decimal
        -category: String
        -bedrooms: Int
        -bathrooms: Int
        -area: Decimal
        -status: Enum
        -is_premium: Boolean
        -views: Int
        -created_at: Timestamp
        +publish()
        +update()
        +delete()
        +increaseViews()
    }

    class Photo {
        -id: UUID
        -listing_id: UUID
        -url: String
        -cloudinary_id: String
        -is_primary: Boolean
        +upload()
        +delete()
    }

    class Contact {
        -id: UUID
        -listing_id: UUID
        -name: String
        -email: String
        -phone: String
        -message: String
        -contact_method: Enum
        +create()
        +markAsContacted()
    }

    class Subscription {
        -id: UUID
        -agency_id: UUID
        -plan: Enum
        -price: Decimal
        -max_listings: Int
        -renewal_date: Timestamp
        +upgrade()
        +downgrade()
        +renew()
        +cancel()
    }

    class Payment {
        -id: UUID
        -subscription_id: UUID
        -amount: Decimal
        -payment_method: Enum
        -transaction_id: String
        -status: Enum
        +process()
        +refund()
    }

    User "1" --> "*" Agency
    User "1" --> "*" Contact
    Agency "1" --> "*" Listing
    Agency "1" --> "*" Subscription
    Listing "1" --> "*" Photo
    Listing "1" --> "*" Contact
    Subscription "1" --> "*" Payment
```

---

