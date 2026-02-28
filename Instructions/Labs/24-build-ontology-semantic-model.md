---
lab:
    title: 'Build an ontology (preview) from a semantic model in Fabric IQ'
    module: 'Create ontology with Microsoft Fabric IQ'
---

# Build an ontology (preview) from a semantic model in Fabric IQ

There are two ways to build a Fabric IQ ontology: manually, by creating each entity type and relationship from scratch, or automatically, by generating the structure from a Power BI semantic model. This lab uses the semantic model approach.

In this lab, you'll load sample data for a fictitious healthcare company into a lakehouse and eventhouse, build a semantic model on top of it, and then generate an ontology from that model. The sample data includes facilities, departments, rooms, individuals, and medical equipment. Each table in the semantic model becomes an entity type, and each relationship between tables becomes a relationship type in the ontology.

This lab takes approximately **45** minutes to complete.

> **Note**: You need a [Microsoft Fabric trial](https://learn.microsoft.com/fabric/get-started/fabric-trial) to complete this exercise. You'll also need to enable the following [tenant settings](https://learn.microsoft.com/fabric/iq/ontology/overview-tenant-settings): **Enable Ontology item (preview)** and **User can create Graph (preview)**.

## Create a workspace

Before working with ontologies in Fabric, you need a workspace with a Fabric capacity.

1. Navigate to the [Microsoft Fabric home page](https://app.fabric.microsoft.com/home?experience=fabric) at `https://app.fabric.microsoft.com/home?experience=fabric` in a browser, and sign in with your Fabric credentials.
1. In the menu bar on the left, select **Workspaces** (the icon looks similar to &#128455;).
1. Create a new workspace with a name of your choice, selecting a licensing mode in one of the following workspace types: *Fabric*, *Fabric Trial*, or *Power BI Premium*.
1. When your new workspace opens, it should be empty.

## Create a lakehouse with sample data  

Now you'll create a lakehouse and load hospital operations data that will form the basis of your ontology.

1. In your workspace, select **+ New item** > **Lakehouse**.
1. Name the lakehouse `LamnaHealthcareLH` and select **Create**.
1. When the lakehouse opens, you'll upload CSV files and convert them to tables.

### Upload and load the hospital data files

You'll upload five CSV files to the lakehouse containing entity data for hospitals, departments, rooms, patients, and vital sign monitors, then convert each file to a Delta table.

1. Download all sample CSV files from the [course repository sample-data folder](../sample-data/):
   - **Hospitals.csv** - Healthcare facilities in your network
   - **Departments.csv** - Hospital departments (ICU, Emergency, Surgical)
   - **Rooms.csv** - Individual rooms within departments
   - **Patients.csv** - Current patients and their room assignments
   - **VitalSignEquipment.csv** - Monitoring equipment assigned to patients (which patient is being monitored, and when monitoring started)
   - **VitalSignsReadings.csv** - Patient vital sign measurements (heart rate, oxygen levels, respiratory rate) collected over time from the vital sign equipment

1. Upload the five lakehouse files:
   - In the lakehouse, select **Upload files** from the main view
   - Browse to and select these five files: **Hospitals.csv**, **Departments.csv**, **Rooms.csv**, **Patients.csv**, **VitalSignEquipment.csv**
   - Select **Open**
   - Select **Upload** to upload all five files at once
   - Wait for the upload to complete
   
   > **Note**: Do not upload **VitalSignsReadings.csv** to the lakehouse. You'll load it to the eventhouse in the next section, where it belongs as time-series data.

1. Convert each uploaded file to a table:
   - In **Explorer**, select the **Files** folder, where you should see all five CSV files
   - For each file, select the **ellipsis (...)** to the right of the file name
   - Select **Load to Tables** > **New table**
   - Configure the table:
     - **Table name**: Fabric pre-fills this from the filename with a lowercase first letter (e.g., `hospitals`, `departments`, `rooms`, `patients`, `vitalSignEquipment`). 
     - **Column header**: Check **Use header for column names**
     - **Separator**: Leave as comma (`,`)
   - Select **Load**
   - Repeat this process for all five files

1. Verify you have five tables in the **Tables** section: `hospitals`, `departments`, `rooms`, `patients`, and `vitalSignEquipment`.

## Create an eventhouse with streaming data

Next, you'll create an eventhouse to store real-time vital signs data as a time-series entity in your ontology.

1. In your workspace, select **+ New item** > **Eventhouse**.
1. Name the eventhouse `LamnaHealthcareEH` and select **Create**.
1. A default KQL database is created with the same name. Select the KQL database to open it.

### Ingest vital signs data

1. Select the KQL database `LamnaHealthcareEH`, then select **Get data** > **Local file**.
1. In the **Select or create a destination table** section, select **+ New table** and enter `VitalSignsReadings` as the table name.
1. Under **Add up to 1,000 files**, select **Browse for files** and upload the **VitalSignsReadings.csv** file you downloaded earlier.
1. Select **Next**, then continue through the ingestion wizard, keeping the default settings.
1. Select **Finish** to complete the ingestion.
1. Verify the **VitalSignsReadings** table appears in the KQL database with 20 rows.

## Create a semantic model

Now you'll create a Power BI semantic model from your lakehouse. As you define each relationship, think about what it means in the real world: a department *belongs to* a hospital, a room *is part of* a department, a patient *is admitted to* a room, and a piece of equipment *is assigned to* a patient. These business relationships are exactly what ends up in the ontology.

1. In your workspace, open the **LamnaHealthcareLH** lakehouse.

1. From the lakehouse ribbon, select **New semantic model**.

1. In the **New semantic model** dialog, as shown below:
   - In the **Direct Lake semantic model name** field, enter `LamnaHealthcareModel`.
   - Select all five tables: **hospitals**, **departments**, **rooms**, **patients**, **vitalSignEquipment**.
   - Select **Confirm**.
   - The semantic model opens directly, showing your five tables.

   ![Screenshot showing the New semantic model dialog with the model name entered and five tables selected](./media/24-new-semantic-model.png)

 ### Define relationships between tables

Relationships in the semantic model define how tables connect to each other. These relationships become the relationship types in your ontology.

1. In the ribbon, select **Manage relationships** > **+ New relationship**.

2. Create the first relationship with these settings, then select **Save**:
   - **From table**: departments | **Column**: HospitalId
   - **To table**: hospitals | **Column**: HospitalId
   - **Cardinality**: Many to one (*:1)
   - **Cross filter direction**: Both
   - Select **Save**

3. Select **+ New relationship** again and create three more relationships:

   | From table | From column | To table | To column |
   |---|---|---|---|
   | rooms | DepartmentId | departments | DepartmentId |
   | patients | CurrentRoomId | rooms | RoomId |
   | vitalsignequipment | PatientId | patients | PatientId |

   Use the same settings for each: cardinality **Many to one (\*:1)**, cross filter **Both**, then **Save**.

4. Verify your **Manage relationships** pane shows exactly 4 active relationships, matching the image below, then select **Close**.

   ![Screenshot showing the Manage relationships pane with four active relationships](./media/24-semantic-model-relationships.png)

5. In the top navigation bar, select the **×** next to **LamnaHealthcareModel** to close the semantic model. Return to your workspace item list.

6. In your workspace item list, select the **ellipsis (...)** next to **LamnaHealthcareModel** and select **Open data model**.

   > **Note**: Selecting the item name directly opens the report builder, which doesn't have the **Generate Ontology** button. You must use **Open data model** from the ellipsis menu.

## Generate the ontology

Now that your semantic model is ready and your lakehouse data is in place, you'll generate the ontology from the semantic model editor. The semantic model captures your static hospital data — hospitals, departments, rooms, patients, and vital sign equipment. The eventhouse vital signs readings data is time-series data, which is added to the ontology manually after generation.

1. From the top ribbon, select **Generate Ontology**.

   ![Screenshot showing the Generate Ontology button in the semantic model ribbon](./media/24-generate-ontology-button.png)

1. Select your workspace name from the drop-down box.

1. In the name dialog, enter `LamnaHealthcareOntology`, then select **Create**

   > **Tip**: Ontology names can include numbers, letters, and underscores — no spaces or dashes.

Wait a few moments while the system is **Generating Ontology**.

The system generates **5 entity types** (Hospitals, Departments, Rooms, Patients, VitalSignEquipment) with all their properties and **4 relationship types** based on the semantic model relationships. Time-series data from the eventhouse is not included automatically — you'll add that as a separate step after configuring the relationship bindings.

> **Note**: The auto-generated relationship names follow a `{source}_has_{target}` pattern (for example, `departments_has_hospitals`). 

## Set entity type keys

Before you can configure relationship bindings, each entity type needs a key property defined. The key is the column that uniquely identifies each instance — equivalent to a primary key.

1. In the **Entity Types** list, select **Hospitals**.
1. In the **Entity type configuration** pane, under **Key properties**, select **+ Add key**.
1. Select **HospitalId** and select **Save**.
1. Repeat for the remaining entity types:

   | Entity type | Key property |
   |---|---|
   | Departments | DepartmentId |
   | Rooms | RoomId |
   | Patients | PatientId |
   | VitalSignEquipment | EquipmentId |

## Configure relationship bindings

The generated ontology includes relationship type definitions, but you must bind each relationship to its source data table before the connections become queryable. You'll configure one relationship with detailed steps, then use a reference table for the remaining three.

After selecting the source table, you map two columns so the system knows how to identify both entities in each row. Both columns come from the same source table — this is the equivalent of a foreign key relationship in a relational database.

In the image below, the source table is Departments:

- **Section 1 (Source entity type)**: `DepartmentId` — the primary key that identifies this department instance
- **Section 2 (Target entity type)**: `HospitalId` — the foreign key that identifies which hospital this department belongs to

![Screenshot showing relationship configuration pane with source and target entity column mappings](./media/24-relationship-binding.png)

### Configure the Departments–Hospitals relationship

1. In the ontology canvas, select the **departments** entity in the **Entity Types** navigation bar.
2. Select the relationship line between **Departments** and **Hospitals**.

3. In the **Relationship configuration** pane on the right, configure the source data location:
   - **Workspace**: Select your workspace
   - **Lakehouse**: Select **LamnaHealthcareLH**
   - **Schema**: Select **dbo**
   - **Table**: Select **Departments**
   
   > **Note**: Select the Departments table (not Hospitals) because it contains both the department identifier and the hospital reference. The Hospitals table has no column pointing back to departments, so it can't express the direction of the relationship.

4. Configure the entity type mappings:
   - Under **1. Source entity type**: Select **Departments**
     - **Source column**: Select **DepartmentId**
   - Under **2. Target entity type**: Select **Hospitals**
     - **Source column**: Select **HospitalId**

5. Select **Create**.

### Configure remaining relationships

Follow the same process for the remaining three relationships.

| Relationship | Source table | Source entity column | Target entity column |
|---|---|---|---|
| Rooms → Departments | LamnaHealthcareLH > dbo > **Rooms** | Rooms: **RoomId** | Departments: **DepartmentId** |
| Patients → Rooms | LamnaHealthcareLH > dbo > **Patients** | Patients: **PatientId** | Rooms: **CurrentRoomId** |
| VitalSignEquipment → Patients | LamnaHealthcareLH > dbo > **VitalSignEquipment** | VitalSignEquipment: **EquipmentId** | Patients: **PatientId** |

> **Note**: The generator may pre-fill the **source entity column** with the wrong value. The source entity column must match the entity's own key property (shown under **Entity type key property** on the right). If the **Apply** button is disabled, check that the source entity column matches the entity key — for example, `PatientId` for Patients, `RoomId` for Rooms, or `EquipmentId` for VitalSignEquipment.

> **Note**: In the current preview, the **Patients → Rooms** relationship may return a `Contextualization sourceKeyRefBindings` error when you select **Apply**. This is a known preview issue. If you encounter it, skip this relationship and continue — the remaining configuration steps are unaffected.

All four relationships now have source data configured. Your ontology understands the complete healthcare data model: hospitals contain departments, departments contain rooms, patients are admitted to rooms, and vital sign equipment is assigned to patients.

## Add VitalSignsReadings as a time-series entity

The semantic model only contains static lakehouse tables, so the generated ontology has 5 entity types. Now you'll add a sixth — **VitalSignsReadings** — as a time-series entity type sourced from your eventhouse. You'll then connect it to VitalSignEquipment with a relationship.

### Add the VitalSignsReadings entity type

1. In the ontology ribbon, select **Add entity type**.

1. Name the entity type `VitalSignsReadings` and select **Create**.

1. The **Entity type configuration** pane opens. Under **Key properties**, select **+ Add key**, choose **ReadingId**, and select **Save**.

1. Select **Add data to entity type** to configure the data binding.

1. In the **OneLake catalog**, locate **LamnaHealthcareEH** (eventhouse) in your workspace and select it.

1. Select **Connect**.

1. Select the **VitalSignsReadings** table and select **Next**.

1. For **Binding type**, select **Time series**.

1. For **Source data timestamp column**, select **Timestamp**.

1. Map all columns to their matching properties (they should auto-map by name):
   - ReadingId → ReadingId
   - EquipmentId → EquipmentId
   - Timestamp → Timestamp
   - HeartRate → HeartRate
   - OxygenSaturation → OxygenSaturation
   - RespiratoryRate → RespiratoryRate

1. Select **Save** to create the binding.

### Add the generates relationship

Now connect VitalSignEquipment to VitalSignsReadings so the ontology knows which equipment produced which readings.

1. In the ontology ribbon, select **Add relationship type**.

1. Configure the relationship:
   - **Relationship type name**: `generates`
   - **Source entity type**: `VitalSignEquipment`
   - **Target entity type**: `VitalSignsReadings`

1. Select **Add relationship type**.

1. In the **Relationship configuration** pane, configure the source data:
   - **Source type**: Select **Eventhouse**
   - **Eventhouse**: Select **LamnaHealthcareEH**
   - **Table**: Select **VitalSignsReadings**
   - Under **Source entity type (VitalSignEquipment)**: select **EquipmentId** as the source column  
   - Under **Target entity type (VitalSignsReadings)**: select **EquipmentId** as the source column

1. Select **Create**.

Your ontology now has **6 entity types** and **5 relationships**, with all entity data and relationship bindings fully configured.

## Verify the ontology

Your ontology is now complete with static entities from the lakehouse, a time-series entity from the eventhouse, and five relationships connecting them. Let's verify it works.

1. Select **Rooms** from the Entity Types list.
1. In the ontology ribbon, select **Entity type overview**.
1. You'll see an "Updating your ontology" message while the system processes your data in the background. After 1-2 minutes, refresh your browser to display the entity type overview.

   You'll see tiles showing:
   - **Relationship graph**: Visual representation of how Rooms connects to Departments, Patients, and transitively to VitalSignEquipment
   - **Property charts**: Bar charts showing the distribution of values like RoomType
   - **Entity instances table**: All 10 room instances with their properties

1. In the **Entity instances** table, select any room instance (for example, **ICU-302**).
1. The instance view opens, showing properties for this specific room and its connected entities — the department it belongs to, and the patients currently admitted.

1. Navigate to **VitalSignEquipment** and open an instance (for example, **VS-1001**). Verify you can see both static properties (EquipmentId, PatientId, EquipmentType) *and* time-series vital signs readings from the eventhouse.

   This confirms that a single entity instance can combine static reference data with real-time streaming data.

You've successfully created a complete ontology by building a semantic model from your lakehouse, generating the ontology structure automatically, extending it with a time-series entity from an eventhouse, and connecting everything with relationship bindings. Your ontology now represents the healthcare domain with:
- **6 entity types**: Hospitals, Departments, Rooms, Patients, VitalSignEquipment, VitalSignsReadings
- **5 relationships**: all bound to source data and fully queryable
- **Static + time-series data**: combined within a single connected graph

## Clean up resources (optional)

You can keep this workspace and ontology to continue exploring Fabric IQ capabilities. If you want to remove the resources you created in this exercise, follow these steps:

1. In the bar on the left, select the icon for your workspace to view all of the items it contains.
1. Select **Workspace settings**.
1. In the **General** section, select **Remove this workspace**.
