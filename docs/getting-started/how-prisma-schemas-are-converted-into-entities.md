---
id: how-prisma-schema-is-converted-into-entities
title: How Prisma Schema Is Converted Into Amplication Entities
sidebar_label: How Prisma Schema Is Converted Into Amplication Entities
slug: /getting-started/how-prisma-schema-is-converted-into-entities
pagination_prev: how-to/import-prisma-schema
---

# How Prisma Schema Is Converted Into Amplication Entities

After you [import your prisma schema](/how-to/import-prisma-schema), it'll go through a series of conversions so that it matches Amplication's internal structure.

These conversions are what you see in the log while your `schema.prisma` file is processing.

:::note
See the [Log Warnings](#log-warnings) section on possible warnings you might encounter while your `schema.prisma` is processing.
:::

Amplication will process your schema by running it through 3 different phases:

1. Validate
2. Prepare
3. Convert

In this article, you'll learn more about those 3 conversion steps that your schema goes through so that it matches Amplication's internal structure.

### Step 1: Validate   

The first validation step is performed immediately after you upload your `schema.prisma` file.

The purpose of this step is to validate the structure and syntax of the `schema.prisma` file.
This validation process is grounded in the [Prisma conventions](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#naming-conventions) and other restrictions.

Besides checking structure and syntax, the validate step also checks if the schema contains any models to process. If no models are present, you will see an error, and the conversion process will stop.

### Step 2: Prepare   

During the prepare phase, Amplication runs the schema through 5 different conversion operations:

1. Model Names
2. Field Names
3. Field Types
4. Model IDs
5. Model ID Field

Each operation represents a transformation to be made on the schema, and their collective execution ensures the schema is in the correct format before being used for Amplication’s entities and fields creation.

#### Model Names

This part renames models in the Prisma schema to follow Amplication's best practices and conventions.

Amplication's format is model names in **PascalCase**, **no underscores**, and **singular**.

All original model names are preserved in the database with the `@@map` custom attribute where necessary.

:::note
If the model already has the **`@@map`** attribute, it won’t be added, even if the model name was formatted. 
:::

Here's an example of a transformation that would happen on one model:

```diff title="schema.prisma"
-model course_rating  
+model CourseRating {
  id        String    @id @default(cuid())
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  rating    Int      

+@@map("course_rating")
}
```

#### Field Names

Field names will also go through a similar conversion.

Amplication's format for field names is **camelCase** and **no underscores**.

It also ensures that original field names are preserved in the database by adding `@map` custom attributes where needed.

:::note
If the field already has the **`@map`** attribute, it won’t be added, even if the field name was formatted.
:::

Here's a quick example of a simple field name transformation:

```diff title="schema.prisma"
model Product {
  id          String   @id @default(cuid())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  name        String?
-item_price  Float?
+itemPrice   Float?    @map("item_price")
  description String?
  orders      Order[]
}
```

#### Field Types

Now that Amplication has updated both the model names and field names, it has to update the types of the fields.

For example, here's how two models could see transformations:

```diff name="schema.prisma"  
-model products
+model Product {
  id          String   @id @default(cuid())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  name        String?
  itemPrice   Float?
  description String? 
- orders      orders[]
+ orders      Order[]
}

-model orders
+model Order {
  id         String    @id @default(cuid())
  createdAt  DateTime  @default(now())
  updatedAt  DateTime  @updatedAt
  totalPrice Int?
- product    products?  @relation(fields: [productId], references: [id])  
+ product    Product?  @relation(fields: [productId], references: [id])
  productId  String?
}
```

#### Model IDs

This phase converts `@@id` attributes in your models to `@@unique` attributes.

It also introduces an `id` field of type `String` to every model that uses a composite ID.

Here's an example of this transformation:

```diff title="schema.prisma"
model DomainUnit {
+ id       String @id @default(cuid())
  domainId String
  unitId   String

- @@id([domainId, unitId])
+ @@unique([domainId, unitId])
}
```

#### Model ID field  

Finally, there's a conversion of all ID fields.

This part ensures that your schema aligns with Amplication's conventions of having an **`id`** field for each model.

non-ID fields named **`id`** are renamed to **`${modelName}Id`** to prevent any collisions with the actual ID field.

If an ID field (a field with an `@id` attribute) has a different name, it's renamed to **`id`**.

In both cases, a `@map` custom attribute is added to the field with the original field name to preserve the actual field name in the database.

Here's an example:

```diff title="schema.prisma"
model RandomName {
- randomId.         Int     @id  
+ id                Int     @id @map("randomId")
- id.               Int?
+ randomNameId      Int?    @map("id")
  fullName          String
} 
```

### Step 3: Convert

In this final step, Prisma data types get converted into Amplication data types.

Here is a table summarizing the Prisma to Amplication data type conversions:

| Prisma | Amplication | Description |
|-|-|-|  
| String (ID) | Id | Converts ID field with String type to Amplication Id type, defaults to cuid |
| Int (ID) | Id (autoincrement) | Converts ID field with Int type to autoincrement Amplication Id |   
| Boolean | Boolean | Same Boolean type |
| DateTime (@default(now())) | CreatedAt | DateTime with now() default converts to CreatedAt |
| DateTime (@updatedAt) | UpdatedAt | DateTime with updatedAt attribute converts to UpdatedAt |
| DateTime | DateTime | Plain DateTime remains DateTime |
| Float, Decimal | Decimal Number | Float and Decimal convert to Decimal Number |
| Int, BigInt | WholeNumber | Int and BigInt convert to WholeNumber |
| String | SingleLineText | String converts to SingleLineText |    
| Json | Json | Json types are equivalent |  
| Enum | OptionSet | Enum converts to OptionSet |
| Enum[] | MultiSelectOptionSet | Enum array converts to MultiSelectOptionSet |
| Model relation | Lookup | Relation to another model becomes Lookup |

## Common Id Field Conversion Scenarios 

During schema conversion, the `id` field will require special handling in some cases.

#### 1. If the `id` field already has a `@default` attribute, it gets removed.

Having multiple `@default` attributes is invalid.
Therefore, if your `id` field already has a `@default` attribute, it will be removed during conversion.
This is because a `@default` attribute matching the `id` type (like `cuid()`) gets added during conversion.

```graphql
// Original model (After Introspection)
model User {
  id Int @default(0)
  name String
}

// After Importing Into Amplication
model User {
  id Int @default(autoincrement())
  name String  
}
```

#### 2. If a model doesn't have a field that can become the `id`, we add an `id` field.

The added `id` field will be of type `String` with a default `cuid()` value.

```graphql 
// Original model (After Introspection)
model Product {
  price Decimal @unique
  name String
}

// After Importing Into Amplication
model Product {
  id String @id @default(cuid()) 
  price Decimal @unique
  name String
}
```

#### 3. If multiple unique fields could become the `id`, the one named `id` gets the `@id` attribute. 

Here's an example:

```graphql
// Original Model (After Introspection)
model User {
  email String @unique
  name String @unique
  id Int @unique 
}

// After Importing Into Amplication
model User {
  email String @unique
  name String @unique
  id Int @unique @id @default(autoincrement())
}
```

#### 4. If no unique field is named `id`, one gets renamed to `id` and mapped to its original name.

If no unique field is named `id`, one will be renamed to `id` and mapped to its original name with the `@map` custom attribute.

```graphql
// Original Model (After Introspection)
model User {
  email String @unique
  name String @unique
  username Int @unique
}

// After Importing Into Amplication
model User {
  id String @unique @map("email")
  name String @unique
  username Int @unique
}
```

## Log Warnings

While your `schema.prisma` file is processing you'll see various types of logs like `(Info)` and `(Warning)` giving you information on what's happening in real-time.

The schema will still process with warnings, but it's possible you might want to make some changes to your schema file and re-upload after correcting the warnings.

![Import Prisma Schema Upload Log](./assets/import-prisma-schema-log.png)

### 1. Custom Attributes on enum keys

Amplication currently does not support adding custom attributes to enum and enum keys.
Instead, Amplication allows you to define options as an `optionSet` or `multiSelectOptionSet`.

So if your Prisma schema includes `@map` or `@@map` custom attributes on an enum or enum keys, these attributes will not be accurately represented after processing.
The final schema file will not contain these mapping attributes.  

If your app logic relies on `@map` and `@@map` custom attributes, you'll need to manually adjust the generated `schema.prisma` file or your app code to ensure everything works correctly.

### 2. Composite IDs on models   

A model in Prisma can use the `@@id` custom attribute and combine fields to create a unique identifier for each record.
According to Amplication's best practices, each model should have a single, distinct `id` field.

During schema processing. Amplication modifies IDs created with the `@@id` custom attribute to be `@@unique` custom attributes.
It also introduces a new `id` field of type `String` to each model.   

For example, if your generated `schema.prisma` file looks like the following:

```tsx title="schema.prisma"
model Employee {
  firstName String
  lastName  String
  birthdate DateTime

  @@id([firstName, lastName, birthdate])
}
```

Amplication will transform it to look like the following:  

```diff title="schema.prisma"
model Employee {
+ id String @id @default(cuid())
  firstName String
  lastName  String
  birthdate DateTime

- @@id([firstName, lastName, birthdate])  
+ @@unique([firstName, lastName, birthdate])
}
```

As the schema is processed, we log warning entries if we encounter a model without an `id` field and an existing `@@id` attribute.

This approach maintains uniqueness and integrity for each record in the database, but it may impact how relationships and queries are designed, given the shift from a composite id with `@@id` to a singular one.
