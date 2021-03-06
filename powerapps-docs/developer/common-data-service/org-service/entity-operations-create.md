---
title: "Create entities using the Organization Service (Common Data Service for Apps) | Microsoft Docs" # Intent and product brand in a unique string of 43-59 chars including spaces
description: "<Description>" # 115-145 characters including spaces. This abstract displays in the search result.
ms.custom: ""
ms.date: 10/31/2018
ms.reviewer: ""
ms.service: "powerapps"
ms.topic: "article"
author: "brandonsimons" # GitHub ID
ms.author: "jdaly" # MSFT alias of Microsoft employees only
manager: "ryjones" # MSFT alias of manager or PM counterpart
search.audienceType: 
  - developer
search.app: 
  - PowerApps
  - D365CE
---
# Create entities using the Organization Service

This topic will include examples using both late-bound and early-bound programming styles. More information: 
- [Early-bound and Late-bound programming using the Organization service](early-bound-programming.md)
- [Generate early-bound classes for the Organization service](generate-early-bound-classes.md)

## Basic Create

The following examples show how to create an entity record using the late-bound and early-bound style.

<!-- TODO make this an include? -->
Each of the examples uses a `svc` variable that represents an instance of a class that implements the methods in the <xref:Microsoft.Xrm.Sdk.IOrganizationService> interface. For information about the classes that support this interface see [IOrganizationService Interface](iorganizationservice-interface.md).

> [!NOTE]
> Each entity has a unique identifier attribute which you can specify when creating an entity. In most cases you should allow the system to set this for you because the values generated by the system are optimized for best performance.


### Late-bound example

The following example shows using the <xref:Microsoft.Xrm.Sdk.Entity> class to create an account record using the  <xref:Microsoft.Xrm.Sdk.IOrganizationService>.<xref:Microsoft.Xrm.Sdk.IOrganizationService.Create*> method. 

```csharp
//Use Entity class with entity logical name
var account = new Entity("account");

// set attribute values
    // string primary name
    account["name"] = "Contoso";
    // Boolean (Two option)
    account["creditonhold"] = false;
    // DateTime
    account["lastonholdtime"] = new DateTime(2017, 1, 1);
    // Double
    account["address1_latitude"] = 47.642311;
    account["address1_longitude"] = -122.136841;
    // Int
    account["numberofemployees"] = 500;
    // Money
    account["revenue"] = new Money(new decimal(5000000.00));
    // Picklist (Option set)
    account["accountcategorycode"] = new OptionSetValue(1); //Preferred customer

//Create the account
Guid accountid = svc.Create(account);
```

### Early-bound example

The following example shows using the generated `Account` class to create an account record using the  <xref:Microsoft.Xrm.Sdk.IOrganizationService>.<xref:Microsoft.Xrm.Sdk.IOrganizationService.Create*> method.

The `Account` class is derived from the <xref:Microsoft.Xrm.Sdk.Entity> class

```csharp
var account = new Account();
// set attribute values
    // string primary name
    account.Name = "Contoso";
    // Boolean (Two option)
    account.CreditOnHold = false;
    // DateTime
    account.LastOnHoldTime = new DateTime(2017, 1, 1);
    // Double
    account.Address1_Latitude = 47.642311;
    account.Address1_Longitude = -122.136841;
    // Int
    account.NumberOfEmployees = 500;
    // Money
    account.Revenue = new Money(new decimal(5000000.00));
    // Picklist (Option set)
    account.AccountCategoryCode = new OptionSetValue(1); //Preferred customer

//Create the account
Guid accountid = svc.Create(account);
```

## Use the CreateRequest class

Instead of using the <xref:Microsoft.Xrm.Sdk.IOrganizationService>.<xref:Microsoft.Xrm.Sdk.IOrganizationService.Create*> method, you can use either the late-bound <xref:Microsoft.Xrm.Sdk.Entity> class or the early-bound entity classes with the <xref:Microsoft.Xrm.Sdk.Messages.CreateRequest> class by setting the entity instance to the <xref:Microsoft.Xrm.Sdk.Messages.CreateRequest.Target> property and then using the <xref:Microsoft.Xrm.Sdk.IOrganizationService>.<xref:Microsoft.Xrm.Sdk.IOrganizationService.Execute*> method to get a <xref:Microsoft.Xrm.Sdk.Messages.CreateResponse>. The id of the created record will be in the <xref:Microsoft.Xrm.Sdk.Messages.CreateResponse.id> property value.

```csharp
var request = new CreateRequest() { Target = account };
var response  = (CreateResponse)svc.Execute(request);
Guid accountid = response.id;
```

### When to use the CreateRequest class

You must use the <xref:Microsoft.Xrm.Sdk.Messages.CreateRequest> class if you want to pass optional parameters. There are two cases where you might need special parameters.
 - When you want duplicate detection rules to be applied. More information: [Check for Duplicate records](#check-for-duplicate-records)
 - When you are creating a record that represents a solution component, such as a [WebResource](../reference/entities/webresource.md) and want to associate it with a specific solution. In this case, you would include the value of the [Solution.UniqueName](../reference/entities/solution.md#BKMK_UniqueName) using the `SolutionUniqueName` parameter. More information: [Use messages with the Organization service](use-messages.md)

## Create related entities in one operation

When you create a new entity record you can also create related entity records in the same operation.

The following late-bound and early-bound examples will create an [Account](../reference/entities/account.md) and a [Contact](../reference/entities/contact.md) related to that account using the [contact account_primary_contact](../reference/entities/contact.md#BKMK_account_primary_contact) one-to-many relationship where the [account primarycontactid](../reference/entities/account.md#BKMK_PrimaryContactId) lookup is the `ReferencingAttribute`.

The examples will also create three related [Task](../reference/entities/task.md) entity records using the [account Account_Tasks](../reference/entities/account.md#BKMK_Account_Tasks) one-to-many relationship where the [task regardingobjectid](../reference/entities/task.md#BKMK_RegardingObjectId) lookup 
is the `ReferencingAttribute`.

### Late-bound example

In the late-bound style you must explicitly add the related entity(ies) to an <xref:Microsoft.Xrm.Sdk.EntityCollection> and then use the <xref:Microsoft.Xrm.Sdk.Relationship> class to specify the relationship using the `SchemaName` of the relationship before you can add them to the <xref:Microsoft.Xrm.Sdk.Entity>.<xref:Microsoft.Xrm.Sdk.Entity.RelatedEntities> property.

```csharp
// Use Entity class with entity logical name
var account = new Entity("account");

// Set attribute values
    // string primary name
    account["name"] = "Sample Account";

// Create Primary contact
var primaryContact = new Entity("contact");
primaryContact["firstname"] = "John";
primaryContact["lastname"] = "Smith";

// Add the contact to an EntityCollection
EntityCollection primaryContactCollection = new EntityCollection();
primaryContactCollection.Entities.Add(primaryContact);

// Set the value to the relationship
account.RelatedEntities[new Relationship("account_primary_contact")] = primaryContactCollection;

// Add related tasks to create
var taskList = new List<Entity>() {
            new Entity("task") { ["subject"] = "Task 1" },
            new Entity("task") { ["subject"] = "Task 2" },
            new Entity("task") { ["subject"] = "Task 3" }
        };

// Add the tasks to an EntityCollection
EntityCollection tasks = new EntityCollection(taskList);

// Set the value to the relationship
account.RelatedEntities[new Relationship("Account_Tasks")] = tasks;

// Create the account
Guid accountid = svc.Create(account);
```

### Early-bound example

With early-bound classes you can write less code because the classes include definitions of the relationships.

```csharp
var account = new Account();
// set attribute values
    // string primary name
    account.Name = "Sample Account";

    // Set the account primary contact
    account.account_primary_contact = new Contact()
    { FirstName = "John", LastName = "Smith" };

// Set a list of Tasks to create
account.Account_Tasks = new List<Task>() {
        new Task() { Subject = "Task 1" },
        new Task() { Subject = "Task 2" },
        new Task() { Subject = "Task 3" }
    };

// Create the account
Guid accountid = svc.Create(account);
```

## Associate entities on create

You can associate any new record with an existing record when you create it in the same way you would when updating it. You must use an <xref:Microsoft.Xrm.Sdk.EntityReference> to set the value of a lookup attribute.

This lookup attribute assignment is the same for both early and late-bound styles.

```csharp
//Use Entity class with entity logical name
var account = new Entity("account");

// set attribute values
    //string primary name
    account["name"] = "Sample Account";

Guid primarycontactid = new Guid("e6fa5509-2582-e811-a95e-000d3af40ae7");

account["primarycontactid"] = new EntityReference("contact", primarycontactid);

//Create the account
Guid accountid = svc.Create(account);
```

### Use alternate keys

If you don't know the id of the entity and the following conditions are true:
- You have configured alternate keys for the entity
- You know the key values

You can use the alternate <xref:Microsoft.Xrm.Sdk.EntityReference> constructors using the `keyName` and `keyValue` parameters.
```csharp
account["primarycontactid"] = new EntityReference("contact", "sample_username", "john.smith123");
```
> [!NOTE]
> Alternate keys are usually used only for data integration scenarios

More information: 
- [Define alternate keys to reference records](../../../maker/common-data-service/define-alternate-keys-reference-records.md)
- [Use an alternate key to create a record](../use-alternate-key-create-record.md)
- [Work with alternate keys](../define-alternate-keys-entity.md)


## Check for duplicate records

More information: [Detect duplicate data using the Organization service](detect-duplicate-data.md)

## Set default values from the primary entity

When people create new records in the application they are usually created in the context of another record. For example, you might create a new contact record in the context of an account. When this happens certain attribute values from the account entity are copied into the contact form. This expedites the creation of the new related record because the new record will have some default values set so that the person editing the record to be created doesn't need to enter those values. They can change the values if they like before saving.

The values that will be copied over when a new record is created this way is controlled by configurations applied to the CDS for Apps environment, so it can vary between environments. 

More information: 
- [Map entity fields](../../../maker/common-data-service/map-entity-fields.md)
- [Customize entity and attribute mappings](../customize-entity-attribute-mappings.md)

As a developer, you can use the <xref:Microsoft.Crm.Sdk.Messages.InitializeFromRequest> class to generate an entity with those default values already set.

The following code will create a new contact that is associated with an existing account. The contact entity will be associated with the specified account and certain attribute values, like `Telephone1` and various address values shared between the account and contact entity will be set by default.

```csharp
//The account that the contact will be associated with:
var parentAccount = new EntityReference("account", new Guid("a976763a-ba1c-e811-a954-000d3af451d6"));

// Initialize a new contact entity with default values from the account.
var request = new InitializeFromRequest()
{
    EntityMoniker = parentAccount,
    TargetEntityName = "contact"

};

var response = (InitializeFromResponse)svc.Execute(request);
//Get the Entity from the response
Entity contact = response.Entity;

// Set values that are not from the account
contact["firstname"] = "Joe";
contact["lastname"] = "Smith";

// Create the contact
Guid contactid = svc.Create(contact);
```

## Use Upsert

Another way to create an entity is by using the <xref:Microsoft.Xrm.Sdk.Messages.UpsertRequest> class. An upsert will create a new entity when there is no existing record that has the unique identifiers included in the entity passed with the request.

More information: [Use Upsert](entity-operations-update-delete.md#use-upsert)


### See also

[Retrieve an entity using the Organization Service](entity-operations-retrieve.md)<br />
[Update and Delete entities using the Organization Service](entity-operations-update-delete.md)<br />
[Associate and disassociate entities using the Organization Service](entity-operations-associate-disassociate.md)<br />

