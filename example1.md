# Software Requirements Specification (SRS)

# MyCharts Application

---

## 1. Introduction

### 1.1 Purpose  
The MyCharts Application is a web-based service designed to enable users with minimal technical expertise to generate, manage, and download charts in various formats. It simplifies chart creation by providing templates, supporting data uploads, and integrating the Highcharts library for visualization.

### 1.2 Scope  
This application allows users to:  
- Download CSV templates for supported chart types.  
- Upload CSV files to generate charts.  
- Save and download charts in PDF, PNG, SVG, and HTML formats.  
- Purchase quotas for chart creation.
- View and download generated charts.

### 1.3 Definitions, Acronyms, and Abbreviations  
- **CSV**: Comma-Separated Values file format.  
- **Quota**: A limit on the number of charts a user can create.  
- **Highcharts**: JavaScript library used for chart rendering.  

### 1.4 References  
- Highcharts Library: https://www.highcharts.com  
- Google Authentication API: https://developers.google.com/identity  
- CSV Format Specification: https://tools.ietf.org/html/rfc4180  

### 1.5 Overview  
This document outlines functional and non-functional requirements, use cases, and interface specifications for the MyCharts Application.

---

## 2. System Overview

### 2.1 Product Perspective  
MyCharts is a SaaS (Software as a Service) standalone web application. It uses a database (e.g., PostgreSQL, MySQL or/and MongoDB) for user and chart storage and integrates Google OAuth for authentication.

### 2.2 Product Functions  
1. User authentication via Google accounts.  
2. Download CSV templates for chart types.  
3. Upload CSV files to generate charts.  
4. Save charts in PDF, PNG, SVG, and HTML formats.  
5. Display and manage user-generated charts.  
6. Charge quotas for chart creation.  

### 2.3 User Characteristics  
Primary users include educators, students, and professionals needing simple chart generation. No coding or technical skills are required.

### 2.4 Operating Environment  
- **Frontend**: Modern browsers (Chrome, Firefox, Safari).  
- **Backend**: Server with Node.js or Python runtime.  
- **Database**: PostgreSQL, MySQL, MongoDB etc.  
- **Network**: Internet connectivity required for backend-database communication.  

### 2.5 Assumptions and Dependencies  
- Google OAuth is available for user authentication.  
- Highcharts library is accessible for chart rendering.  
- Users comply with CSV formatting guidelines.  

---

## 3. Functional Requirements

### 3.1 Functional Requirements Specification  
1. Authenticate users via Google accounts.  
2. Provide downloadable CSV templates for 3 chart types (Basic line, Line with annotations, Basic column).  
3. Validate uploaded CSV files against formatting rules.  
4. Generate charts using Highcharts libraries.  
5. Save charts to the server in PDF, PNG, SVG, and HTML formats.  
6. Display a dashboard of user-generated charts.  
7. Charge quotas for chart creation.  
8. Allow users to delete or download charts. 
9. Purchase quotas. 

### 3.2 Use Cases  

#### 3.2.1 Use Case Diagram  
```plantuml
@startuml
left to right direction
actor "User" as user
rectangle MyCharts {
  usecase "Login via Google" as Login
  usecase "Download Template" as Download
  usecase "Upload CSV and Generate Chart" as Upload
  usecase "View/Download Charts from History" as History
  usecase "Buy Quotas" as Quota
  usecase "View Profile" as Profile
}
user --> Login
user --> Download
user --> Upload
user --> History
user --> Quota
user --> Profile
@enduml
```

#### 3.2.2 Use Case 1: Login via Google  
- **Actors**: User  
- **Preconditions**:  
  a) User has a valid Google account.  
- **Main Flow**:  
  1. User clicks the "Login with Google" button on the application's login page.  
  2. System redirects the user to Google's OAuth authentication interface.  
  3. User enters Google credentials and grants permission for account access.  
  4. System validates credentials and retrieves user profile data (email, name).  
  5. System redirects the user to the dashboard and displays a welcome message.  
- **Alternate Flows**:  
  - **Invalid Credentials**: If authentication fails, the system displays "Login Failed: Invalid Google Account."  
- **Postconditions**:  
  a) User is authenticated.  
  b) Dashboard with chart management features becomes accessible.  

- **Activity Diagram**:
  ```plantuml
    start
    :User clicks "Login with Google";
    :Redirect to Google OAuth;
    if (Valid credentials?) then (yes)
        :Retrieve user profile data;
        :Redirect to dashboard;
        :Display welcome message;
    else (no)
        :Display "Login Failed: Invalid Google Account";
    endif
    stop
  ```

#### 3.2.3 Use Case 2: Download CSV Template  
- **Actors**: User  
- **Preconditions**:  
  a) User is logged in.  
- **Main Flow**:  
  1. User navigates to the "Create Chart" section and selects a chart type from a dropdown menu.  
  2. System displays the 3 supported chart types.  
  3. User clicks "Download Template" for the selected chart type.  
  4. System generates a CSV file with:  
     - Predefined column headers matching the chart type.  
     - Example data rows.  
     - Formatting guidelines in a comment section.  
  5. CSV file is downloaded to the user's device.   
- **Postconditions**:  
  a) CSV template is saved locally for data entry.

- **Activity Diagram**:
  ```plantuml
    start
    :User selects chart type from dropdown;
    if (Chart type available?) then (yes)
    :Generate CSV template (headers + examples + guidelines);
    :Download CSV to device;
    else (no)
    :Display "Chart type unavailable";
    endif
    stop
  ```

#### 3.2.4 Use Case 3: Upload CSV and Generate Chart  
- **Actors**: User  
- **Preconditions**:  
  a) User has a valid CSV file formatted per the template.  
  b) User has remaining quota for chart creation.  
- **Main Flow**:  
  1. User navigates to the "Generate Chart" section and uploads a CSV file.  
  2. System performs validation checks:  
     - Column headers match the selected chart type.  
     - Data types are correct (e.g., numeric values for axes).  
     - No empty mandatory fields.  
  3. System renders a preview of the chart using Highcharts.  
  4. User confirms the preview.  
  5. System saves the chart to the server in all supported formats (PDF, PNG, SVG, HTML).  
  6. User's quota is decremented by 1.  
- **Alternate Flows**:  
  - **Invalid CSV**: System highlights errors (e.g., "Column 'Revenue' missing") and blocks submission.  
  - **Quota Exhausted**: System displays "Quota Limit Reached: Upgrade to create more charts."  
- **Postconditions**:  
  a) Chart is stored on the server.  
  b) Chart appears in the user's dashboard.  

- **Activity Diagram**:
    ```plantuml
    start
    :User uploads CSV file;
    partition Validation {
    if (Headers match chart type?) then (yes)
        if (Data types valid?) then (yes)
        if (Mandatory fields filled?) then (yes)
            if (Quota available?) then (yes)
            :Render chart preview;
            else (no)
            :Display "Quota Exhausted";
            stop
            endif
        else (no)
            :Highlight "Mandatory fields missing";
            stop
        endif
        else (no)
        :Highlight "Invalid data types";
        stop
        endif
    else (no)
        :Highlight "Column headers mismatch";
        stop
    endif
    }
    :User confirms preview;
    :Save chart (PDF/PNG/SVG/HTML);
    :Decrement quota by 1;
    stop
    ```

#### 3.2.5 Use Case 4: View/Download Chart from History  
- **Actors**: User  
- **Preconditions**:  
  a) User is logged in.  
  b) At least one chart has been previously generated.  
- **Main Flow**:  
  1. User navigates to the "Chart History" section in the dashboard.  
  2. System displays a list of all previously generated charts with timestamps.  
  3. User selects a chart from the list.  
  4. System displays a preview and provides download options (PDF, PNG, SVG, HTML).  
  5. User selects a format to download the chart.  
- **Alternate Flow**:  
  - If no charts exist, the system displays a message: "No charts found in history."  
- **Postconditions**:  
  a) The selected chart is downloaded in the chosen format.  

- **Activity Diagram**:
    ```plantuml
    start
    :User navigates to Chart History;
    if (Charts exist?) then (yes)
    :Display list with charts;
    :User selects a chart;
    :Show preview and download options;
    :User selects format;
    :Download chart;
    else (no)
    :Display "No charts found";
    endif
    stop
    ```

#### 3.2.6 Use Case 5: Purchase Additional Quotas  
- **Actors**: User  
- **Preconditions**:  
  a) User is logged in.  
  b) Payment gateway integration is operational.  
- **Main Flow**:  
  1. User navigates to the "Quota Management" section in the dashboard.  
  2. System displays available quota packages (e.g., "10 charts: $1", "50 charts: $3").  
  3. User selects a quota package and confirms the purchase.  
  4. System redirects the user to a secure payment gateway (e.g., Stripe/PayPal).  
  5. User completes the payment process.  
  6. System validates the payment and increases the user's quota limit accordingly.  
  7. System sends a confirmation email with a receipt.  
- **Alternate Flows**:  
  - **Payment Failure**: If payment fails, the system retains the original quota and notifies the user.  
  - **Partial Payment**: System cancels the transaction and restores the initial quota state.  
- **Postconditions**:  
  a) The user's quota limit is updated.  
  b) A transaction record is stored in the system.

- **Activity Diagram**:
    ```plantuml
    start
    :User navigates to Quota Management;
    :Display quota packages;
    :User selects package;
    :Redirect to payment gateway;
    if (Payment successful?) then (yes)
    :Increase user quota;
    :Send confirmation email;
    :Store transaction record;
    else (no)
    :Display "Payment Failed";
    endif
    stop
    ```

#### 3.2.7 Use Case 6: View My Profile  
- **Actors**: User  
- **Preconditions**:  
  a) User is logged in.  
- **Main Flow**:  
  1. User clicks the "My Profile" button in the dashboard navigation menu.  
  2. System retrieves and displays the following profile details:  
     - Name and profile picture (from Google account).  
     - Email address associated with the Google account.  
     - Current chart creation quota (e.g., "15/20 charts remaining").  
     - Account creation date.  
     - Last login timestamp.  
  3. User reviews the displayed information.  
- **Alternate Flows**:  
  - **Profile Data Unavailable**: If the system cannot retrieve data (e.g., server error), it displays "Profile temporarily unavailable."  
- **Postconditions**:  
  a) User views their profile details.

- **Activity Diagram**:
    ```plantuml
    start
    :User clicks "My Profile";
    if (Profile data retrievable?) then (yes)
    :Display name, email, quota, account date, last login;
    else (no)
    :Display "Profile temporarily unavailable";
    endif
    stop
    ```

---

## 4. Non-Functional Requirements

### 4.1 Performance  
- Chart generation completes within **3 seconds** under normal load conditions.  

### 4.2 Usability  
- **Intuitive UI** with guided workflows (e.g., tooltips, validation hints) to assist non-technical users.  

### 4.3 Security  
- **End-to-end encryption**: User data and charts are encrypted both at Rest and in transit.  

### 4.4 Availability  
- **99% uptime** excluding scheduled maintenance windows.  
- The system should prioritize **Availability over Consistency** during network partitions. While temporary data inconsistencies may occur, the service should remain fully operational to ensure uninterrupted chart generation and access.  

### 4.5 Scalability  
- Support **1,000 concurrent users** with linear performance scaling via cloud-based auto-scaling infrastructure.  

---

## 5. Data Requirements

### 5.1 Data Models  
- **User Profile**: email, quota limits.  
- **Chart Metadata**: Chart ID, type, creation date, json chart object.  

### 5.2 Data Validation  
- CSV files must match template structure.  

---

## 6. Interface Requirements

### 6.1 User Interfaces  
- Dashboard with chart previews.  
- Upload/Download buttons.  
- Quota status display.  

### 6.2 External Interfaces  
- Google OAuth for authentication.  
- Highcharts library for rendering.  

---

## 7. System Constraints

### 7.1 Design Constraints  
- Frontend: React Framework.  
- Backend: REST API.  

### 7.2 Environmental Constraints  
- Requires modern browser support.  

---

## 8. Verification and Validation

### 8.1 Verification Plan  
- Unit tests for chart generation logic.  
- Integration tests for Google OAuth.  

### 8.2 Validation Plan  
- User testing for ease of CSV upload and chart customization.  

---

## 9. Appendices  

### 9.1 Glossary  
- **Quota**: Credits remaining for chart generation per user.  
- **CSV**: Data file format for chart input.  