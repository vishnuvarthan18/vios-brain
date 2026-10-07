---
source: personal Mac ~/Vishnu/projects /creative work/business/EDS Inventory Managment System  SOW _ V4.0.docx.pdf
---

EDS - Inventory Management System
                SOW
             Version 4.0
       Date: 3rd October 2024
  doodleblue Innovations PVT LTD
                                 TABLE OF CONTENTS

(1) Overview
(2) Scope of Work
(3) Declaration of Assumptions
(4) Engagement Model
(5) Tech Stack
(6) High Level Project Plan
(7) Project Efforts & Fees
(8) Project Management Procedures
(9) Deployment Phase
(10) Payment Milestone
(11) Service Terms
(12) Accepted and Agreed




1

                              EDS Egypt | Inventory Management System SOW

                              admin@doodleblue.com ​| ​www.doodleblue.com
                                                   SOW

(1) Overview

This document outlines the proposal for developing a comprehensive Integrated Inventory Management
System (IMS) coupled with a mobile application. The objective of this project is to enhance the efficiency
of inventory tracking, simplify data entry processes, and improve quality control operations across
various business functions.

Objectives
The primary goals of this integrated solution are to:
    ● Streamline inventory tracking: Ensure accurate and real-time monitoring of stock levels.
    ● Enhance data entry processes: Simplify the input of inventory data and related contracts.
Improve quality control : Implement systematic checks to maintain product standards.

Key Modules

1. Customer Onboarding
    ● This module will facilitate the onboarding of new customers into the inventory system. It will
        include features for capturing essential customer data, managing account setup, and
        integrating customer preferences into the inventory management process.
2. Barcode Management
    ● To improve efficiency, the system will incorporate barcode management, enabling the easy
        creation, printing, and scanning of barcodes. This will facilitate quick identification and
        tracking of inventory items, reducing human error in data entry.

3. Item History Tracking
     ● This module will maintain comprehensive records of each item’s movement throughout the
        inventory lifecycle. It will track item acquisition, usage, and disposal, providing insights into
        trends and helping with inventory forecasting.

4. Data Entry for Contracts
    ● The system will support the seamless entry of data related to various contracts including
        ADSL (Asymmetric Digital Subscriber Line), mobile services, and digital wallets. This will
        ensure that all relevant information is centralized, accessible, and up-to-date.



2

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
5. Robust Reporting
    ● A powerful reporting module will generate detailed reports on inventory levels, sales trends,
       customer interactions, and contract status. Customizable reporting options will allow
       stakeholders to view data that is most relevant to their needs.



Mobile Application Features

1. Barcode Scanning
    ● The mobile app will feature a user-friendly interface with integrated barcode scanning
        capabilities. This will enable employees to scan items at various stages of the inventory
        process, from receiving shipments to fulfilling orders.

2.Real-Time Updates
    ● With the mobile application, any changes to inventory data will be updated in real-time,
        ensuring that all users access the most current information. This will help prevent stock
        discrepancies and enhance decision-making.

3.Quality Control Functionality
   ● The app will include features designed for quality control, allowing users to conduct checks
        at different points in the inventory process. Notifications and alerts can be set up for items
        that do not meet predefined quality standards, ensuring that only high-quality products are
        delivered to customers.



(2) Scope of Work


Key Features

    Web Application


             Module                       Sub Module                                      Description

                                                                   Secure login functionality with multi-user role
                                        Login Module              support, authentication via email/password, and
     Inventory Management                                                      session management


3

                                EDS Egypt | Inventory Management System SOW

                                admin@doodleblue.com ​| ​www.doodleblue.com
    Module            Sub Module                                   Description

                                          Registering customer details (legal name, address, tax
                Customer Onboarding
                                                       ID, authorized persons, etc.)

                                           Allocating barcodes based on the estimated volume
                 Barcode Allocation
                                                       of boxes for each customer

                                           Order details, quantity, delivery address; Real-time
                Procurement Tracking
                                                          tracking, notifications

                                          Creating work orders for customer requests to pick up
                Work Order Creation
                                                      boxes and track their status

                                          Managing barcode scans at multiple stages (packing,
              Barcode Scanning (Web )
                                            loading, receiving) and updating box statuses

                                            Update box status through the system, such as "In
                   Status Updates
                                                         Transit," "In," or "Out"

                                          Store boxes in shelves, scan locations, and update the
                  Inventory Storage
                                                system with real-time location and status

                                          Track the history of each item with timestamps, status
              Item History and Tracking
                                                      changes, and location updates

                                           Generate work orders for box retrieval, scan boxes
                Retrieval and Delivery
                                                     during loading and delivery

                  Barcode Types &          Support multiple barcode formats for different sizes
                   Configuration              of boxes, folders, documents, and locations

                                              Generate reports on box history, inventory
                Reporting and History
                                           movements, and invoicing at the end of the month

                                           Create invoices based on completed work orders at
                 Invoice Generation
                                                          the end of the month

                                            For creating new users and managing role-based
                   Admin Module
                                                              permissions

                    Status Chart          Display the total number of boxes by status on a daily
                                                                 basis


4

             EDS Egypt | Inventory Management System SOW

             admin@doodleblue.com ​| ​www.doodleblue.com
         Module                  Sub Module                                    Description

                            Box Additions Chart        Show the number of new boxes added on a daily
                                                                           basis

                         Contracts Barcoded Chart     Track the number of contracts barcoded on a daily
                                                                            basis
       Dashboards
    (Internal Report)   Data Entry Complete Chart      Show the number of contracts with "Data Entry
                                                             Complete" status on a daily basis

                           QC Completed Chart          Show the number of boxes completed in the QC
                                                                      process daily

                           Contract Totals Chart       Track the total number of contracts added daily

                            Active Users Chart           Show the number of active users daily, with a
                                                         breakdown by role (Receiver, Data Entry, QC)

                           Productivity Per User     Track productivity for each user, such as the number
                                  Chart                   of boxes handled, contracts barcoded, etc.

                                                      The Data Entry Module will enable users to input,
                                                     validate, and manage contract data related to ADSL,
                                 Overview            Mobile, and Wallet services. Each data entry will be
                                                      verified, and status updates will automatically be
                                                                   applied to the inventory.

                                                      A form for entering contract data for ADSL services,
                                                                             including:
                                                       - Contract Archive Code (Unique identifier for each
                            Data Entry for ADSL                               contract)
                                 Contracts                         - Landline (10-digit numeric)
                                                               - ACC Number (Numeric identifier)
                                                     - Validation logic to ensure no duplicates, and correct
                                                                           format inputs.

                                                        When all required contract data is entered, the
                            Real-Time Updates       system will automatically mark the corresponding box
                                                              status as "Data Entry Complete."

       Data Entry
                            Simultaneous User          Multiple data entry agents can work on different
                                 Support               contracts simultaneously, with real-time conflict

5

                        EDS Egypt | Inventory Management System SOW

                        admin@doodleblue.com ​| ​www.doodleblue.com
          Module             Sub Module                                      Description

                                                                        prevention.

                                                                        Input fields for:
                                                      - SIM Serial (19-digit numeric starting with 8920)
                                                                - Dial Serial (11-digit numeric)
                    Data Entry for Mobile and
                        Wallet Contracts            - Contract Archive Code (Unique identifier for each
                                                                            contract)
                                                    - System validation to ensure correct data formats
                                                                  and non-duplicate entries.

                                                   SIM and Dial serials will be validated automatically. If
                                                   both fields are empty, the contract will be marked as
                         Data Validation
                                                    "Rejected." For rejected contracts, users must input
                                                                additional customer details.

                                                    Automatically updating box status after data entry
                    Data Entry Status Update
                                                                 and contract validation

                                                    When all delivery sheets and contracts are entered
                    Automatic Status Updates        for a box, the system will automatically change the
                                                         status of the box to "Data Entry Complete

                                                   Only authorized users (Data Entry Agents) can access
                        Role-Based Access            and modify contract forms. Administrators can
                                                    oversee progress and resolve any system conflicts.

       QC Module                                  Allowing inspectors to validate the data, marking boxes
                       Quality Control (QC)
                                                               as "QC Done" after passing inspection

                                                   Generating Receiving Reports, ADSL Report, Mobile
                                                               Report, Wallet Report, Boxes Report
       Reports            Client Reports
                                                    Generate downloadable reports in PDF/Excel with
                                                                         filtering options.

                                                     We will also migrate the existing data set to the
                                                              proposed platform & test to confirm the
                                                                   workability of the application.
Data Migration      Existing Data set migration             During development we will request EDS to
                                                             share sample datasets with us once this is
                                                            completed in dev we will do the Production
                                                                          Data migration.

6

                   EDS Egypt | Inventory Management System SOW

                   admin@doodleblue.com ​| ​www.doodleblue.com
    Mobile App


              Module                      Sub Module                                Description


                                        Barcode Scanning          Scanning boxes at customer site, during loading,
                                           (Receiving)             and at the receiving area using mobile devices.

          Receiver Module             Synchronization with       Synchronization between the mobile app and the
                                            Server                      server for real-time data updates.

                                                                  Updating the status of boxes to "Received," "In
                                     Barcode Status Update
                                                                    Transit," and "IN" based on scan results.

                                                                   Quality inspectors can verify box status, data,
         Quality Control (QC)            QC Inspection
                                                                         and mark items as passed/failed.
              Module




(3) Declaration of Assumptions for the Integrated Web application and Mobile App

Inventory Management Assumptions

1. Customer Data Validation and Standardization

     ●     All customer data, including legal names, addresses, and tax IDs, will be thoroughly validated
          before being entered into the system. This process ensures consistency and accuracy, reducing
          the risk of errors that could impact inventory management and customer relations.
          Standardizing data formats will facilitate easier reporting and integration with other systems.

2. Multiple User Roles and Access Levels

     ●    The system will support various user roles (e.g., administrators, warehouse staff, and sales
          personnel), each with specific access permissions. This hierarchy will enable tailored user
          experiences, ensuring that individuals only access the information and functions necessary for
          their roles. For example, warehouse staff may have permissions limited to inventory
          management, while admins have full access to system settings and reporting.

3. Automatic Barcode Allocation


7

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
    ●   Barcodes will be generated automatically based on estimated customer volumes, streamlining
        the tracking process. This feature will allow for efficient inventory management, ensuring that
        items are easily identifiable without the need for manual barcode entry. The system will consider
        historical data and anticipated demand to allocate barcodes effectively.

4. Handling Large Volumes of Unique Barcodes
    ● The system is designed to manage a substantial volume of inventory items, each assigned a
       unique barcode for tracking purposes. This capability ensures that the inventory remains
       organized and easily navigable, minimizing the chances of misplacing items. The architecture will
       support scalability, accommodating future growth in inventory size.

5. Real-Time Updates for Box Status Changes
    ● The system will enable real-time updates for the status of inventory boxes (e.g., “in transit,” “in,”
        “out”). This feature will provide users with up-to-the-minute information, allowing for better
        tracking and management of inventory flow. Timely updates will help in making informed
        decisions, especially during high-demand periods.

6. Barcode Scanning Integration
    ● Barcode scanning will be possible through both the web interface and the mobile application,
        ensuring seamless integration across platforms. This dual capability will enhance operational
        efficiency, allowing staff to scan items regardless of their location, which is particularly useful in
        large warehouses or during transportation.

7. Detailed Item History Tracking
    ● The system will maintain comprehensive histories for each inventory item, tracking all changes in
        location and status. This tracking will provide insights into item movement and usage patterns,
        aiding in inventory forecasting and audit trails. Users will be able to access historical data to
        identify trends and make data-driven decisions.

    ●   The web application will be designed to support simultaneous access from multiple locations
        without performance degradation. This capability is crucial for organizations operating across
        various sites, ensuring that all users can access and update inventory data concurrently, thus
        enhancing collaboration.

9. Integration of Inventory Storage and Mobile App
     ● The inventory storage and tracking module will be fully integrated with the mobile application,
         allowing for seamless receiving and dispatching processes. This integration will facilitate
         real-time updates and ensure that all inventory movements are documented accurately in the
         system.


8

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
10. Invoice Generation from Completed Work Orders

    ●   Invoices will be generated automatically based on completed work orders at the end of each
        month, adhering to predefined templates. This automation will streamline the billing process,
        reducing administrative overhead and minimizing the risk of errors in invoicing.

Data Entry Assumptions

1. Multiple Data Entry Forms

    ●   Data entry will involve various forms specific to different contracts (ADSL, Mobile & Wallet), each
        designed with fields pertinent to the respective services. This specialization will ensure that
        users provide all necessary information for each type of contract, enhancing data completeness
        and accuracy.

2. Automatic Status Updates

    ●   The system will automatically update the status of inventory boxes once all necessary data entry
        is completed. This feature will ensure that inventory records are always current, reflecting the
        latest operational changes without manual intervention.



3. Differentiated User Permissions

    ●   Users involved in data entry will have a distinct set of permissions compared to other roles, such
        as receivers and administrators. This differentiation will help maintain data integrity and security,
        ensuring that only authorized personnel can modify critical information.

4. Duplicate Entry Prevention

    ●   The system will automatically reject duplicate entries for contract archive codes, SIM serials, and
        Dial serials. This feature will minimize errors and maintain the accuracy of the inventory
        database, contributing to more reliable reporting and management.

5. Support for Simultaneous Data Entry

    ●   The system will accommodate multiple users performing data entry across various contracts and
        boxes simultaneously. This support will enhance efficiency, especially during peak operational
        periods when timely data entry is crucial.


9

                                EDS Egypt | Inventory Management System SOW

                                admin@doodleblue.com ​| ​www.doodleblue.com
6. Field Validation Checks

     ●   Each data field will undergo validation checks to ensure adherence to predefined formats (e.g.,
         10-digit numeric landlines, unique contract archive codes). This validation will prevent incorrect
         data entry, maintaining the integrity of the inventory database.

7. Tracking of Rejected Contracts

     ●   The system will store the status of rejected contracts along with the reasons for rejection. This
         feature will help users understand and address data entry issues, improving overall data quality
         and operational efficiency.

8. Modification Restrictions after Completion

     ●   Once data entry is completed for a box, modifications will be restricted unless explicitly
         authorized. This control measure will enhance data integrity, preventing unauthorized changes
         that could lead to discrepancies.

9. Keyboard Shortcuts for Efficiency

     ●   The web interface for data entry will support keyboard shortcuts to facilitate quick navigation
         between fields. This feature will improve user experience and productivity by allowing users to
         input data more efficiently.




10. Real-Time Data Saving

     ●   All data entered into the system will be saved in real-time to prevent loss in case of
         disconnection or system failure. This feature will ensure data security and reliability, minimizing
         the impact of technical issues on operational processes.

Reports Assumptions

1.Availability of Various Reports

     ●   Reports will be generated for different operational aspects (receiving, ADSL, Mobile, Wallet,
         boxes) as well as internal productivity metrics. This diversity will provide stakeholders with
         comprehensive insights into inventory management and operational performance.
10

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
2. Filterable Reports

     ●   Users will be able to filter reports by date ranges and specific criteria (e.g., box type, status, user
         type). This functionality will enable targeted analysis, allowing users to drill down into data that
         is most relevant to their needs.

3. Exportable Report Formats
    ● The system will allow reports to be generated in various formats (PDF, Excel, CSV) for external
        sharing. This flexibility will facilitate communication with stakeholders and support data analysis
        outside the system.

4. Real-Time Dashboards

     ●   Internal users will have access to real-time dashboards to track the status of boxes and overall
         performance metrics. These dashboards will provide visual insights, enabling quick
         decision-making and performance monitoring.

5. Productivity Tracking

     ●   Productivity reports will track key metrics, including active days, the number of boxes processed,
         and contracts handled per user. This tracking will assist in evaluating individual and team
         performance, identifying areas for improvement.

6. Access Control for Reports

     ●   Reports will be accessible only to authorized user roles based on predefined permissions. This
         control will ensure that sensitive information is only available to those who require it for their
         roles, enhancing data security.




7. No Automatic Monthly Report Generation

     ●   The system will not automatically generate and send reports at the end of each month. This
         decision allows users to customize their reporting needs and schedule rather than relying on
         potentially irrelevant automated reports.




11

                                  EDS Egypt | Inventory Management System SOW

                                  admin@doodleblue.com ​| ​www.doodleblue.com
Project Scope Assumptions

1. Clearly Defined Project Scope

     ●   The project scope will be explicitly defined and agreed upon by all stakeholders. This clarity will
         help prevent scope creep and ensure that all parties have a mutual understanding of project
         objectives and deliverables.

2. Realistic Project Timeline

     ●   The project timeline will be realistic and achievable, taking into account the complexity of the
         application and the availability of necessary resources. This consideration will help ensure timely
         delivery and successful project completion.




(4) Engagement Model

A. The typical flow will be:

     ●   Product Concept Definition -> Client Approval - Requirement Gathering sign off.
     ●   Design Creation -> QA -> Client Approval - Design Completion Sign Off.
     ●   Web & Mobile Frontend Creation -> QA -> Client Approval - Frontend Development. Completion
         Sign Off.

     ●   Post completion & production push, we will run a Training session, where around 10 EDS
         employees will be given a download of how to use the system in real time. These sessions will be
         done in Virtual meetings via Google meet/Zoom etc.


B. We will maintain & track daily work on a tracker + have weekly calls at a minimum + monthly
executive reviews + catch up around releases as per the schedule defined.




12

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
(5) Tech Stack




                 Vertical                                       Technologies

        Front-End Development                                      React JS

        Back-End Development                                       Node JS

             Mobile App                                         React Native

              Database                                        MySQL/MongoDB

            Infrastructure                                          AWS




(6) High level Project Plan


  Program Plan:


                                                                 Month 1     Month 2     Month 3
 S.No        Modules                   Sub Modules
                                                              W1 W2 W3 W4 W1 W2 W3 W4 W1 W2 W3 W4
   1      Login Module        Web Login
   2                          Customer Onboarding
   3                          Barcode Allocation
   4                          Work Order Creation
   5                          Barcode Scanning (Web)
   6                          Status Updates
   7        Inventory         Inventory Storage
   8       Management         Item History and Tracking
   9                          Retrieval and Delivery
  10                          Barcode Types & Configuration
  11                          Reporting and History
  12                          Invoice Generation
  13                          Admin Module
  14                          Status Chart
  15        Dashboard         Box Additions Chart
13

                                  EDS Egypt | Inventory Management System SOW

                                  admin@doodleblue.com ​| ​www.doodleblue.com
  16                         Contracts Barcoded Chart
  17                         Data Entry Complete Chart
  18                         QC Completed Chart
  19                         Contract Totals Chart
  20                         Active Users Chart
  21                         Productivity Per User Chart
  22                         ADSL Form
  23        Data Entry       Mobile & Wallet Form
  24                         Data Entry Status Update
                             Receiving, ADSL, Mobile, Wallet,
  25         Reports
                             Boxes Reports
  26     QC Module (Web)
  27                          Mobile App
  28                         Barcode Scanning (Receiving)
  29                         Synchronization with Server
  30                         Barcode Status Update
  31                         Offline Mode
             Receiver
  32                         Work Order Assignment
  33                         Login Module (Mobile)
  34                         QC Module (Mobile)
  35                         Offline Mode
         Quality Control &
  36                        QC Module
          Data Migration
  37         Final UAT      User acceptance testing
                            Pushing the application to
  38     Rollout & Training production & doodleblue team
                            train the EDS employees


Note : Timelines are to be considered from the project start date post approval.


(7) Project Efforts & Fees

     ●   Timeline - 3-3.5 Months
     ●   Effort - 3250 Hours
     ●   In the interest of the long term relationship, we are willing to execute this project at a
         discounted cost of USD 10,000.
     ●   USD 2500 to be paid after 1 year of successful completion of the project.




14

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
(8) Project Management Procedures

     1. Monitoring Mechanisms:
          i. Daily Scrum call & Weekly Conference Calls
         ii. Delivery Sign Off for each milestone
         iii. Gateway review Approval per each milestone
                     The tester will test the coding for completion of development and integration.

     2. Regular communication strategy
          i. The Project Manager and Developer, will be available on Skype Instant Messenger during
             an agreed upon time of business hours. A list of IMIDs will be circulated at the beginning of
             the project via email.

     3. Version Control
          i. doodleblue will be checking in code on a sprint basis to its version control.



(9) Deployment Phase

     ●   First, the work will be uploaded to the Staging Server, after checking on regression for
         previous builds.
     ●   After passing the UAT for each and every successful delivery milestone, the code will be
         migrated to the client server upon receiving the sign off and Payment milestone
         acceptance.
     ●   Upon completion of the project and signoff and successful payment from the client, the
         code will be shipped to the client server

     ●   doodleblue will ship the complete code to the production (client server) assuming
         cloud/infrastructure environment and setup are in place
     ●   Post completion of the production migration all controls of the website will be transferred to Trio
         TechDesign.



(10) Payment Milestone


              ❖ Initial Payment (30%) - USD 3000.
              ❖ Monthly Payment based on Project Completion percentage.




15

                                  EDS Egypt | Inventory Management System SOW

                                  admin@doodleblue.com ​| ​www.doodleblue.com
(11) Service Terms

     1. doodleblue will provide minor bug-fixing & servicing beyond the go-live sign-off date for the
        duration of 30 days.
     2. Major fixes during the 30 days period will be discussed and fixed accordingly with the base cost
        which is mutually agreed by both the parties.
     3. Post-go-live support beyond the 30 days period would be covered as a part of the AMC on a
        separate contract (Which we can discuss post commence of the project and finalize based on
        mutual convenience




(12) Accepted and Agreed
doodleblue Innovations Private Limited               EDS Egypt
By: _____________________________                    By:_____________________________


Name: Atishe Chordia                                 Name: Yasser Megahed


Title: CEO                                           Title: Chairman & CEO


Date:                                                Date: 3-Oct-2024




16

                                 EDS Egypt | Inventory Management System SOW

                                 admin@doodleblue.com ​| ​www.doodleblue.com
