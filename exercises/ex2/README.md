# Exercise 2 - Calling an ABAP function module in SAP S/4HANA from SAP Data Intelligence

In this section we will demonstrate how to create a custom ABAP operator in SAP S/4HANA and trigger its execution from a Pipeline in SAP Data Intelligence. The ABAP operators are developed in the S/4HANA system as normal development objects, hence, they can be transported within the system landscape.<br>

The basis for the integration with SAP Data Intelligence are the ABAP Pipeline Engine in SAP S/4HANA and the ABAP Subengine in SAP Data Intelligence, both linked with either an RFC or a Websocket RFC connection.

After a new ABAP Operator has been created, it can immediately be used in the SAP Data Intelligence Modeler by leveraging its dynamic ABAP Integration operator. These operators in SAP Data Intelligence are actually shells that point to the original implementation in the connected S/4HANA system.

There are two variants of this operator type available in Data Intelligence:
1. SAP ABAP Operator: This can be used with any ABAP operator delivered by SAP (in namespace `com.sap`). Examples for out-of-the-box operators are (the already known) ABAP CDS Reader, ODP Reader, SLT Connector, Cluster Table Splitter (for Business Suite systems), and the ABAP Converter. 
2. Custom ABAP Operator: This can be used with any ABAP operator created by customers (in namespace `customer.<xyz>`).<br><br>

![](/exercises/dd2/images/dd2-001a.JPG)

In order to use the ABAP Subengine, the following prerequisites have to be met:
1. A supported ABAP system is available (see table below)
2. The ABAP system can be reached via RFC or WebSocket RFC (HTTPS is still supported but will get deprecated soon)
3. A user with the necessary authorizations (see SAP Note [2855052](https://launchpad.support.sap.com/#/notes/2855052))
4. An RFC or Websocket RFC connection has been created in the Connection Manager
5. Whitelisting of operators and data objects (tables, views, etc.) in the S/4HANA system (see SAP Note [2831756](https://launchpad.support.sap.com/#/notes/2831756)).<br>

The ABAP Pipeline Engine is supported starting from the following releases (you can also run this scenario with a SAP Business Suite system, but then it is required to install the (non-modifying) DMIS add-on on that system.)<br>
ABAP Edition | Minimum version | Recommended Version
------------ | --------------- | -------------------
S/4HANA on Premise | OP1909 + Note 2873666 |
S/4HANA Cloud | CE2002 | 
Netweaver < 7.52 (ECC, BW, SRM, ...) | Add-On DMIS 2011 SP17 + Note 2857333 or 2857334 | Add-On DMIS 2011 SP19
Netweaver >= 7.52 (ECC, BW, SRM, ...) | Add-On DMIS 2018 SP02 + Note 2845347 | Add-On DMIS 2018 SP04
> **Note:**
> Please always also consult the most up-to-date Availability Matrix.

<br>ABAP Operators are created in the ABAP System by implementing the BAdI: `BADI_DHAPE_ENGINE_OPERATOR`.<br>
The BAdI implementation consists of a class with **two methods** that must be redefined. It is recommended that the BAdI implementation extends the abstract class
`cl_dhape_graph_oper_abstract`.
- GET_INFO: Returns metadata about the operator
- NEW_PROCESS (with its local class `lcl_process`): Creates a new instance of the operator.

`lcl_process` uses a simple event-based model and can be implemented by redefining one or more of the following methods:
- ON_START: Called once, before the graph is started
- ON_RESUME: Called at least once, before the graph is started or resumed
- STEP: Called frequently (loop)
- ON_SUSPEND: Called at least once, after the graph is stopped or suspended
- ON_STOP: Called once, after the graph is stopped

## Exercise 2.1 - Create your own custom ABAP Operator in SAP S/4HANA

Like the CDS Views, custom ABAP Operators could also be manually implemented in S/4HANA (Class Builder) or in the ABAP Development Tools (ADT) on Eclipse.<br>
However, in order to reduce manual activities to a minimum, there is a framework available that supports you in the creation of all artifacts in the S/4HANA ABAP backend that are required for your own ABAP operator. That framework consists of two reports that must be executed in sequence:
- `DHAPE_CREATE_OPERATOR_CLASS`: Generate an implementation class
- `DHAPE_CREATE_OPER_BADI_IMPL`: Create and configure a BAdI implementation

After running these two reports, the generated implementation class can be adapted as needed.<br>
Here is a step-by-step guideline for creating a custom ABAP Operator. In the specific use case below, the ABAP Operator in S/4HANA should receive a string from a Pipeline ABAP Operator in Data Intelligence, reverses the string, and sends it back to the Pipeline ABAP Operator in Data Intelligence.

1. Logon to the SAP GUI of your conneted S/4HANA system and run transaction `SE38` (ABAP Editor), enter `DHAPE_CREATE_OPERATOR_CLASS` and ***Execute*** (![](/exercises/dd2/images/Execute.JPG) or ***F8***) this report.<br><br>
![](/exercises/dd2/images/dd2-002a.JPG)<br>

2. Enter the required parameters and ***Execute***.<br><br>
![](/exercises/dd2/images/dd2-003a.JPG)<br>

3. Now assign a package or choose 'Local Object', then ***Save*** (![](/exercises/dd2/images/Save.JPG)).<br><br>
![](/exercises/dd2/images/dd2-004a.JPG)<br>

4. You should now see the following screen. Close that windows by clicking ***Exit*** (or ***Shift+F3***).<br><br>
![](/exercises/dd2/images/dd2-005a.JPG)<br>

5. Go back to transaction `SE38` (ABAP Editor). This time, enter `DHAPE_CREATE_OPER_BADI_IMPL` and ***Execute*** (![](/exercises/dd2/images/Execute.JPG) or ***F8***) this report.<br><br>
![](/exercises/dd2/images/dd2-006a.JPG)<br>

6. Enter the required parameters and ***Execute***.<br><br>
![](/exercises/dd2/images/dd2-007a.JPG)<br>

7. Now assign a package or choose 'Local Object', then ***Save*** (![](/exercises/dd2/images/Save.JPG)).<br><br>
![](/exercises/dd2/images/dd2-008a.JPG)<br>

8. On the next screen (Enhancement Implementation), click on ***Implementing Class*** on the left side, then double click on the name of your Implementing Class, in this case `ZCL_DHAPE_OPER_REV_STR_TA99`.<br><br>
![](/exercises/dd2/images/dd2-009b.JPG)<br>

9. This opens the Class Builder (`SE24`). Double click on the `GET_INFO` method in oder to assign the input and output ports of the ABAP Operator. Parameters are not needed in our use case.<br><br>
![](/exercises/dd2/images/dd2-010b.JPG)<br>

10. In the method `GET_INFO`, outcomment the three lines which specify the parameters (parameters are not needed in this scenario). The rest can be left as is. Click the ***Save*** button.<br><br>
![](/exercises/dd2/images/dd2-011b.JPG)<br>

11. Go back and double click on the `NEW_PROCESS` method in order to implement the wanted functionality for our ABAP Operator.<br><br>
![](/exercises/dd2/images/dd2-012b.JPG)<br>

12. On the next screen, double click on the local class `lcl_process`.<br><br>
![](/exercises/dd2/images/dd2-013b.JPG)<br>

13. As we are going to implement a new method `on_data`, we have to declare the method in the class definition.<br>
Add the following code snippet:<br>
```abap
  PRIVATE SECTION.
  METHODS: on_data.
```
![](/exercises/dd2/images/dd2-014b.JPG)<br>

14. We can outcomment the parameter value retrieval (see line 30 in screenshot below).<br>
Then overwrite the existing `step( )` method with the following code:<br>
```abap
  METHOD if_dhape_graph_process~step.
    rv_progress = abap_false.
    CHECK mv_alive = abap_true.

    IF mo_out->is_connected( ) = abap_false.
      IF mo_in->is_connected( ).
        mo_in->disconnect( ).
      ENDIF.
      rv_progress = abap_true.
      mv_alive = abap_false.
    ELSE.
      IF mo_in->has_data( ).
        CHECK mo_out->is_blocked( ) <> abap_true.
        rv_progress = abap_true.
        on_data( ).
      ELSEIF mo_in->is_closed( ).
        mo_out->close( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ELSEIF mo_in->is_connected( ) = abap_false.
        mo_out->disconnect( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ENDIF.
    ENDIF.
  ENDMETHOD.
```
<br> If `has_data( )` returns true, i.e. if the ABAP Operator receives a signal from the corresponding Data Intelligence Pipeline operator, we call the `on_data( )` method, which contains the wanted functionality (reverse an incoming string and send it back). Include the following lines after the `step( )` method:
```abap
  METHOD on_data.
    DATA lv_data TYPE string.
    mo_in->read_copy( IMPORTING ea_data = lv_data ).

    lv_data = reverse( lv_data ).

    mo_out->write_copy( lv_data ).
  ENDMETHOD.
```
<br> Now click the ***Save*** button.<br><br>
![](/exercises/dd2/images/dd2-014c.JPG)<br><br>

The complete code of the local class `lcl_process` should now look as follows:

```abap
CLASS lcl_process DEFINITION INHERITING FROM cl_dhape_graph_proc_abstract.

  PUBLIC SECTION.
    METHODS: if_dhape_graph_process~on_start  REDEFINITION.
    METHODS: if_dhape_graph_process~on_resume REDEFINITION.
    METHODS: if_dhape_graph_process~step      REDEFINITION.

  PRIVATE SECTION.
  METHODS: on_data.

    DATA:
      mo_util         TYPE REF TO cl_dhape_util_factory,
      mo_in           TYPE REF TO if_dhape_graph_channel_reader,
      mo_out          TYPE REF TO if_dhape_graph_channel_writer,
      mv_myparameter  TYPE string.

ENDCLASS.

CLASS lcl_process IMPLEMENTATION.

  METHOD if_dhape_graph_process~on_start.
    "This method is called when the graph is submitted.
    "Note that you can only check things here but you cannot initialize variables.
  ENDMETHOD.

  METHOD if_dhape_graph_process~on_resume.
    "This method is called before the graph is started.

    "Read parameters from the config here
    "mv_myparameter = to_upper( if_dhape_graph_process~get_conf_value( '/Config/myparameter' ) ).

    "Do initialization here.
    mo_util     = cl_dhape_util_factory=>new( ).
    mo_in       = get_port( 'in' )->get_reader( ).
    mo_out      = get_port( 'out' )->get_writer( ).
  ENDMETHOD.

  METHOD if_dhape_graph_process~step.
    rv_progress = abap_false.
    CHECK mv_alive = abap_true.

    IF mo_out->is_connected( ) = abap_false.
      IF mo_in->is_connected( ).
        mo_in->disconnect( ).
      ENDIF.
      rv_progress = abap_true.
      mv_alive = abap_false.
    ELSE.
      IF mo_in->has_data( ).
        CHECK mo_out->is_blocked( ) <> abap_true.
        rv_progress = abap_true.
        on_data( ).
      ELSEIF mo_in->is_closed( ).
        mo_out->close( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ELSEIF mo_in->is_connected( ) = abap_false.
        mo_out->disconnect( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ENDIF.
    ENDIF.
  ENDMETHOD.

  METHOD on_data.
    DATA lv_data TYPE string.
    mo_in->read_copy( IMPORTING ea_data = lv_data ).

    lv_data = reverse( lv_data ).

    mo_out->write_copy( lv_data ).
  ENDMETHOD.

ENDCLASS.
```
<br>***Save*** the local class and activate (![](images/Activate.JPG)) your ABAP Operator implementations.<br><br>

15. When you clicked the Activation button, you are prompted for a selection of objects. Check both and confirm (![](images/Confirm_black.JPG)).<br><br>
![](/exercises/dd2/images/dd2-015b.JPG)<br><br>

The ABAP Operator implementation is now finished. The operator can immediately be used in SAP Data Intelligence Pipeline. The next section of this Deep Dive demo describes how this is done.<br><br>

## Exercise 2.2 - Integrate the custom ABAP Operator in a SAP Data Intelligence Pipeline

SAP Data Intelligence provides multiple Operator shells for the integration with ABAP Operators in SAP S/4HANA. On the one hand, there are the Operator shells that point to pre-defined ABAP Operators in ABAP systems, such as ABAP CDS Reader, ODP Reader, SLT Connector, Cluster Table Splitter (for Business Suite systems), or the ABAP Converter. On the other hand, you can also trigger custom ABAP Operators by using the "Custom ABAP Operator" shell.<br>
Technically, the approaches for calling function modules in S/4HANA are the same in both cases. The only difference is the namespace under which these ABAP Operators are maintained and selected. While the pre-built Operators belong to the namespace `com.sap.`, custom ABAP Operators are assigned to `customer`.

The integration of ABAP Operators is done via Pipelines in the SAP Data Inteligence Modeler.

1.	Logon to SAP Data Intelligence to access the Launchpad application and click on the ***Modeler*** tile.<br><br>
![](/exercises/dd2/images/dd2-016b.JPG)<br><br>

2.	In the DI Modeler, make sure you are in the ***Graphs*** tab (see left side) and click the ***+*** symbol in order to create a new Pipeline.<br><br>
![](/exercises/dd2/images/dd2-017b.JPG)<br><br>

3.	A new Pipeline canvas opens and the design focus automatically changes to the ***Operators*** tab (see left side). Drag the ***Custom ABAP Operator*** icon from the Operator list and drop it onto the canvas. Then do one click on the ***Custom ABAP Operator*** node in the canvas and open the configuration panel by clicking on the related symbol.<br><br>
![](/exercises/dd2/images/dd2-018b.JPG)<br><br>

4.	In the configuration panel on the right side, select the ***ABAP Connection*** (RFC or Websocket RFC connection) to the SAP S/4HANA system that provides the ABAP Operator. If done, click on the selection button of the field for the ***ABAP Operator***.<br><br>
![](/exercises/dd2/images/dd2-019b.JPG)<br><br>

5.	From the pop-up window, select the custom ABAP Operator that you want to call from the Pipeline. In our case, it's the ABAP Operator that receives a string, reverses it, and sends it back to the client. Hence, choose ***Operator Class: String Reversion*** and click ***OK***<br><br>
![](/exercises/dd2/images/dd2-020b.JPG)<br><br>

6.	As you can see, the ABAP Operator node in the Pipeline canvas gets automatically updated with the operator's name in S/4HANA and the ports that we have defined in the previous section of this Deep Dive demo. (The `GET_INFO( )`method in our operator's ABAP class provides the corresponding meta information.)<br><br>
![](/exercises/dd2/images/dd2-021b.JPG)<br><br>

7.	For verifying the functionality of the ABAP Operator call, we'll be using a ***Terminal*** Operator in the Pipeline. This operator allows the sending of user inputs and the reception of the results. Drag the ***Terminal*** icon from the Operator list and drop it onto the Pipeline canvas. Then connect<br>
- the output port of the ABAP Operator with the input port of the Terminal Operator and
- the output port of the Terminal Operator with the input port of the ABAP Operator.
Then ***Save*** the Pipeline.<br><br>
![](/exercises/dd2/images/dd2-022b.JPG)<br><br>

8.	For saving the Pipeline, you are prompted for the name of the pipeline (including namespace information), a description, and the category under which the Pipeline can be found in the ***Graphs*** tab of the Modeler. Fill in the needed and click ***OK***.<br><br>
![](/exercises/dd2/images/dd2-023b.JPG)<br><br>

9.	The Pipeline now gets validated by SAP Data Intelligence. You can see the results in the ***Validation*** tab of the status section in the Modeler UI. If okay, you can now start the Pipeline by clicking on the ***Play*** symbol in the menue bar.<br><br>
![](/exercises/dd2/images/dd2-025b.JPG)<br><br>

10.	Change back to the ***Status*** tab of the status section in the Modeler UI. Once the status has turned to ***running***, click one time on the ***Terminal*** node and open the Terminal UI with a click on the corresponding icon.<br><br>
![](/exercises/dd2/images/dd2-026b.JPG)<br><br>

11.	In the lower section of the Terminal UI, you can now enter the inputs (after the SDH prompt) that shall be send to the ABAP Operator in S/4HANA. After each input press ***Return***. As a resonse from our custom ABAP Operator, you should now receive the reversed strings in the upper section of the Terminal UI, what proves the functionality and integration success.<br><br>
![](/exercises/dd2/images/dd2-027b.JPG)<br><br>

11.	Don't forget to stop the Pipeline again if you haven't embedded a ***Graph Terminator*** before.<br><br>
![](/exercises/dd2/images/dd2-028b.JPG)<br><br>

We've now successfully implemented a custom ABAP Operator in S/4HANA and consumed it from a Data Intelligence Pipeline.
<br><br>

## Exercise 2.3 - Making use of custom ABAP Operators in SAP Data Intelligence for ABAP report execution

So far, we have missed to proof whether or not our ABAP CDS Views and the consuming Data Intelligence Pipelines are really providing delta information. This would require access to the S/4HANA system in order to conduct changes on the data basis, hence on the EPM Business Partner table or on the EPM Sales Order object.

The Enterprise Procurement Model (EPM) demo application comes with a report that allows you to generate EPM Sales Order data. This report **`SEPM_DG_EPM_STD_CHANNEL`** can be started with the transaction **`SEPM_DG`** in S/4HANA.<br><br>
Here is an **example** of how the Report looks like in the SAP GUI:<br><br>
![](/exercises/ex2/images/ex2-000b.JPG)<br><br>

As another option to trigger changes on EPM data in our S/4HANA system, we have created a variant of the above report **`SEPM_DG_EPM_STD_CHANNEL`** and a Custom ABAP Operator **`customer.teched.socreate`** which executes this report variant.<br><br>
**FYI**, the following lines in the Local Class `lcl_process`, which instantiated by the `NEW_PROCES( )` method of the Operator Class `ZCL_DHAPE_OPER_CREATE_SO`, implements this functionality:

```abap
  METHOD on_data.
    DATA lv_data TYPE string.
    mo_in->read_copy( IMPORTING ea_data = lv_data ).

    SUBMIT SEPM_DG_EPM_STD_CHANNEL USING SELECTION-SET 'SEPM_TECHED_SO' AND RETURN.
    lv_data = ' --> One additional EPM Sales Order with five related Sales Order Items created.'.

    mo_out->write_copy( lv_data ).
  ENDMETHOD.
```
<br>
You are asked to leverage this Custom ABAP Operator in this exercise with the goals to
- experience how the execution of ABAP Function Modules in a remote S/4HANA system can be triggered from SAP Data Intelligence Pipeline and
- finally have an approach to change data on the Enterprise Procurement Model in S/4HANA in order to verify the delta capabilities of the ABAP CDS Views and your Pipeline implementations.<br><br>

>**Important Note**<br>
>Please consider that we are working on the same central data basis (the EPM tables in S/4HANA) which is used by all workshop participants. Hence, Sales Order records that you create by using the Custom ABAP Operator will become change data for all workshop participants.

After completing these steps you will have created a Pipeline that triggers the execution of a report variant in the connected S/4HANA for EPM data generation.

1. Log on to SAP Data Intelligence and enter the Launchpad application. Then start the ***Modeler*** application.
   - Follow the link to your assigned Data Intelligence instance, e.g. https://vsystem.ingress.xyz.dhaas-live.shoot.live.k8s-hana.ondemand.com/app/datahub-app-launchpad/.
   - The tenant name is **"workshop"**.
   - In the next pop-up window, enter your assigned user name (e.g. ***"TA99"***) and your individual password received from the DI user registration.<br><br>
   ![](/exercises/ex2/images/ex1-002c.JPG)<br><br>
   - From the Launchpad, start the ***Modeler*** application by clicking on the corresponding tile.<br><br>
   ![](/exercises/ex2/images/ex2-003b.JPG)<br><br>

2. Make sure you are in the ***Graphs*** tab of the Modeler UI (see left side). Then click the ***+*** symbol in order to create a new Pipeline.<br><br>
![](/exercises/ex2/images/ex2-004b.JPG)<br><br>

3. Now a new Pipeline canvas is opened on the right side and the Modeler UI automatically switches to the ***Operators*** tab (see left side). In the list of operators, drag the ***Custom ABAP Operator*** and drop it into the Pipeline canvas. Click the ABAP CDS Reader node in the canvas one time and then click the ***configuration*** icon.<br><br>
![](/exercises/ex2/images/ex2-005b.JPG)<br><br>

4. In the configuration panel for the ***Custom ABAP Operator***, specify the S/4HANA Connection: Select `S4_RFC_TechEd`. Then click on the ***ABAP Operator*** selection icon.<br><br>
![](/exercises/ex2/images/ex2-006b.JPG)<br><br>

5. In the pop-up windows, select the **"Creation of EPM Sales Order"** option. Then click ***OK***.<br><br>
![](/exercises/ex2/images/ex2-007b.JPG)<br><br>

7. From the list of Operators, drag the ***Terminal*** operator into the Pipeline canvas. Connect **the output port of the Custom ABAP Operator with the input port of the Terminal operator**. Then connect the **output port of the Terminal operator with the input port of the Custom ABAP Operator**.<br>
With this Pipeline setup, we can - from the Terminal UI - trigger the ABAP Operator call and the reception of confirmation message at the same time.<br><br>
![](/exercises/ex2/images/ex2-008b.JPG)<br><br>

8. ***Save*** the Pipeline and enter the following parameters prompted in the pop-up windows:<br>
   - Name: `teched.XXXX.EPM_FM_Call_SO_Generator`, where XXXX is your user name, for example "teched.TA99.EPM_FM_Call_SO_Generator"
   - Description: `XXXX - Generate EPM SO data via ABAP FM call`, where XXXX is your user name, for example "TA99 - Generate EPM SO data via ABAP FM call"
   - Category: `dat262'.
![](/exercises/ex2/images/ex2-009b.JPG)<br><br>

9.	***Run*** your Pipeline. After the Pipeline status has changed from ***pending*** to ***running***, right-click on the ***Terminal*** operator and click on ***Open UI***.<br><br>
![](/exercises/ex2/images/ex2-010b.JPG)<br><br>

10.	In the Terminal UI, put the ***cursor to the prompt line ('SDH >')*** on the lower section of the UI and press ***RETURN***. In the upper part of the UI, you can then see the confirmation from the S/4HANA ABAP Operator than one Sales Order Header record and five related Sales Order Items have been generated.<br><br>
![](/exercises/ex2/images/ex2-011b.JPG)<br><br>

***Congratilations***, you have successfully implemented a Data Intelligence Pipeline that triggers the execution of a function module in S/4HANA and allows direct interaction!
In the next section, you can leverage this remote data generation option to verify the delta capabilities of our previously built Pipelines for ABAP CDS View based data replication.<br><br>


## Exercise 2.4 - Triggering a custom ABAP Operator to verify your Delta Replication of EPM Sales Orders

You can now test the delta processing capabilities of the ABAP CDS View based data extraction. A nice task would be to check if Pipeline for Sales Orders replications and enrichment that you have built in [Exercise 1.8 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3](../ex1#exercise-13---implement-a-pipeline-for-delta-transfer-of-enhanced-epm-sales-order-data-from-s4hana-to-an-s3-object-store) is really processing the delta records from EPM in S/4HANA.<br><br>

1. For so doing, change to the ***Graphs*** tab of the Modeler UI (see left side). Then enter your user name in the search field (if you made your user name a part of the Pipeline names or descriptions) and start the search. You will now get a list of the Pipelines that you have implemented. Click on your 'Sales Order Replication and Enrichment' Pipeline icon. If the displayed name is too short to recognize a unique name, just hover with your mouse over the Pipeline icons (see below).<br><br>
![](/exercises/ex2/images/ex2-012b.JPG)<br><br>

2. The Pipeline is opened in the canvas area of the Modeler UI. ***Run*** it.<br><br>
![](/exercises/ex2/images/ex2-013b.JPG)<br><br>

3. If you see that both of your Pipelines (data generating as well as data replication and enrichment) are in a ***running*** status, open the Wiretap UI.<br><br>
![](/exercises/ex2/images/ex2-014b.JPG)<br><br>

4. You can see now that the initial load from the ABAP CDS View in S/4HANA was successfully conducted. Please **leave the Wiretap UI open**.<br><br>
![](/exercises/ex2/images/ex2-015b.JPG)<br><br>

5. On the tab bar of the canvas area, switch to the Data Generation Pipeline and open the Terminal UI.<br><br>
![](/exercises/ex2/images/ex2-016b.JPG)<br><br>

6. Put the cursor on the prompt of the lower part (input part) of the Terminal UI and press ***RETURN***. You will see the confirmation from the ABAP Operator about the records created in the EPM demo app.<br><br>
![](/exercises/ex2/images/ex2-017b.JPG)<br><br>

7. In the Browser switch back to the ***Wiretap UI*** of your Replication and Enrichment Pipeline. You should now see that a message with new records came in, which all have the "U" indicator for Updates. This verifies the delta-enablement of the ABAP CDS View and the immediate integration with Data Intelligence.<br><br>
![](/exercises/ex2/images/ex2-018b.JPG)<br><br>

8. Now please throw one last inspecting glance at the files on S3. Here, you can again use the data browser feature in the Data Intelligence ***Metadata Explorer***. Open the application via the Launchpad.<br><br>
![](/exercises/ex2/images/ex2-019b.JPG)<br><br>

9. In the Metadata Explorer main screen, click on ***Browse Connections***.<br><br>
![](/exercises/ex2/images/ex2-020b.JPG)<br><br>

10. Click on the Connection **"TechEd2020_S3"** and drill further down to the folders **"DAT262"** and that of **your user** (e.g. "TA99").
![](/exercises/ex2/images/ex2-021b.JPG)<br><br>
![](/exercises/ex2/images/ex2-022b.JPG)<br><br>

11. Click on the ***More Actions*** menue (the three dots) of the file ***Enriched_Sales_Orders*** - that contains the delta records - and click on ***View Fact Sheet***.<br> (Remark: this file get overwritten with each data update as S3 does not provide *append* capabilities. If you would like to preserve the complete history, you can still add a *File Reader* and then a *File Writer* operator the to Pipeline behind the *Structured File Writer*. The latter does appends in memory and writes back the completed full recordset to S3.)<br><br>
![](/exercises/ex2/images/ex2-023b.JPG)<br><br>

12. You can now see that also the processing of the delta data within the Pipeline to S3 worked seamlessly. The delta records contain the enrichments and the "U" indicator that with which updates are flagged by the ABAP CDS View CDC framework.<br><br>
![](/exercises/ex2/images/ex2-024b.JPG)<br><br>

**Very well done!** You successfully followed the integration approach for data processing and functional execution between S/4HANA and SAP Data Intelligence and know how to realize such implementations.



## Summary

In the  Exercises, we have worked on the implementation of delta-enabled data sources and remote functionality in S/4HANA and have leveraged these features directly in SAP Data Intelligence. We could now extend these use cases for more complex scenarios. The general implementation approaches and the support that the Data Intelligence applications provide would then still be the same.<br>

In case you have asked yourself if there are similar options for the integration with other ABAP systems such as ECC or BW, the answer is 'yes'! In these cases, you can make use of the unmodifying DMIS add-on, which provides the ABAP Pipeline Engine also for these systems.<br>

This given, you can realize real-time replication scenarios via SLT integration to SAP Data Intelligence, leverage the ODP Reader operators, and also trigger function module execution on ECC or BW systems, too.<br>

More information can be found [here](https://blogs.sap.com/2019/10/29/abap-integration-for-sap-data-hub-and-sap-data-intelligence-overview-blog/).<br><br>

**THANK YOU VERY MUCH** for having participated in this Hands On workshop. We hope you have enjoyed it!

Bengt Mertens and Tobias Karpstein<br><br>

**************************************************

**Related TechEd2020 Lectures and Workshops**

- [Introduction to SAP Data Intelligence [DAT111]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750108001uev5)
- [Road Map: SAP Data Intelligence [DAT829]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555751546001uZ7S)
- [Integrating SAP S/4HANA into SAP Data Intelligence [DAT206]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555751031001uFT1)
- [Integrating SAP S/4HANA into SAP Data Intelligence: Overview and Use Cases [DAT203]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750868001ucdH)
- [Create and Manage End-to-End Data Pipelines with SAP Data Intelligence [DAT263]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555751331001uH22)
- [How SAP Data Intelligence Supports Your Data Governance Projects [DAT112]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750159001uApr)
- [A Data Governance Journey with SAP Data Intelligence [DAT163]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750580001uaPx)
- [Create Data Flows Covering Both SAP and Non-SAP Using SAP Data Intelligence [DAT164]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750632001uCze)
- [Upscaling to Hybrid: EIM and SAP Data Intelligence [DAT205]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750978001u6PR)
- [SAP Data Intelligence: Extensibility & Business Content [DAT942]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555751700001uDa1)
- [Hybrid Data Management SAP Data Intelligence Cloud [DAT110]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555750055001ucgf)
- [Expert Q&A on SAP Data Intelligence [DAT943]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1602555751752001uc3V)
- [SAP Data Intelligence as Part of SAP HANA Cloud [ST102]](https://events.sapteched.com/widget/sap/sapteched2020/Catalog/session/1603382154918001u9LA)

<br><br>
