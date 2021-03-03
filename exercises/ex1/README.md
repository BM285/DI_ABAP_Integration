# Exercise 1 - Replicating data from ABAP CDS Views in SAP Data Intelligence

## Some Key Words

**ABAP Development Tools (ADT)**, formerly known as "ABAP in Eclipse", is the integrated ABAP development environment built on top of the Eclipse platform. Its main objective is to support developers in today’s increasingly complex development environments by offering state-of the art ABAP development tools. You can find more information about ADT **[here](https://tools.hana.ondemand.com/#abap)**.<br>
<br>
**CDS (Core Data Services)** is an extension of the ABAP Dictionary that allows you to define semantically rich data models in the database and to use these data models in your ABAP programs. CDS is a central part of enabling code push-down in ABAP applications.<br>
You can find more information about CDS in the **[ABAP keyword documentation](https://help.sap.com/doc/abapdocu_751_index_htm/7.51/en-US/abencds.htm)** and the **[SAP Community](https://community.sap.com/topics/abap)**.<br>
<br>
Starting with SAP S/4HANA Cloud 1905 and SAP S/4HANA 1909 FPS01 (on-premise), **Change Data Capture (CDC)** is supported for ABAP CDS Views. After an ABAP CDS View is enabled for this delta method, changes in tables belonging to this view are recorded by the CDC mechanism. From a technology perspective, this delta method makes use of real-time database triggers on table level based on SLT technology.<br>**INSERT**, **UPDATE** and **DELETE** operations can be recorded by the framework.<br>

Those who are interesed in more information about Change Data Capture for ABAP CDS Views may like to read the related part of the **[Data Intelligence - ABAP Integration Guide](https://help.sap.com/viewer/3a65df0ce7cd40d3a61225b7d3c86703/Cloud/en-US/55b2a17f987744cba62903e97dd99aae.html)** or the blog **[CDS based data extraction – Part II Delta Handling](https://blogs.sap.com/2019/12/16/cds-based-data-extraction-part-ii-delta-handling/)**.<br><br>

## Exercise 1.1 - Create a simple ABAP CDS View in ABAP Develoment Tools (ADT)

After having completed the steps in the first (ABAP) part we will have created two new delta-enabled custom ABAP CDS Views on our SAP S/4HANA system. Our goal is to leverage these CDS Views later on to access the Customer and Sales Order data of the Enterprise Procurement Model (our demo dataset) from Pipelines in SAP Data Intelligence.<br><br>

We are now going to create a CDS (Core Data Services) View using ABAP Development Tools (ADT). In our specific case, it will be a CDS View to access data of the EPM table SNWD_BPA, which contains the Business Partner record set.<br>

The object names in the screenshots may be different from the names proposed in the text. **Please follow the text based instructions**.<br>

1. Create a CDS View
In the context menu of your package choose ***New*** and then choose ***Other ABAP Repository Object***.<br><br>
![](/exercises/dd1/images/1-001a.JPG)

2.	Select ***Data Definition***, then choose ***Next***.<br><br>
![](/exercises/dd1/images/1-002a.JPG)

3. Enter the following values, then choose Next.
- Name ```Z_CDS_EPM_BUPA_TAxx``` (where TAxx is you workshop user name, e.g. Z_CDS_EPM_BUPA_TA99)
- Description: **CDS View for EPM Business Partner Extraction**
- Package: **$TMP**
- Referenced Object: **SNWD_BPA**<br><br>
![](/exercises/dd1/images/1-003a.JPG)

4.	Accept the default transport request (local) by simply choosing ***Next*** again.<br><br>
![](/exercises/dd1/images/1-004a.JPG)

5.	Select the entry ***Define View***, then choose ***Finish***.<br><br>
![](/exercises/dd1/images/1-005a.JPG)

6.	The new view appears in an editor, with an error showing up because of the still missing SQL View name.<br>
In this editor, enter value for the SQL View name in the annotation **`@AbapCatalog.sqlViewName`**, e.g. **`ZSQL_BUPA_TA99`**.<br>
The SQL view name is the internal/technical name of the view which will be created in the database.<br>
**`Z_CDS_EPM_BUPA_TA99`** is the name of the CDS view which provides enhanced view-building capabilities in ABAP. 
You should always use the CDS view name in your ABAP applications.<br><br>
The data source plus its fields have automatically been added to the view definition because of the reference to the data source object we gave in step 3.
If you haven't provided that value before, you can easily search for and add your data source using the keyboard shortcut ***CTRL+SPACE***.<br><br>
![](/exercises/dd1/images/1-006a.JPG)

7.	Delete the not needed fields in the SELECT statement, add the annotation ```@ClientHandling.type: #CLIENT_DEPENDENT``` and beautify the view.<br><br>
   ![](/exercises/dd1/images/1-007a.JPG)<br><br>
   The code may now look as follows:
     ```abap
     @AbapCatalog.sqlViewName: 'ZSQL_BUPA_TA99'
     @AbapCatalog.compiler.compareFilter: true
     @AbapCatalog.preserveKey: true
     @ClientHandling.type: #CLIENT_DEPENDENT
     @AccessControl.authorizationCheck: #CHECK
     @EndUserText.label: 'CDS View for EPM Business Partner Extraction'
     
     define view Z_CDS_EPM_BUPA_TA99
         as select from SNWD_BPA
         
     {
         key node_key as NodeKey,
             bp_role as BpRole,
             email_address as EmailAddress,
             phone_number as PhoneNumber,
             fax_number as FaxNumber,
             web_address as WebAddress,
             address_guid as AddressGuid,
             bp_id as BpId,
             company_name as CompanyName,
             legal_form as LegalForm,
             created_at as CreatedAt,
             changed_at as ChangedAt,
             currency_code as CurrencyCode
     }
     ```

8.	***Save (CTRL+S or disk symbol in menue bar)*** and ***Activate (CTRL+F3 or magic wand symbol in menue bar)*** the CDS View.<br>
(first ![](/exercises/dd1/images/1-008a.JPG) 
then ![](/exercises/dd1/images/1-008b.JPG))<br><br>

9.	We are now able to verify the results in the ***Data Preview*** by choosing ***F8***. Our CDS View data preview should look like this:<br><br>
![](/exercises/dd1/images/1-009a.JPG)<br><br>

We have now successfully created the first simple CDS View in SAP S/4HANA. In the next step we'll enable it for extraction and delta processing based on CDC.

## Exercise 1.2 - Extraction and Delta enablement for simple ABAP CDS Views

Extraction and Delta enablement for simple ABAP CDS Views is pretty easy! The only step to do is adding the `@Analytics` annotation to the view that sets the enabled flag and the change data capture approach.<br>

Let's continue with the simple ABAP CDS View that we have implemented in the previous section and introduce the CDC delta for **`Z_CDS_EPM_BUPA_TA99`**.<br><br>

1. In ADT's Project Explorer, we navigate to our package and then to ***Core Data Services --> Data Definitions*** and double-click on the ABAP CDS View `Z_CDS_EPM_BUPA`.<br><br>
![](/exercises/dd1/images/dd1-010a.JPG)<br><br>

2. Under the existing list of annotations, enter the following lines:
   ```abap
   @Analytics:{
       dataExtraction: {
           enabled: true,
           delta.changeDataCapture.automatic: true
       }
   }
   ```
   <br>![](/exercises/dd1/images/dd1-011a.JPG)<br><br>

3. ***Save*** (CTRL+S or ![](/exercises/dd1/images/1-008a.JPG)) and ***Activate*** (CTRL+F3 or ![](/exercises/dd1/images/1-008b.JPG)) the CDS View.<br><br>

That's it! In this simple case, the framework can derive the relation between the fields of the ABAP CDS View and key fields of the underlying table itself. Whenever a record is inserted, updated or deleted in the underlying table, a record with the respective table key is stored in a generated logging table.<br><br>

## Exercise 1.3 - Creating a more complex ABAP CDS View in ADT (joining multiple tables)

In this part of the Exercise we'll be creating a more complex ABAP CDS View, again using the ABAP Development Tools (ADT). We will implement an ABAP CDS View which will join the EPM tables `SNWD_SO`, `SNWD_SO_I`, `SNWD_PD`, and `SNWD_TEXTS` in order to fetch all Sales Order relevant data, including its positions, products, and product names.<br>

(As a reminder: The entity relationsships of the tables can be found [here](../ex0#short-introduction-to-the-enterprise-procurement-model-epm-in-sap-s4hana).)<br><br>

In a later step, also this ABAP CDS View will be enabled for Extraction and Change Data Capturing (CDC) for an event based processing of Sales Order related deltas to the target storage in S3.<br><br>

Again, the object names in the screenshots may be different from the names proposed in the text. Please follow the text based instructions.<br><br>

1. Create a CDS View
In the context menu of your package choose ***New*** and then choose ***Other ABAP Repository Object***.<br><br>
![](/exercises/dd1/images/1-001a.JPG)

2. Select ***Data Definition***, then choose ***Next***.<br><br>
![](/exercises/dd1/images/1-002a.JPG)

3. Enter the following values, then choose Next.
- Name, e.g. ```Z_CDS_EPM_SO_TA99``` (use your individual workshop user after the last underscore)
- Description: **CDS View for EPM Sales Order object extraction**
- Package: **$TMP**
- Referenced Object: **SNWD_SO**<br><br>
![](/exercises/dd1/images/dd1-012a.JPG)

4. Accept the default transport request (local) by simply choosing ***Next*** again.<br><br>
![](/exercises/dd1/images/1-004a.JPG)

5. Select the entry ***Define View***, then choose ***Finish***.<br><br>
![](/exercises/dd1/images/1-005a.JPG)

6. The new view appears in an editor, with an error showing up because of the still missing SQL View name.<br>
In this editor, enter value for the SQL View name in the annotation **`@AbapCatalog.sqlViewName`**, e.g. **`ZSQL_SO_TA99`**.<br>
The SQL view name is the internal/technical name of the view which will be created in the database. 
**`Z_CDS_EPM_SO_TA99`** is the name of the CDS view which provides enhanced view-building capabilities in ABAP. 
You should always use the CDS view name in your ABAP applications.<br><br>
The pre-defined data source plus its fields have automatically been added to the view definition because of the reference to the data source object we gave in step 3.
If you haven't provided that value before, you can easily search for and add your data source using the keyboard shortcut ***CTRL+SPACE***.<br><br>
![](/exercises/dd1/images/dd1-013a.JPG)

7.	Delete the not needed fields in the SELECT statement, add the annotation ```@ClientHandling.type: #CLIENT_DEPENDENT``` and beautify the view a bit.<br><br>
![](/exercises/dd1/images/dd1-014a.JPG)<br><br>

8. For joining the EPM Sales Order Header table (`SNWD_SO`) with other related EPM tables (Sales Order Item: `SNWD_SO_I`, Product:`SNWD_PD`, Text (e.g. product names):`SNWD_TEXTS`), we can follow two different approaches.<br>
   - **JOINS**, according to classical SQL concepts and always fully executing this join condition whenever the CDS View is triggered.
     An example would be<br>```select from SNWD_SO as so left outer join SNWD_SO_I as item on so.node_key = item.parent_key```.
   - **ASSOCIATIONS**, which are a CDS View specific kind of joins. They can obtain data from the involved tables on Join conditions but the data is only fetched if required. For example, your CDS view has 4 Associations configured and user is fetching data for only 2 tables, the ASSOICATION on other 2 tables will not be triggered. This may save workload and may increase the query performance.<br> An example for a similar join condition with associations would be<br>```select from SNWD_SO as so association [0..1] to SNWD_SO_I as item	on so.node_key = item.parent_key```
   
   In our specific case, we always need to fetch data from all involved tables. Hence, we choose the classical JOIN for this example and include the following lines:<br>
   ...
   ```abap
   left outer join snwd_so_i as item on so.node_key = item.parent_key
   left outer join snwd_pd as prod on item.product_guid = prod.node_key
   left outer join snwd_texts as text on prod.name_guid = text.parent_key and text.language = 'E'
   ```
   ...<br><br>
   ![](/exercises/dd1/images/dd1-014b.JPG)<br><br>

9.	Add the wanted fields from the other tables in the join condition (and don't forget to make all key fields of all involved tables elements of the ABAP CDS View).<br><br>
   ![](/exercises/dd1/images/dd1-015a.JPG)<br><br>
   The ABAP CDS View may now look as follows:
     ```abap
     @AbapCatalog.sqlViewName: 'ZSQL_SO_TA99'
     @AbapCatalog.compiler.compareFilter: true
     @AbapCatalog.preserveKey: true
     @ClientHandling.type: #CLIENT_DEPENDENT
     @AccessControl.authorizationCheck: #CHECK
     @EndUserText.label: 'CDS View for EPM Sales Order object extraction'
     
     define view Z_CDS_EPM_SO_TA99 as select from snwd_so as so
         left outer join snwd_so_i as item on so.node_key = item.parent_key
         left outer join snwd_pd as prod on item.product_guid = prod.node_key
         left outer join snwd_texts as text on prod.name_guid = text.parent_key and text.language = 'E'
     {
         key item.node_key       as ItemGuid,
         so.node_key             as SalesOrderGuid,
         so.so_id                as SalesOrderId,
         so.created_at           as CreatedAt,
         so.changed_at           as ChangedAt,
         so.buyer_guid           as BuyerGuid,
         so.currency_code        as CurrencyCode,
         so.gross_amount         as GrossAmount,
         so.net_amount           as NetAmount,
         so.tax_amount           as TaxAmount,
         item.so_item_pos        as ItemPosition,
         prod.product_id         as ProductID,
         text.text               as ProductName,   
         prod.category           as ProductCategory,
         item.gross_amount       as ItemGrossAmount,
         item.net_amount         as ItemNetAmount,
         item.tax_amount         as ItemTaxAmount,
         prod.node_key           as ProductGuid,
         text.node_key           as TextGuid
     }
     ```

10. ***Save*** (CTRL+S or ![](/exercises/dd1/images/1-008a.JPG)) and ***Activate*** (CTRL+F3 or ![](/exercises/dd1/images/1-008b.JPG)) the CDS View.<br><br>

11. Verify the results in the ***Data Preview*** by pressing ***F8***. Our CDS View data preview should look like this:<br><br>
![](/exercises/dd1/images/dd1-016a.JPG)<br><br>

## Exercise 1.4 - Extraction and Delta enablement for a complex ABAP CDS Views (joining multiple tables)

The main task for exposing a CDS view with CDC delta method is to provide the mapping information between the fields of a CDS view and the key fields of the underlying tables. The mapping is necessary to enable a comprehensive logging for each of the underlying tables and subsequently a consistent selection/(re-)construction of records to be provided for extraction. This means the framework needs to know which tables to log, i.e. monitor for record changes.

Given one record changes in possibly only one of the underlying tables, the framework needs to determine which record/s are affected by this change in all other underlying tables and need to provide a consistent set of delta records.

All key fields of the main table and all foreign key fields used by all on-conditions of the involved join(s) need to be exposed as elements in the CDS views.

The first step is to identify the tables participating in the join and its roles in the join. Currently only Left-outer-to-One joins are supported by the CDC framework. These are the common CDS views with one or more joins based on one main table. Columns from other (outer) tables are added as left outer to one join, e.g. join from an item to a header table.

Given there are no restrictions applied to the CDS view, the number of records of the CDS view constitute the number of records of the main table. All records from the main table are visible in the CDS view. Deletions of a record with regards to this CDS view only happen, if the record in the main table is deleted.

Secondly the developer needs to provide the mapping between the key fields of the underlying tables and their exposure as elements in the CDS view. Please check the following figure in which you see the representation of all underlying key fields surfacing in the CDS view.<br>

![](/exercises/dd1/images/dd1-017a.JPG)<br><br>
***Source:*** [CDS based data extraction – Part II Delta Handling](https://blogs.sap.com/2019/12/16/cds-based-data-extraction-part-ii-delta-handling/) (an excellent blog by Simon Kranig).<br><br>

In case of our EPM tables, we only need to consider one key field per table, which is the field `node_key` in all cases.
For mapping the key fields of all underlying tables to the fields of the CDS view, we use the annotation `Analytics.dataExtraction.delta.changeDataCapture.mapping`. This implies the requirement to make the key fields of all tables in the join condition elements of the ABAP CDS View.<br><br>

1. Include the following code under the list of existing annotations:
   ```abap
   @Analytics:{
       dataCategory: #FACT,
       dataExtraction: {
           enabled: true,
           delta.changeDataCapture: {
               mapping:
               [{table: 'SNWD_SO',
                 role: #MAIN,
                 viewElement: ['SalesOrderGuid'],
                 tableElement: ['node_key']
                },
                {table: 'SNWD_SO_I',
                 role: #LEFT_OUTER_TO_ONE_JOIN,
                 viewElement: ['ItemGuid'],
                 tableElement: ['node_key']
                },
                {table: 'SNWD_PD',
                 role: #LEFT_OUTER_TO_ONE_JOIN,
                 viewElement: ['ProductGuid'],
                 tableElement: ['node_key']
                },
                {table: 'SNWD_TEXTS',
                 role: #LEFT_OUTER_TO_ONE_JOIN,
                 viewElement: ['TextGuid'],
                 tableElement: ['node_key']
                }
               ]
          }
       }
   }
   ```
   ![](/exercises/dd1/images/dd1-018a.JPG)<br><br>

2. ***Save*** (CTRL+S or ![](/exercises/dd1/images/1-008a.JPG)) and ***Activate*** (CTRL+F3 or ![](/exercises/dd1/images/1-008b.JPG)) the CDS View.<br><br>

3. Verify the results in the ***Data Preview*** by pressing ***F8***. The ABAP CDS View should still provide the same data as before delta-enabling.<br><br>

We have now enabled our Sales Order object CDS View for Change Data Capture and are able to obtain any delta from one of the involved tables.<br>

In the next session, we test the Initial Load and Delta Load capabilities with a small, exemplary Data Intelligence Pipeline implementation.

In the following exercise sections, we will now leverage the new ABAP CDS Views as a source for data processing in Data Intelligence Pipelines.<br><br>

## Exercise 1.5 - Consuming the EPM Business Partner ABAP CDS Views in SAP Data Intelligence

The next steps will comprise
- obtaining the Business Partner master data in S/4HANA's demo application **Enterprise Procurement Model (EPM)** and make the records available for the corporate Data Analysts in an S3 object store.
- also persisting the transactional data for EPM Sales Orders in S3.
- In both cases, any single change of these data sources in the S/4HANA system has to be instantly and automatically replicated to the related files in S3 in append mode.
- Additionally, the Sales Order data have to be enriched with Customer master data. For the initial load and then on every change committed to the EPM Sales Order data in S/4HANA.

(As a reminder: You can recap the relationship between the relevant EPM table entities that are used in this exercise [here](../ex0#short-introduction-to-the-enterprise-procurement-model-epm-in-sap-s4hana))<br>

After completing these steps you will have created a Pipeline that reads EPM Customer data from an ABAP CDS View in S/4HANA and displays it in a Terminal UI.<br>
For the integration of ABAP CDS Views of S/4HANA, SAP Data Intelligence provides standard ABAP operator shells that can easily be configured according to your use cases.<br><br>

1. Log on to SAP Data Intelligence and enter the Launchpad application. Then start the ***Modeler*** application.
   - Follow the link to your assigned Data Intelligence instance, i.e. https://vsystem.ingress.dh-6srbrjhsl.dh-canary.shoot.live.k8s-hana.ondemand.com/login/?tenant=dat262.
   - The tenant name is ***"workshop"*** (not "dat262" like in the screenshots).
   - In the next pop-up window, enter your assigned DI user name (e.g. ***"TA99"***) and your individual password received from the user registration.<br><br>
   ![](/exercises/ex1/images/ex1-002c.JPG)<br><br>
   - From the Launchpad, start the ***Modeler*** application by clicking on the corresponding tile.<br><br>
   ![](/exercises/ex1/images/ex1-003b.JPG)<br><br>

2.	Make sure you are in the ***Graphs*** tab of the Modeler UI (see left side). Then click the ***+*** symbol in order to create a new Pipeline.<br><br>
![](/exercises/ex1/images/ex1-004b.JPG)<br><br>

3.	Now a new Pipeline canvas is opened on the right side and the Modeler UI automatically switches to the ***Operators*** tab (see left side). In the list of operators, drag the ABAP CDS Reader and drop it into the Pipeline canvas. Click the ABAP CDS Reader node in the canvas one time and then click the ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-006b.JPG)<br><br>

4.	In the configuration panel for the ABAP CDS Reader, specify the S/4HANA Connection: Select `S4_RFC_TechEd`. Then click on the ***Version*** selection icon.<br><br>
![](/exercises/ex1/images/ex1-007b.JPG)<br><br>

5.	In the pop-up window, mark the different versions of the ABAP CDS Reader and read through their documentation. As you can see, the ***ABAP CDS Reader V2*** provides a standard message type output (no conversion from ABAP object required). select this entry and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-008b.JPG)<br><br>

6.	The ABAP CDS Reader operator shell has received the operator's metadata from S/4HANA and has configured the Pipeline node (for example, the operator now has an output port). Now fill in the configuration parameters in the panel:
      - Label: `Buyer - S/4 ABAP CDS`
      - Subscription Type: `New`
      - Subscription Name: `BP001_XXXX`, where XXXX is your user name, for example "BP001_TA99"
      - ABAP CDS Name: `Z_CDS_BUYER_Delta`, which is the "simple" CDS View that got created in the Deep Dive 1 demo
      - Transfer Mode: `Initial Load`
      - Leave all other parameters as-is.<br><br>
   ![](/exercises/ex1/images/ex1-009b.JPG)<br><br>

7.	From the operator list on the left side, drag the ***Terminal*** operator and drop it into the Pipeline canvas. Then connect the output port of the ABAP CDS Reader with the input port of the Terminal operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-010b.JPG)<br><br>

8.	The Terminal displays string type data. Because the output of the ABAP CDS Reader is a message, its body/payload needs to be converted into a string. After you have connected the operator ports you are, hence, prompted for a conversion approach. Select the first of the two options (only the body will be processed) and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-011b.JPG)<br><br>

9.	The selected converter is now added to the Pipeline. Please click on the ***Auto Layout*** button to arrange the Pipeline nodes.<br><br>
![](/exercises/ex1/images/ex1-012b.JPG)<br><br>

10.	***Save*** your Pipeline.
      - Click on the Disk symbol in the menue bar.<br><br>
      ![](/exercises/ex1/images/ex1-013b.JPG)<br><br>
      - Because you save the Pipeline for the first time, you are prompted for some inputs.<br>
      As Name, enter **`teched.XXXX.EPM_Customer_Replication_to_S3`**, where **XXXX** is your user name, for example "teched.TA99.EPM_Customer_Replication_to_S3".<br>
      As Description, please enter **`XXXX - Replicate S/4HANA EPM Customer Data to S3`**,where **XXXX** is your user name, for example "TA99 - Replicate S/4HANA EPM Customer         Data to S3". (The description will show up in the Pipeline status information later on.)<br>
      As Catergory, enter **`dat262`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
      Click ***OK***.<br><br>
      ![](/exercises/ex1/images/ex1-014b.JPG)<br><br>

11.	After you have saved the Pipeline, it will get validated by SAP Data Intelligence. Check the validation results. If okay, you can now execute the Pipeline by clicking the ***Play*** icon in the menue bar. Then change to the ***Status*** tab in the Modeler UI status section on the lower right side.<br><br>
![](/exercises/ex1/images/ex1-015b.JPG)<br><br>

12.	Once the status of your Pipeline has changed to ***running***, click on the ***Terminal*** operator node one time and then on its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-016b.JPG)<br><br>

13.	You should now see that EPM Customer master data is coming in, which proves that the integration with the S/4HANA CDS View is working as expected. By the way: the truncation of payload content can be determined by the ***Max size (bytes)*** parameter in the Terminal operator configuration.<br><br>
![](/exercises/ex1/images/ex1-017b.JPG)<br><br>

14.	***Stop*** the Pipeline execution again by clicking the marked icons in the menue bar or in the status section. The status of the Pipeline will then change from ***running*** to ***stopping*** and finally to ***completed***.<br><br>
![](/exercises/ex1/images/ex1-018b.JPG)<br><br>

**Congratulations!** You have now successfully implemented a Pipeline that consumes an ABAP CDS View from the connected SAP S/4HANA system.
As a next step, you will persist the EPM Customer master data in an S3 Object Store and switch from the Initial Load to a Delta Load transfer mode.

## Exercise 1.6 - Extend the Pipeline to transfer the Customer data into an S3 Object Store with Initial Load and Delta Load modes

After completing the following steps you will have extended the Data Intelligence Pipeline with an additional persistency for the data in S3. And in order to also obtain any deltas that have occurred on the S/4HANA EPM Business Partner table `SNWD_BPA`, the transfer mode will get changed to Delta Load after initially loading the data into a file in S3.

1. If not already done, open the Pipeline from the previous section (**`teched.XXXX.EPM_Customer_Replication_to_S3`**, where XXXX is your user name). Click on the Terminal operator and then the "waste bin" icon in order to delete the Terminal operator. Do the same for the Converter operator. Just keep the the ABAP CDS Reader operator in the Pipeline canvas.<br><br>
![](/exercises/ex1/images/ex1-019b.JPG)<br><br>

2. Make sure that the ***Operators*** tab is in scope in the Modeler UI (see left side). From the list of operators, drag the ***Wiretap*** operator and drop it into the Pipeline canvas. Similar to a Terminal operator, this operator can wiretap a connection between two operators in a Pipeline and display the traffic to the browser window (or to an external websocket client that connects to this operator). But the Wiretap operator also supports throughput of type *message* (amongst others), so that no type conversion is needed between the ABAP CDS Reader and the Wiretap operator.<br>
Now connect the **output port of the ABAP CDS Reader** with the **input port of the Wiretap operator** by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-020b.JPG)<br><br>

3. From the list of operators, drag the ***Write File*** operator and drop it in the Pipeline canvas. Then connect the **output port of the Wiretap operator** with the **input port of the Write File operator** by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-021b.JPG)<br><br>

4. The message format from the Wiretap operator output must be transformed to a file format. For this reason you are prompted to choose an appropriate converter operator. On the pop-up window, select the first option (transfer the content). Click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-022b.JPG)<br><br>

5. Click on the ***Write File*** operator and click its ***configuration*** icon. (As a reminder: You can always change from the configuration panel to the online documentation of each standard Pipeline operator, by clicking on the "Show Documentation" icon ![](/exercises/ex1/images/Show_Documentation.JPG) on the upper right side of the Modeler UI above the configuration panel).<br><br>
![](/exercises/ex1/images/ex1-023b.JPG)<br><br>

6. Enter the needed configuration parameters for the ***Write File*** operator. These are:
   - Label: **`Customer Master to S3`**
   - Connection: Choose type ***Connection Management*** and then the connection ID **`TechEd2020_S3`**
   - Path mode: **`Static (from configuration)`**
   - Path: **`/XXXX/Customer_Master.csv`**, where XXXX is your User name, for example "/TA99/Customer_Master.csv"
   - Mode: **`Append`**
   - Join batches: **`True`**<br><br>
![](/exercises/ex1/images/ex1-024b.JPG)<br><br>

7. Now ***Save*** your Pipeline, verify the validation results and - if okay - run the Pipeline by clicking on the ***Play*** symbol in the menue bar.<br><br>
![](/exercises/ex1/images/ex1-025b.JPG)<br><br>

8. Once the Pipeline has the status ***running***, click on the ***Wiretap*** operator and then click its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-026b.JPG)<br><br>

9. In the ***Wiretap UI*** you should now see the processed messages coming from the ABAP CDS Reader. (As you can also see, the message header contains the field names. For simplicity reasons, we're not going to extract these during this exercise, but stay with the generic C0..C99 column names).<br><br>
![](/exercises/ex1/images/ex1-027b.JPG)<br><br>

10. ***Stop*** the Pipeline.<br><br>
![](/exercises/ex1/images/ex1-028b.JPG)<br><br>

11. As a verification of the successful load of the data to the file in S3, we can use the ***Metadata Explorer*** as integrated part of SAP Data Intelligence. Please go back to the Launchpad and click on the corresponding tile.<br><br>
![](/exercises/ex1/images/ex1-029b.JPG)<br><br>

12. In the ***Metadata Explorer*** application main menue, click on ***Browse Connections***.<br><br>
![](/exercises/ex1/images/ex1-030b.JPG)<br><br>

13. In order to easily find our connection to the target S3 Object Store, you may leverage the search functionality. Enter `TechEd` into the search field and click on the spyclass icon. Click on the **`TechEd2020_S3`** tile.<br><br>
![](/exercises/ex1/images/ex1-031b.JPG)<br><br>

14. On the next drill-down view, click on the directory that you had specified in the ***Write File*** operator, and then drill down to your specific User directory, for example **`TA99`**.<br><br>
![](/exercises/ex1/images/ex1-032b.JPG)<br><br>

15. If your Pipeline ran successfully, you'll find a file with your specified name (`Customer_Master.csv`) Open the ***More Actions*** menue and select ***View Fact Sheet***.<br><br>
![](/exercises/ex1/images/ex1-033b.JPG)<br><br>

16. In the ***Fact Sheet***, which provides some more overview and statistical information about the new file, go to tab ***Data Preview***.<br><br>
![](/exercises/ex1/images/ex1-034b.JPG)<br><br>

17. Now you can see that the EPM Customer data got loaded into the target file in S3. Success!<br><br>
![](/exercises/ex1/images/ex1-035b.JPG)<br><br>

In order to fetch any changes in S/4HANA on the Business Partner table `SNWD_BPA`, we can leverage the delta enablement in the ABAP CDS View that we had conducted during the Deep Dive 1 demo by adding the `@Analytics`annotation below:<br>
```abap
@Analytics:{
    dataExtraction: {
        enabled: true,
        delta.changeDataCapture.automatic: true
    }
}
```
<br>

17. Go back to the Modeler application and your Pipeline. Click on the ***ABAP CDS Reader*** operator and then on its ***configuration*** icon. In the configuration panel, change the entry for the ***Transfer mode*** parameter from `Initial Load`--> `Delta Load`.<br><br>
![](/exercises/ex1/images/ex1-036b.JPG)<br><br>

18. ***Save*** the Pipeline and ***Run*** it again. As long as the Pipeline stays in a ***running*** status, it will fetch any updates on the Business Partner table of the EPM demo application in S/4HANA by leveraging the Change Data Capturing (CDC) functionality in S/4HANA ABAP CDS Views. Your target persistency in S3 is instantly and automatically kept updated.<br><br>
![](/exercises/ex1/images/ex1-037b.JPG)<br><br>

**Very well done!** You have implemented a Pipeline that extracts Initial Load and Delta data fron an ABAP CDS View in S/4HANA and have interlinked it with a non-SAP target storage in S3.

## Exercise 1.7 - Implement a Pipeline for delta transfer of enhanced EPM Sales Order data from S/4HANA to an S3 Object Store

In the next section, we'll also take care for the Sales Order transaction data from EPM and will right away establish a replication (initial load plus following delta processing) transfer mode.

1. In SAP Data Intelligence, open the ***Modeler*** application. Make sure the scope of the Modeler UI is on tab ***Graphs*** (see left side). Then click the ***+*** button to create a new Pipeline.<br><br>
![](/exercises/ex1/images/ex1-004b.JPG)<br><br>

2. Now a new Pipeline canvas is opened on the right side and the Modeler UI automatically switches the scope to the ***Operators*** tab (see left side). In the list of operators, drag the ABAP CDS Reader and drop it into the Pipeline canvas. Click the ABAP CDS Reader node in the canvas one time and then click the ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-006b.JPG)<br><br>

3.	In the configuration panel for the ABAP CDS Reader, specify the S/4HANA Connection: Select `S4_RFC_TechEd`. Then click on the ***Version*** selection icon.<br><br>
![](/exercises/ex1/images/ex1-007b.JPG)<br><br>

4.	In the pop-up window, select the option ***ABAP CDS Reader V2*** and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-008b.JPG)<br><br>

6.	The ABAP CDS Reader operator shell has received the operator's metadata from S/4HANA and has configured the Pipeline node (for example, the operator now has an output port). Now fill in the configuration parameters in the panel:
      - Label: `Sales Order - S/4 ABAP CDS`
      - Subscription Type: `New`
      - Subscription Name: `SO001_XXXX`, where XXXX is your user name, for example "SO001_TA99"
      - ABAP CDS Name: `Z_CDS_SO_SOI_Delta`, which is the "more complex" CDS View that got created in the Deep Dive 1 demo
      - Transfer Mode: `Replication`
      - Leave all other parameters as-is.<br><br>
   ![](/exercises/ex1/images/ex1-038b.JPG)<br><br>
   
7.	From the operator list on the left side, drag the ***Wiretab*** operator and drop it into the Pipeline canvas. Then connect the output port of the ABAP CDS Reader with the input port of the Wiretab operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-039b.JPG)<br><br>

8. From the list of operators, drag the ***Write File*** operator and drop it in the Pipeline canvas. Then connect the **output port of the Wiretap operator** with the **input port of the Write File operator** by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-040b.JPG)<br><br>

9. The message format from the Wiretap operator output must be transformed to a file format. For this reason you are prompted to choose an appropriate converter operator. On the pop-up window, select the first option (transfer the content). Click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-041b.JPG)<br><br>

10. Click on the ***Write File*** operator and click its ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-042b.JPG)<br><br>

11. Enter the needed configuration parameters for the ***Write File*** operator. These are:
   - Label: **`Sales Order to S3`**
   - Connection: Choose type ***Connection Management*** and then the connection ID **`TechEd2020_S3`**
   - Path mode: **`Static (from configuration)`**
   - Path: **`/XXXX/Sales_Order.csv`**, where XXXX is your User name, for example "/TA99/Sales_Order.csv"
   - Mode: **`Append`**
   - Join batches: **`True`**<br><br>
![](/exercises/ex1/images/ex1-043b.JPG)<br><br>

12.	***Save*** your Pipeline.
      - Click on the Disk symbol in the menue bar.
      - Because you save the Sales Order Pipeline for the first time, you are prompted for some inputs.<br>
      - As Name, enter **`teched.XXXX.EPM_SalesOrder_Replication_to_S3`**, where **XXXX** is your user name, for example "teched.TA99.EPM_SalesOrder_Replication_to_S3".<br>
      - As Description, please enter **`XXXX - Replicate S/4HANA EPM Customer Data to S3`**,where **XXXX** is your user name, for example "TA99 - Replicate S/4HANA EPM Sales  Order Data to S3". (The description will show up in the Pipeline status information later on.)<br>
      - As Catergory, enter **`dat262`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
      - Click ***OK***.<br><br>
      ![](/exercises/ex1/images/ex1-044b.JPG)<br><br>

13.	After you have saved the Pipeline, it will get validated by SAP Data Intelligence. Check the validation results. If okay, you can now execute the Pipeline by clicking the ***Play*** icon in the menue bar. Then change to the ***Status*** tab in the Modeler UI status section on the lower right side.<br><br>
![](/exercises/ex1/images/ex1-045b.JPG)<br><br>

14.	Once the status of your Pipeline has changed to ***running***, click on the ***Wiretap*** operator node one time and then on its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-046b.JPG)<br><br>

15. In the ***Wiretap UI*** you should now see the processed Sales Order messages coming from the ABAP CDS Reader. This proves that the integration with the S/4HANA CDS View is working as expected.<br><br>
![](/exercises/ex1/images/ex1-047b.JPG)<br><br>

16. For validating the correct creation of the file in S3, please leverage the Data Intelligence Metadata Explorer again. Go back to the Launchpad and start the Metadata Explorer application, if not already launched from the previous exercise.<br><br>

17. In the ***Metadata Explorer*** application main menue, click on ***Browse Connections***.<br><br>
![](/exercises/ex1/images/ex1-030b.JPG)<br><br>

18. In order to easily find our connection to the target S3 Object Store, you may leverage the search functionality. Enter `TechEd` into the search field and click on the spyclass icon. Click on the **`TechEd2020_S3`** tile.<br><br>
![](/exercises/ex1/images/ex1-031b.JPG)<br><br>

19. On the next drill-down view, click on the directory that you had specified in the ***Write File*** operator, and then drill down to your specific User directory, for example **`TA99`**.<br><br>
![](/exercises/ex1/images/ex1-032b.JPG)<br><br>

20. If your Pipeline ran successfully, you'll find a file with your specified name (`Sales_Order.csv`) Open the ***More Actions*** menue and select ***View Fact Sheet***.<br><br>
![](/exercises/ex1/images/ex1-048b.JPG)<br><br>

21. In the ***Fact Sheet***, which provides some more overview and statistical information about the new file, go to tab ***Data Preview***.<br><br>
![](/exercises/ex1/images/ex1-049b.JPG)<br><br>

22. Success! Now you can see that the EPM Customer data got loaded into the target file in S3. While the Pipeline is running, this file would get automatically updated with each change in the S/4HANA data sources (the tables `SNWD_SO`, `SNWD_SO_I`, `SNWD_PD`, `SNWD_TEXTS` joined through the ABAP CDS View `Z_CDS_SO_SOI_Delta`).<br><br>
![](/exercises/ex1/images/ex1-050b.JPG)<br><br>

21. However, in order to go on with the exercise, please stop the Pipeline for now. We will validate the delta processing in Exercise 2.<br><br>
![](/exercises/ex1/images/ex1-051b.JPG)<br><br>

**Congratulations!** You have created the Sales Order extraction from a delta-enabled, more complex ABAP CDS View into the S3 Object Store. Because you have chosen the transfer mode ***"Replication"*** in the CDS Reader operator configuration, the Pipeline has conducted the Initial Load and would now wait for any changes (inserts, updates, deletions) in the S/HANA EPM Sales Order object as long as it is running.

As a next step, you will enrich the Sales Order data with some Customer Details (Name and Legal Form) during the Replication process.


## Exercise 1.8 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3

In this last part of the S/4HANA ABAP CDS View intergration exercise, you will establish a Pipeline that replicated the Sales Order data from the delta-enabled ABA CDS View in S/4HANA and joins it with some of the details that you have replicated from the Customer master data, i.e. company name and legal form.

1. In SAP Data Intelligence, open the ***Modeler*** application. Make sure the scope of the Modeler UI is on tab ***Graphs*** (see left side). If not still open from the previous exercise part, search for your Sales Order Pipeline by entering your user name into the search field on top of the list of Pipelines. Click on Pipeline with the label `XXXX - Replicate S/4HANA EPM Sales  Order Data to S3`, where XXXX is your user name, for example "TA99 - Replicate S/4HANA EPM Sales  Order Data to S3". The Pipeline will get opened then or will get the focus on the UI if it was alraedy opened. If you cannot see the complete label of the Pipeline, just hover with your mouse over the pipeline item in the list.<br><br>
![](/exercises/ex1/images/ex1-052b.JPG)<br><br>

2. Click on the pull-down menue of the ***Save*** icon (disk symbol) and choose ***Save As***.<br><br>
![](/exercises/ex1/images/ex1-053b.JPG)<br><br>

3. You are prompted for some inputs. Please enter the following values.
    - As Name, enter **`teched.XXXX.EPM_SalesOrder_Replication_Enrich_to_S3`**, where **XXXX** is your user name, for example "teched.TA99.EPM_SalesOrder_Replication_Enrich_to_S3".<br>
    - As Description, please enter **`XXXX - Replicate and Enrich S/4HANA EPM Sales Order Data to S3`**,where **XXXX** is your user name, for example "TA99 - Replicate and Enrich S/4HANA EPM Sales Order Data to S3". (The description will show up in the Pipeline status information later on.)<br>
    - As Catergory, enter **`dat262`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
    - Then click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-054b.JPG)<br><br>

4. A new Pipeline has been created as a copy of your original Sales Order replication Pipeline. You will be using the File Operator outputs (i.e. the file path and name) as a trigger for the join processes with the Customer master data.<br><br>

5. Open the configuration of the File Writer operator and enter the new target path/file name **`/TA99/Sales_Order_Staging.csv`**. Change Mode to **`Overwrite`**. Then save the Pipeline.<br><br>
![](/exercises/ex1/images/ex1-055b.JPG)<br><br>

6. Click on the ***Operators*** tab on the left side in order to get the list of operators. Find the ***Graph Terminator*** operator and drag it from the operator list into the canvas. Then connect the upper output port ("file") of the File Writer with the input port of the Graph Terminator operator.<br><br>
The Graph Terminanor allows us to run the Pipeline once, and when the new file got generated, the File Writer will send a termination signal to the Graph Terminator, which then stops the Pipeline. We do this just for the purpose of generating the new output file as a reference / import sample for the next steps.<br><br>
![](/exercises/ex1/images/ex1-056b.JPG)<br><br>

7. ***Save*** and ***Run*** the Pipeline. Wait until the Pipeline status turns from ***pending*** to ***processing*** and finally to ***completed***.<br><br>

8. After the completion of the Pipeline execution, **remove the Graph Terminator** operator from the Pipeline again.<br><br>
![](/exercises/ex1/images/ex1-057b.JPG)<br><br>

9. From the list of operators on the left side, drag the ***Structured File Consumer*** operator and drop it into the Pipeline canvas. Then open the configuration panel and do the following entries:
   - **Storage Type**: `S3`
   - **S3 Configuration**: After clicking on the pencil icon, select `Configuration Manager`and as connection ID `TechEd2020_S3`<br><br>
   - Increase the **Fetch size** to `10000`
   - **Fail on string truncation** should be `False`<br><br>
     ![](/exercises/ex1/images/ex1-058b.JPG)<br><br>
   - Then click the **S3 Source File** folder icon to browse through the S3 bucket. Drill down to your individual folder and select the file **`Sales_Order_Staging`**. Then click ***OK***.<br><br>
     ![](/exercises/ex1/images/ex1-060b.JPG)<br><br>
     
10. You are now able to open the ***Data Preview*** of the connected file in S3. Give it a try, if you want!<br>
Now connect the **upper output port ("file") of the File Writer** operator with the **input port of the Structured File Consumer** operator. The ***Structured File Consumer*** operator takes the signal on the input port just as a trigger for commencing its logic. It's an optional input but our approach ensures that the operator is only executed after the file from the previous node has been successfully written on S3. (If the input port of a Structured File Consumer is not connected, the operator will start with the Pipeline execution.)<br>
When you create the link between the operators, a conversion of the data type is required from type `message.file` to type `message`. The ***Converter*** is automatically proposed when the link between the ports is established. Choose the option for ***Path extraction*** (which reflects the minimum transfer payload, as we use the input just as a trigger signal).<br><br>
![](/exercises/ex1/images/ex1-061b.JPG)<br><br>
![](/exercises/ex1/images/ex1-062b.JPG)<br><br>

11. We still need another ***Structured File Consumer*** operator for the Customer master data extraction from S3. In order to save typing efforts, just copy the existing operator:<br>
   - Click on the ***Structured File Consumer*** operator
   - Press <CTRL+C> on your keyboard
   - Press <CTRL+V> on your keyboard
A new copy of the operator is being included in the Pipeline canvas.<br><br>
![](/exercises/ex1/images/ex1-063b.JPG)<br><br>

12. On this new operator, open the configuration panel and click the **S3 Source File** folder icon to browse through the S3 bucket. Drill down to your individual folder and select the file **`Customer_Master.csv`**. Then click ***OK***..<br><br>
![](/exercises/ex1/images/ex1-064b.JPG)<br><br>
![](/exercises/ex1/images/ex1-065b.JPG)<br><br>

13. From the list of operators on the left side, drag the ***Data Transform*** operator and drop it into the Pipeline canvas. Then connect the ***output ports of the Structured File Consumers*** with the ***Data Transform box***. This will create the needed input ports of the Data Transform operator.<br><br>
![](/exercises/ex1/images/ex1-066b.JPG)<br><br>

14. Double-click on the ***Data Transform*** operator.<br><br>
![](/exercises/ex1/images/ex1-067b.JPG)<br><br>

15. In the details view, from the list of Data Operations on the left side, drag the ***Join*** operation into the canvas with the existing two ***input nodes*** and connect these with the ***Join*** operation.<br><br>
![](/exercises/ex1/images/ex1-068b.JPG)<br><br>

16. Double-click on the ***Join*** operation.<br><br>
![](/exercises/ex1/images/ex1-069b.JPG)<br><br>

17. Re-arrange the position of input tables on the canvas as per your convenience and then connect the **"C5" field of Join_Input1** (Sales Order) with the **"C0" field of Join_Input2** (Customer). This will map the Customer GUID fields of the two tables. We're using an INNER JOIN.<br><br>
![](/exercises/ex1/images/ex1-070b.JPG)<br><br>

18. Proceed to tab ***Columns*** in order to select the output fields. Click on the ***Auto map by name*** icon (see red box), which enter all fields of the two input nodes to the list of output fields. Then mark the buttom occurrences of the output fields "C0" to "C7" (of Join_Input2!) and delete these.<br><br>
![](/exercises/ex1/images/ex1-073b.JPG)<br><br>

19. Now mark the buttom occurrences of the output fields "C10" to "C18" (of Join_Input2!) and delete these.<br><br>
![](/exercises/ex1/images/ex1-074b.JPG)<br><br>

20. Click on the ***pencil*** icon in the line of field "C8" in order to edit the entry. Rename **`C8`** to **`CUSTOMER_NAME`**. Click on the white space in the the form, then ***OK***.<br><br>
![](/exercises/ex1/images/ex1-075b.JPG)<br><br>

21. Click on the ***pencil*** icon in the line of field "C9" in order to edit the entry. Rename that field from **`C9`** to **`LEGAL_FORM`**. Click on the white space in the the form, then ***OK***. (Renaming the fields was necessary in order to avoid double names).<br><br>
![](/exercises/ex1/images/ex1-076b.JPG)<br><br>

22. Navigate back.<br><br>
![](/exercises/ex1/images/ex1-077b.JPG)<br><br>

23. Click with the right mouse button on the output port of the ***Join*** operation. Then click on ***Create Data target***.<br><br>
![](/exercises/ex1/images/ex1-078b.JPG)<br><br>

24. A new output node has been created, which will now be available on the Data Transfer operator. Navigate further back.<br><br>
![](/exercises/ex1/images/ex1-079b.JPG)<br><br>

25. Back on the Pipeline canvas, drag the ***Structured File Producer*** operator from the list of operators into the Pipeline canvas. Then connect the ***output port of the Data Transform*** with the ***input port of the Structured File Producer***.<br><br>
![](/exercises/ex1/images/ex1-080b.JPG)<br><br>

26. Open the configuration panel of the ***Structured File Producer*** operator and enter the following parameters:<br>
    - **Storage Type**: `S3`
    - **S3 Configuration**: After clicking on the pencil icon, select `Configuration Manager`and as connection ID `TechEd2020_S3`
    - **S3 file name:** `/TA99/Enriched_Sales_Order.csv`
    - **Format:** `CSV`
    - Leave the **CSV Properties** as-is
    - Increase **Batch size:** to `10000`
    - **Write Part Files** should be `False`<br><br>
    - ***Save*** the Pipeline
    - ***Run*** the Pipeline<br><br>
![](/exercises/ex1/images/ex1-081b.JPG)<br><br>

27. When the Pipeline is in status ***running***, you can take a look into your folder in the S3 bucket, again using the ***Data Intelligence Metadata Explorer*** application. You should see the file that was specified in the *Structured File Producer* operator ("Enriched_Sales_Order.csv"). Click the ***Glasses*** icon on that tile in order to see the ***Data Preview*** of the ***Fact Sheet***.<br><br>
![](/exercises/ex1/images/ex1-082b.JPG)<br><br>

28. As you can see, the two columns with the company name and the legal form of the Customer master data table have been added to the Sales Order data.<br><br>
![](/exercises/ex1/images/ex1-083b.JPG)<br><br>

29. As long as the Pipeline is running, you would now receive any updates in S/4HANA on the EPM Sales Order data, enriched with the lookups of the EPM Customer master data. The file in S3 would look as displayed below. In column "C20", you get the ***update*** indicator "U". Columns "C21" and "C22" contain the enriched field contents.<br><br>
![](/exercises/ex1/images/ex1-084b.JPG)<br><br>

## Summary

**Congrats!** You have now implemented a Pipeline that receives the Initial Loads and the Change Data Capture (CDC) information from different S/4HANA ABAP CDS Views.<br>

As a matter of fact, in this virtual workshop at TechEd, we can't provide access to the SAP GUI of the connected SAP S/4HANA system in order to run the EPM data generation transaction **`SEPM_DG`**.

In the next exercise we will mitigate this problem and allow you to execute a variant of this ABAP Report, leveraging the ABAP Function Module calls from SAP Data Intelligence.
Please continue to [Exercise 2 - Triggering the execution of a function module in a remote S/4HANA system](../ex2/README.md).

<br>
