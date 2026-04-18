# Software Requirements Specification (SRS)

# saasPlug Application

---

## 1. Introduction

### 1.1 Purpose  
The saasPlug Application is a web-based service that allows EV users to find and book chargers from any provider.

### 1.2 Scope  
This application allows users to: 
-Login with Google
- Search the avaliable charging points  
- Book the charging point they want  
- Register for charging providers   
- View analytics of provider
- View saasPlug usage invoice for providers
- View global statistics / analytics from the saasPlug operator 

### 1.3 Definitions, Acronyms, and Abbreviations  
- **EV**: Electric Vehicle.  
  
### 1.4 Overview  
This document outlines functional and non-functional requirements, use cases, and interface specifications for the saasPlug Application.

---

## 2. System Overview

### 2.1 Product Perspective  
SaasPlug is a SaaS (Software as a Service) standalone web application. It uses a database  for user and analytics storage.

### 2.2 Product Functions 
1. Login user with Google.
2. Search fot the avaliable charging points.  
3. Booking the charging point of user preference.  
4. Registration for charging providers.  
5. Display analytics for each provider.  
6. Display saasPlug usage invoice for providers.  
7. Display global statistics / analytics from the saasPlug operator  

### 2.3 User Characteristics  
Primary users include electric vehicle users who need a simple app to find a charging point near them and also charging providers looking for customers . No coding or technical skills are required.

### 2.4 Operating Environment  
- **Frontend**: Modern browsers (Chrome, Firefox, Safari).   
- **Backend**: Server with Node.js,Python and React. 
- **Database**: Docker Containers.  
- **Network**: Internet connectivity required for backend-database communication.  

### 2.5 Assumptions and Dependencies  
- Google OAuth is available for user authentication.

---

## 3. Use cases

### 3.1 Use Case Diagram  
```plantuml
@startuml saasPlug_UseCases
left to right direction
skinparam packageStyle rectangle

actor "EV User" as user
actor "Charging Provider" as provider
actor "saasPlug Operator" as operator

rectangle "saasPlug System" {
    
    package "Identity & Access" {
        usecase "Login with Google" as login_google
        usecase "Register Provider Account" as reg_provider
    }

    package "Charging Operations" {
        usecase "Search Charging Points" as search
        usecase "Reserve Charging Point" as reserve
    }

    package "Provider Dashboard" {
        usecase "View Personal Stats/Analytics" as view_stats
        usecase "Export Usage Data" as export
        usecase "View & Pay saasPlug Invoice" as pay_bill
    }

    package "System Administration" {
        usecase "View Global Analytics" as global_stats
    }
}

' User Relations
user --> login_google
user --> search
user --> reserve

' Provider Relations
provider --> reg_provider
provider --> view_stats
provider --> export
provider --> pay_bill

' Operator Relations
operator --> global_stats

@enduml
```

### 3.2 Use Cases Activity Diagrams

### 3.2.1 Login with Google

- **Activity Diagram**:

  ```plantuml
    @startuml activity_register
    title Activity Diagram: saasPlug Registration & Onboarding

    start
    :User clicks "Login/Sign up with Google";
    :Redirect to Google OAuth;

    if (Google Authentication Successful?) then (yes)
        :Receive Google Identity Token;
        :Auth Service checks if User exists in DB;
  
        if (User already exists?) then (no)
            :Create new User Profile;
            :Assign default Role (EV_USER);
            :Publish "UserRegistered" to Kafka;
        else (yes)
            :Retrieve existing Profile;
        endif

        :Display Home Dashboard;
  
        if (User wants to register as Provider?) then (yes)
            :User selects "Become a Provider";
            :Display Provider Registration Form;
            :User enters Company Details & API Endpoint;
    
            if (Validate Business Info?) then (success)
                :Update User Role to PROVIDER;
                :Create Provider Profile;
                :Link Charger API Endpoint to Account;
                :Publish "ProviderOnboarded" to Kafka;
                :Show Provider Dashboard;
            else (failure)
                :Show Validation Errors;
                stop
            endif
        else (no)
            :Continue as EV User;
            :Show Map/Search Interface;
        endif

        stop
    else (no)
    :Display Auth Error Message;
    stop
    endif

    @enduml
  ```

  ### 3.2.2 Resarvation for Charging

  - **Activity Diagram**:

```plantuml
    @startuml activity_reserve
    title Activity Diagram: saasPlug Reservation Logic

    start
    :User searches for chargers (Map/List);
    :Selects specific Charging Point;
    :User requests "Book Now";

    :System checks local Reservation DB;
    if (Timeslot available locally?) then (no)
    :Show "Already Booked" message;
    stop
    else (yes)
    :Call Provider API for real-time status;

    if (Charger Online & Available?) then (yes)
        fork
        :Lock charger via Provider API;
        fork again
        :Create pending record in Reservation DB;
        end fork

        :Confirm Reservation;
        :Update status to "ACTIVE";
        :Publish "ReservationCreated" to Kafka;
        :Display Success Message;
        stop
        

    else (no)
        :Update local Search DB status to "OFFLINE";
        :Show "Charger Unavailable" message;
        stop
    endif
    endif

    @enduml
 ```

---

## 4. Classes and Entities relations

### 4.1 Class Diagram

```plantuml
    @startuml class_diagram
    title saasPlug API and Data Model Class Diagram

    skinparam classAttributeIconSize 0
    hide circle

    ' --- Data Transfer Objects (JSON Models) ---
    package "Data Models (JSON Objects)" <<Rectangle>> {
    
        class UserDTO {
            + userID: UUID
            + email: String
            + role: String
        }

        class ChargingPointDTO {
            + chargerID: UUID
            + providerID: UUID
            + location: GPSCoords
            + powerKW: int
            + status: String
            + kwhPrice: float
        }

        class ReservationDTO {
            + reservationID: UUID
            + userID: UUID
            + chargerID: UUID
            + startTime: DateTime
            + endTime: DateTime
            + status: String
        }

        class InvoiceDTO {
            + invoiceID: UUID
            + providerID: UUID
            + amount: decimal
            + isPaid: boolean
        }
    }

    ' --- API Interfaces ---
    package "APIs (Service Controllers)" <<Rectangle>> {

        interface "IAuthAPI" {
            + Login(): UserDTO
            + Register(): UserDTO
        }

        interface "ISearchAPI" {
            + getChargingPoints(location: GPS, radius: int): List<ChargingPointDTO>
            + getChargingPointDetails(id: int): ChargingPointDTO
        }

        interface "IReservationAPI" {
            + createBooking(userID: int, chargingPointID: int): ReservationDTO
            + getUserBookings(userID: int): List<ReservationDTO>
        }

        interface "IBillingAPI" {
            + getInvoice(providerID: int): InvoiceDTO
            + processPayment(invoiceID: int): PaymentStatus
        }

        interface "IStatisticsAPI" {
            + getReservations(providerID: int, startDate: string, endDate: string): List<ReservationDTO>
            + getInvoices(providerID: int, startDate: string, endDate: string): List<InvoiceDTO>
        }
    }

    ' --- Relationships (Usage) ---
    IAuthAPI ..> UserDTO : "returns"
    ISearchAPI ..> ChargingPointDTO : "returns"
    IReservationAPI ..> ReservationDTO : "returns"
    IBillingAPI ..> InvoiceDTO : "returns"

    @enduml
 ```

### 4.2 ER Diagram

```plantuml
    @startuml er_diagram
    title saasPlug Partitioned ER Diagram (Microservices)

    ' Formatting
    hide circle
    skinparam Linetype ortho

    package "Authorization DB" #E3F2FD {
    entity "User" as user {
        * userID : UUID <<PK>>
        --
        email : String
        googleID : String
        role : Enum(EV_OWNER, PROVIDER, ADMIN)
        createdAt : DateTime
    }
    }

    package "Search DB" #F1F8E9 {
    
    entity "ChargingPoint" as charger {
        * chargerID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
        latitude: float
        longitude: float
        powerKW : Integer
        status : Enum(AVAILABLE, BUSY)
        capacity: Integer
        kwhPrice: Float
    }

    entity "Provider" as provider {
        * providerID : UUID <<PK>>
        --
        name : String
        apiURL : String
    }

    }

    package "Reservation DB" #FFF3E0 {
    entity "Reservation" as res {
        * resID : UUID <<PK>>
        --
        userID : UUID <<FK>>
        chargingPointID : UUID <<FK>>
        startTime : DateTime
        endTime : DateTime
        status : String
    }

    entity "provider" as provider {
        * providerID : UUID <<PK>>
        --
        name: string
        apiURL : String
    }

    entity "ChargingPoint" as charger {
        * chargerID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
    }
    }

    package "Billing DB" #F3E5F5 {
    entity "Invoice" as invoice {
        * invoiceID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
        amount : Decimal
        issuedAt : DateTime
        paid : bool
    }

    entity "PaidReservations" as paid {
        * PaidReservations : UUID <<PK>>
        --
        invoiceID : UUID <<FK>>
        reservationID: UUID <<FK>>
    }

    entity "provider" as provider {
        * providerID : UUID <<PK>>
        --
        name: string
    }

    entity "Reservation" as res {
        * resID : UUID <<PK>>
        --
        userID : UUID <<FK>>
        chargingPointID : UUID <<FK>>
        status : String
    }

    }

    package "Statistics DB" #EFEBE9 {
        entity "User" as user {
        * userID : UUID <<PK>>
        --
        createdAt : DateTime
    }

    entity "ChargingPoint" as charger {
        * chargerID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
        latitude: float
        longitude: float
    }

    entity "Provider" as provider {
        * providerID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
        name : String
    }

    entity "Reservation" as res {
        * resID : UUID <<PK>>
        --
        userID : UUID <<FK>>
        chargingPointID : UUID <<FK>>
        startTime : DateTime
        endTime : DateTime
        status : String
    }

    entity "Bill" as bill {
        * billID : UUID <<PK>>
        --
        providerID : UUID <<FK>>
        amount : Decimal
        issuedAt : DateTime
    }
        
    }

    @enduml
```

## 5. Component Diagram 

```plantuml
    @startuml component_diagram
    title saasPlug Component Diagram

    skinparam componentStyle uml2

    ' --- External Actors ---
    package "Web/Mobile App" {
        [UI]
    }

    package "Google Identity Provider" {
        () "OAuth API" as Google_IF
    }

    package "Provider APIs (External)" {
        () "Charger Info API" as Charger_IF
    }

    package "Payment Gateway (External)" {
        () "Payment API" as Payment_IF
    }

    ' --- API Gateway ---
    package "saasPlug Platform" {
  
        [API Gateway] <<Entry Point>> as Gateway
  
        ' --- Synchronous (REST/gRPC) Interfaces (Lollipops) ---
        Gateway -up-( "REST API" : HTTP
        [UI] -up- "REST API" 

        () "Auth IF" as Auth_IF
        () "Search IF" as Search_IF
        () "Reservation IF" as Res_IF
        () "Billing IF" as Bill_IF
        () "Stats IF" as Stat_IF

        Gateway -[hidden]- Auth_IF 
        Gateway -[hidden]- Search_IF
        Gateway -[hidden]- Res_IF
        Gateway -[hidden]- Bill_IF
        Gateway -[hidden]- Stat_IF

        ' --- Microservices ---
  
        package "Authorization Service" {
        [Auth Component] as AuthComp
        [AuthComp] - Google_IF
    }
  
    package "Search Service" {
        [Search Component] as SearchComp
        [SearchComp] - Charger_IF
    }
  
    package "Reservation Service" {
        [Reservation Component] as ResComp
    }
  
    package "Billing Service" {
        [Billing Component] as BillComp
        [BillComp] - Payment_IF
    }
  
    package "Statistics Service" {
        [Statistics Component] as StatComp
    }

    ' --- Connect Gateway to Microservices (Internal REST/gRPC) ---
    Auth_IF -down-- [AuthComp]
    Gateway --( Auth_IF

    Search_IF -down-- [SearchComp]
    Gateway --( Search_IF

    Res_IF -down-- [ResComp]
    Gateway --( Res_IF

    Bill_IF -down-- [BillComp]
    Gateway --( Bill_IF

    Stat_IF -down-- [StatComp]
    Gateway --( Stat_IF

    ' --- Message Broker (Kafka) ---
    [Apache Kafka] <<Message Broker>> as Kafka

    ' --- Asynchronous (Pub/Sub) Interfaces (Topics) ---
    () "User Events Topic" as UserTopic
    () "Reservation Events Topic" as ResTopic
    () "Billing Events Topic" as BillTopic

    UserTopic -up- Kafka
    ResTopic -up- Kafka
    BillTopic -up- Kafka

    ' --- Connect Microservices to Kafka Topics (Pub/Sub) ---
  
    ' Producers
    [AuthComp] -down-- UserTopic : Publishes
    [ResComp] -down-- ResTopic : Publishes
    [BillComp] -down- BillTopic : Publishes

    ' Consumers (Sockets/Lollipops)
    [BillComp] -down--( UserTopic : Subscribes
    [StatComp] -down--( UserTopic : Subscribes
  
    [BillComp] -down--( ResTopic : Subscribes
    [StatComp] -down--( ResTopic : Subscribes

    [StatComp] -down--( BillTopic : Subscribes

    @enduml
```

## 6.Deployment

```plantuml
    @startuml deplyment_diagram
    title saasPlug Deployment Architecture

    skinparam componentStyle uml2

    node "User Device" as Client {
        artifact "Web/Mobile App" <<UI>> as UI
    }

    node "Cloud Infrastructure (Docker Engine)" {

        ' Entry Point
        node "Network: Frontend-Net" {
            [API Gateway] as Gateway <<Entry Point>>
        }

        ' Service Layer
        node "Network: Backend-Net" {
            
            package "Auth Service" {
                port "p1" as authPort
                [Authorization Service] as AuthService
                database "Auth DB" as AuthDB
            }

            package "Search Service" {
                port "p2" as searchPort
                [Search Service] as SearchService
                database "Search DB" as SearchDB
            }

            package "Reservation Service" {
                port "p3" as resPort
                [Reservation Service] as ResService
                database "Reservation DB" as ResDB
            }

            package "Billing Service" {
                port "p4" as billPort
                [Billing Service] as BillService
                database "Billing DB" as BillDB
            }

            package "Statistics Service" {
                port "p5" as statPort
                [Statistics Service] as StatService
                database "Stats DB" as StatDB
            }
        }

        ' Pub/Sub Layer moved to bottom
        node "Network: Event-Bus" {
            queue "Apache Kafka" as Kafka
        }
    }

    ' External Systems moved to sides to maintain verticality
    cloud "Google OAuth" as Google
    cloud "Provider APIs" as ProviderAPI

    ' --- Connections ---

    UI -down-> Gateway : HTTPS

    ' Routing through Ports
    Gateway -down-> authPort
    Gateway -down-> searchPort
    Gateway -down-> resPort
    Gateway -down-> billPort
    Gateway -down-> statPort

    ' Services to DBs
    AuthService -down-> AuthDB
    SearchService -down-> SearchDB
    ResService -down-> ResDB
    BillService -down-> BillDB
    StatService -down-> StatDB

    ' Pub/Sub Flow (Vertical)
    AuthService -down-> Kafka
    SearchService -down-> Kafka
    ResService -down-> Kafka
    BillService -down-> Kafka
    StatService -down-> Kafka

    ' External Integrations
    AuthService -left- Google
    SearchService -right- ProviderAPI

    @enduml
```

## 7.Sequence Diagram 

### 7.1 Payment 

```plantuml
    @startuml sequence_payment
    title Sequence Diagram: Provider Invoice Retrieval and Payment

    autonumber
    actor "Station Provider" as Provider
    participant "API Gateway" as Gateway
    participant "Billing Service" as BillService
    database "Billing DB" as BillDB
    participant "Payment Gateway" as Stripe <<External>>
    queue "Apache Kafka" as Kafka

    == Invoice Retrieval ==

    Provider -> Gateway : GET /invoices/current
    activate Gateway

    Gateway -> BillService : Request Current Invoice
    activate BillService

    BillService -> BillDB : Generate/Retrieve Invoice Record
    activate BillDB
    BillDB --> BillService : InvoiceDTO
    deactivate BillDB

    BillService --> Gateway : 200 OK (Invoice Details)
    deactivate BillService

    Gateway --> Provider : Display Invoice & "Pay Now" Button
    deactivate Gateway

    == Payment Process ==

    Provider -> Gateway : POST /payments/{invoiceID}
    activate Gateway

    Gateway -> BillService : Process Payment
    activate BillService

    ' External Payment Integration
    BillService -> Stripe : Initiate Transaction (Amount, CardDetails)
    activate Stripe
    Stripe --> BillService : 200 OK (TransactionID: XYZ)
    deactivate Stripe

    ' Persistence
    BillService -> BillDB : Update Invoice Status (PAID)
    activate BillDB
    BillDB --> BillService : Updated
    deactivate BillDB

    ' Notify System of Revenue
    BillService -> Kafka : Publish "PaymentCompleted" Event
    activate Kafka
    deactivate Kafka

    BillService --> Gateway : 200 OK (Receipt)
    deactivate BillService

    Gateway --> Provider : Show Success Message
    deactivate Gateway

    @endluml
```

### 7.2 Resevation

```plantuml
    @startuml sequence_reservation
    title Sequence Diagram: Charging Point Reservation (with Provider API)

    autonumber
    actor "EV User" as User
    participant "API Gateway" as Gateway
    participant "Reservation Service" as ResService
    database "Reservation DB" as ResDB
    participant "Provider External API" as Provider
    queue "Apache Kafka" as Kafka

    User -> Gateway : POST /reservations\n(chargerID, timeslot)
    activate Gateway

    ' Core Reservation Logic
    Gateway -> ResService : Process Reservation
    activate ResService

    ' External Communication
    ResService -> Provider : POST /remote-secure-lock (chargerID)
    activate Provider
    note right: Notifying the hardware owner
    Provider --> ResService : 200 OK (Locked)
    deactivate Provider

    ' Persistence
    ResService -> ResDB : Save Reservation (Status: ACTIVE)
    activate ResDB
    ResDB --> ResService : Saved
    deactivate ResDB

    ' Async Event Production
    ResService -> Kafka : Publish "ReservationCreated" Event
    activate Kafka
    deactivate Kafka

    ResService --> Gateway : 201 Created (ResDTO)
    deactivate ResService

    Gateway --> User : Show Confirmation & QR Code
    deactivate Gateway

    @enduml
```
