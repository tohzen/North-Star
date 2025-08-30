# Solution Architecture: Crabby Cakes Rescues

This document outlines the proposed technical architecture for the Crabby Cakes Rescues Volunteer Management & Automation System on the Salesforce platform.

---
## 1. Data Model

The core of our solution will be a set of standard and custom objects designed to provide a 360-degree view of the shelter's operations.

### **Custom Object: `Volunteer__c`**
This object will be the central hub for all volunteer information.

**Key Fields:**
* `First Name` (Text)
* `Last Name` (Text)
* `Email` (Email)
* `Phone` (Phone)
* `Status__c` (Picklist: Applicant, Active, Inactive, On Hold)
* `Skills__c` (Multi-select Picklist: Dog Walking, Cat Care, Events, Admin, Grant Writing)
* `Onboarding_Complete__c` (Checkbox)
* `Total_Volunteer_Hours__c` (Number, updated by automation)

### **Custom Object: `Animal__c`**
This object will track every animal under the care of the shelter.

**Key Fields:**
* `Animal_Name__c` (Text)
* `Species__c` (Picklist: Dog, Cat, Other)
* `Breed__c` (Text)
* `Intake_Date__c` (Date)
* `Adoption_Status__c` (Picklist: Available for Adoption, In Foster Care, Adopted)

### **Custom Object: `Supply__c`**
This object will provide a simple inventory management system for critical supplies.

**Key Fields:**
* `Supply_Name__c` (Text, e.g., "Dry Dog Food - 30lb Bag")
* `Category__c` (Picklist: Food, Medicine, Cleaning, Bedding)
* `Quantity_on_Hand__c` (Number)
* `Reorder_Level__c` (Number)

### **Leveraging Standard Objects**

For fundraising and adoption tracking, we will leverage the power of the Salesforce Nonprofit Success Pack (NPSP).
* **Donations:** We will use the standard `Opportunity` object to track pledges and incoming donations.
* **Adopters:** We will use the standard `Contact` object to store information about the people who adopt animals and donate.
* **Adoptions:** We will use the standard `Affiliation` object (or create a simple `Adoption__c` junction object) to link a `Contact` record to the `Animal__c` record they adopted.

## 2. Automation

To eliminate manual work and ensure data consistency, we will build a series of automations using Salesforce Flow. The core automation will be the "Post-Adoption Process."

### **Record-Triggered Flow: "Post-Adoption Process"**

* **Trigger:** This Flow will automatically run whenever an `Animal__c` record is updated and the `Adoption_Status__c` field is changed to **"Adopted"**.
* **Automated Actions:** The Flow will immediately perform the following tasks without any human intervention:

    1.  **Create Adoption Record:** It will create a new `Adoption__c` record, linking the `Animal__c` record to the `Contact` record of the new pet parent and stamping the `Adoption_Date__c`.
    2.  **Update Key Reports:** The change in the animal's status will automatically update our key reports and dashboards. The "Available Beds" count on our main dashboard will instantly go up by one, and the "Adoptions This Month" chart will also be updated.
    3.  **Notify Key Personnel:** The Flow will send an automated **email alert** to the Volunteer Coordinator and the Supply Manager. This notification will inform them that the total animal count has decreased, allowing them to accurately plan for volunteer shifts and inventory needs (food, medicine, toys, etc.).
    4.  **Create a Follow-Up Task:** It will create a new "Follow-Up Task" and assign it to a staff member to call the new family in two weeks to see how everything is going.
    5.  **Send "Welcome Home" Email:** It will send a beautiful, pre-written "Congratulations & Welcome Home!" email to the adopter, including helpful tips for the first few weeks with their new family member.
