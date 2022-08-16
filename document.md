# Arachne-UiPath Documentation

## Index

 - [Main Diagram](#main-diagram)
 - [Process Orchestration](#process-orchestration)
 - [Process Centralization](#process-centralization)
   - [Client Process](#client-process)
     - [How to Organize Dependencies](#how-to-organize-dependencies) 
     - [How to Organize Project Directories](#how-to-organize-project-directories)
     - [Argument : Queue Item ID](#argument--queue-item-id)
   - [Client Suit Process](#client-suit-process)
     - [How to Organize Dependencies](#how-to-organize-dependencies-1) 
     - [How to Organize Project Directories](#how-to-organize-project-directories-1)
     - [Argument : Process ID](#argument--process-id)
     - [Argument : Queue Item ID](#argument--queue-item-id-1)
     - [Argument : Update Flag](#argument--update-flag)
     - [Argument : State Machine Logicalization : Request](#argument--state-machine-logicalization--request)
     - [State Machine Logicalization : Strategy](#state-machine-logicalization--strategy)
   - [Invoking Workflow](#invoking-workflow)
   - [Package Directory](#package-directory)

---
<br>

## Main Diagram

<br>
<img src="https://github.com/hyundai-capital-rpa/arachne-uipath-document/blob/main/reference/main.png"> 
<br>

---
<br>

## Process Orchestration

&nbsp; &nbsp; &nbsp; &nbsp; A task &mdash; which is automated by a robot &mdash; generally means that a business flow what a business owner does, and a developer implements based on the business flow. but the thing is that the business flow &mdash; the developer starts to implement &mdash; is what the business owner told in his/her way and it might be optimized for human being working, but not for a robot, especially for the distributed processing. 

&nbsp; &nbsp; &nbsp; &nbsp; Before describe of how we defined a task in the distributed processing, we have to understand the way of starting a process with Arachne-UiPath package and the reason for the difference. A process &mdash; which is implemented on Arachne-UiPath environment &mdash; should not be designed or implemented to be executed by itself not like generally a developer implements with UiPath, and needs an evidence (a `queue item`) for execution in the queue each process is refering on the Orchestrator. As Arachne-UiPath package follows the way of multi-processing within machine pool, the main process retrieves a queue item which should be started, and executes a process related with the retrieved queue item. This is how Arachne-UiPath package maintains the process orchestration, the chain execution, and fast responding to the request. 

<br>
<img src="https://github.com/hyundai-capital-rpa/arachne-uipath-document/blob/main/reference/chain_of_processes.png"> 
<br>

&nbsp; &nbsp; &nbsp; &nbsp; Then what is the chain execution? General execution way of UiPath is to execute a process with the Orchestrator trigger on a machine where UiPath Robot Service installed, and it has the limitation of responding dynamically and the limitation of flexibility of schedules. For overcoming the limitations like these, it needs the main process called `Chain Runner` which is already introduced above. Chain Runner executes a process as the Orchestrator trigger, but dynamically, not statically with triggers, then it will yield its execution (termination). After finishing of the process, Chain Runner will be executed again for executing the next process and this is the way of how we can overcome the limitations and it is called as the chain execution.

&nbsp; &nbsp; &nbsp; &nbsp; However, for maintaing the chain execution, the evidence queue item of the current process &mdash; except 1st scheduled process &mdash; should be put dynamically by the previous process. So we redefined a task as a set of processes and process types. A task &mdash; which is automated by a robot in the distributed processing &mdash; basically is consisted of 3 types of processes for standardized business flow. Please make sure that it is not necessary, but we recommend strongly when a developer makes up a project using Arachne-UiPath package. The details of 3 types of processes are described below.

<br>
<img src="https://github.com/hyundai-capital-rpa/arachne-uipath-document/blob/main/reference/flow_of_chain_execution.png"> 
<br>

- **Task** is only a logical unit for identifying the business owner's business flow and a set of processes. 

  - **Dispatch Process** takes the role of collecting the records to processing. The process retrieves any predefined input data provided from the business owner, then reconstructs the data into queue item (hereinafter referred to as `perform queue item`) which is smallest unit in business-logically and put these on the Orchestrator queue which the perform process refers. Please aware of that each perform queue item will be treated as an evidence of each perform process. And also put an evidence (hereinafter referred to as `after performance queue item`) on the Orchestrator queue which the after performance process refers for collecting the result of each perform queue items. 

  - **Perform Process** is to perform each queue item created from the previous distpach process. The process should not perform or effect other queue items except the evidence of the current execution and the result of the perform queue item should be put on the isolated Orchestrator queue &mdash; which only contains only results &mdash; to preserve the evidence and the result as itself for error tracking later on.

  - **After Performance Process** takes the role of collecting the result records which are created by the perform process. The after performance queue item will be retrieved only if the queue does not contain queue items which of status are "New" or "InProgress" anymore. This is how the main process preserves the order of the task during the distributed processing. 

&nbsp; &nbsp; &nbsp; &nbsp; Yes, it is natural if we came up with [UiPath Robotic Enterprise Framework](https://docs.uipath.com/studio/docs/robotic-enterprise-framework) after reading the details of 3 types of processes. The Arachne-UiPath package is mainly designed to overcome the limitations of UiPath Robotic Enterprise Framework and further enhance the flexibility of the process orchestration.

---
<br>

## Process Centralization

&nbsp; &nbsp; &nbsp; &nbsp; As experiencing large scale of opration over 3 years, we found out that a process needs to be handled in centrialized management, and it should be started from the developer's design time. This is why we made the process called `Client Suit Process` which takes a part of framework on Arachne-UiPath package. Below is what we focused for designing the Client Suit Process.

- Centralize control
- Be flexible from process publish
- Be less influential to process orchestration
- Recycle code as much as possible
- Provide minimal information to understand structure

<br>
<img src="https://github.com/hyundai-capital-rpa/arachne-uipath-document/blob/main/reference/business_flow.png"> 
<br>

&nbsp; &nbsp; &nbsp; &nbsp; The purpose of the Client Suit Process is to standalization of individual developer's implementation & centralization of operation. After calling the process from Chain Runner, the process &mdash; which is also called `Client Process` for distinguishing it from the Client Suit Process &mdash; does not invoke business flows with its control. Not just a concept of code style, there is actual inversion of control on the runtime between the Client Suit Process and itself. Of course, for making it happen, a developer should design a process following our guideline of process centralization. However, by only making inversion of control happen, the process should follow only a standarlized sequence, and is able to be affected updates from the Client Suit Process at the same moment, and also now we are able to hide fatal logic or prevent redesigning during the developer's design time. 

&nbsp; &nbsp; &nbsp; &nbsp; As described above, a developer has no control of a project configuration at the design time if the project follows process centralization. Then how can the developer inject his/her purpose on the project? The Client Suit Process will make you possible to constitute the state machine on the runtime by being requested the argument which XAML files to call, and how the state machine is configured. We call it as the `State Machine Logicalization` and you can see the details of the request format with [this link](#argument--state-machine-logicalization--request). 

<br>

### **Client Process**

- #### How to Organize Dependencies

~~~
project
  - arachne-uipath library
    - process centralization core activities
    - process orchestration core activities
    - process utility core activities
  - libraries for business
~~~

- #### How to Organize Project Directories

  - **Business** container is for business workflow files. these workflow files would be called by the Client Suit Process when the process enters in a business state. 

  - **Closure** container is for closure workflow files. these workflow files would be called by the Client Suit Process when the process enters in a closure state. 

  - **Exception Handle** container is for exception handle workflow files. these workflow files would be called by the Client Suit Process when the process enters in an exception handle state.

  - **Method** container is for any method workflow files. these workflow files are independent from process centralization.

  - **Main.xaml** file is main execution file. and it is a logical callee that requests the Client Suit Process also called as a logical caller. when the runtime entered the logcal caller section, it means its control will be handed to the logical caller.

~~~
project
  - .entities
  - .objects
  - .settings
  - .templates
  - .tmh
  - Business
    - BusinessWorkflowFile01.xaml
    - BusinessWorkflowFile02.xaml
  - Closure
    - ClosureWorkflowFile01.xaml
    - ClosureWorkflowFile02.xaml
  - Exception Handle
    - ExceptionHandleWorkflowFile01.xaml
    - ExceptionHandleWorkflowFile02.xaml
  - Method
    - MethodWorkflowFile01.xaml
    - MethodWorkflowFile02.xaml
  - Main.xaml
  - project.json
~~~

- #### Argument : Queue Item ID
```json
{
    // this is request key to retrieve 
    // queue item to process
    // boolean available
    "in_queueItemID" : 0
    }
```

<br>

### **Client Suit Process**

- #### How to Organize Dependencies

~~~
project
  - arachne-uipath library
    - process centralization core activities
    - process orchestration core activities
    - process utility core activities
~~~

- #### How to Organize Project Directories

  - **Core (*concept*)**, containers under this concept is for containing core workflow files. these workflow files will be locked by the Client Suit Process core library.

    - **Core** container is for core workflow files. a Client Suit Process designer can place any workflow files that don't want to be accessed by client process.

  - **Not Core (*concept*)**, containers under this concept is for containing not core workflow files. these workflow files can be overrided by client process if there are same workflow files in client process project. so that means the main purpose of these containers provides to call workflow files from client process.

    - **Business** container is for business workflow files. a Client Suit Process designer can place any workflow files to be called as default workflow files. these workflow files would be called by Client Suit Process when the process enters in a business state. 

    - **Closure** container is for closure workflow files. a Client Suit Process designer can place any workflow files to be called as default workflow files. these workflow files would be called by the Client Suit Process when the process enters in a closure state. 

    - **Exception** Handle container is for exception handle workflow files. a Client Suit Process designer can place any workflow files to be called as default workflow files. these workflow files would be called by the Client Suit Process when the process enters in an exception handle state.

    - **Method** container is for any method workflow files. these workflow files are independent from process centralization.

  - **Main.xaml** file is main execution file. and it is a logical caller that calls workflow files or organizes the state machine logically from the client process's container.

~~~
project
  - .entities
  - .objects
  - .settings
  - .templates
  - .tmh
  - Core
    - CoreWorkflowFile01.xaml (guid converted)
    - CoreWorkflowFile02.xaml (guid converted)
  - Business
    - BusinessWorkflowFile01.xaml
    - BusinessWorkflowFile02.xaml
  - Closure
    - ClosureWorkflowFile01.xaml
    - ClosureWorkflowFile02.xaml
  - Exception Handle
    - ExceptionHandleWorkflowFile01.xaml
    - ExceptionHandleWorkflowFile02.xaml
  - Method
    - MethodWorkflowFile01.xaml
    - MethodWorkflowFile02.xaml
  - Main.xaml
  - project.json
~~~

- #### Argument : Process ID
```json
{
    // this is request key to retrieve 
    // process id to process
    // string available
    "in_processID" : ""
    }
```

- #### Argument : Queue Item ID
```json
{
    // this is request key to retrieve 
    // queue item to process
    // integer available
    "in_queueItemID" : 0
    }
```

- #### Argument : Update Flag
```json
{
    // this is request key to update 
    // client suit process or not
    // boolean available
    "in_updateFLAG" : true
    }
```

- #### Argument : State Machine Logicalization : Request 
```json
{
    // this is request key to describe
    // how state machine is organized
    // json available
    "in_requestSTR" : {

        // this is request key to decide 
        // strategy of state machine limit
        // any string available
        "Strategy" : "Default",

        // this is request key to decide
        // state machine organization 
        // json available
        // key : state role
        // value : state organization
        "State Machine" : {

            // what states do in business role
            // sequence and retrial formats would be as below
            "Business" : {

                // describe how states are organized logically
                "Sequence" : [

                    // state 01
                    // workflow file paths to call
                    [
                        "relative workflow file path 11", 
                        "relative workflow file path 12",
                    ],
                    
                    // state 02
                    // workflow file paths to call
                    [
                        "relative workflow file path 21",
                        "relative workflow file path 22",
                    ]

                    // and so on ...
                    ],

                // describe max retrial counts with each state    
                "Retrial" : [
                    
                    // state 01
                    // max retrial count
                    0, 

                    // state 02
                    // max retrial count
                    1

                    // and so on ...
                    ]
                },

            // what states do in closure role
            // sequence and retrial are the same as business role ...
            "Closure" : {
                "Sequence" : [[]],
                "Retrial" : []
                },

            // what states do in exception handle role
            // sequence and retrial are the same as business role ...
            "Exception Handle" : {
                "Sequence" : [[]],
                "Retrial" : []
                }
            }
        }
    }
```

- #### State Machine Logicalization : Strategy
```json
{
    // this is request key to select strategy
    // json available
    "Default" : {

        // this is request key to decide
        // state machine strategy organization 
        // json available
        // key : state role
        // value : state strategy organization
        "State Machine" : {

            // what state strategy does in business role
            // sequence and retrial and default retrial formats 
            // would be as below
            "Business" : {

                // index 0 : state minimum count
                // index 1 : state maximum count
                // index 2 : workflow file in a state minimum count
                // index 3 : workflow file in a state maximum count
                "Sequence" : [0, 0, 0, 0],

                // index 0 : state retrial minimum count
                // index 1 : state retrial maximum count
                "Retrial" : [0,0],

                // state default retrial count
                // when request format does not mention a workflow file
                // to override default workflow file  
                "Default Retrial" : 0
                },

            // what state strategy does in closure role
            // sequence and retrial and default retrial formats
            // are the same as business role ...
            "Closure" : {
                "Sequence" : [],
                "Retrial" : [],
                "Default Retrial" : 0
                },

            // what state strategy does in exception handle role
            // sequence and retrial and default retrial formats
            // are the same as business role ...
            "Exception Handle" : {
                "Sequence" : [],
                "Retrial" : [],
                "Default Retrial" : 0
                },
            }
        }
    }
```

<br>

### **Invoking Workflow**

&nbsp; &nbsp; &nbsp; &nbsp; As the project with State Machine Logicalization does not allow a developer to access to the states, and the Client Suit Process invokes workflow files dynamically, the arguments of an individual workflow file should be fixed with following 2 arguments &mdash; `Configuration Dictionary` which of name is `in_configDIC` and which of argument direction is "Input", and the argument `Packing Dictionary` which of name is `io_packDIC` and which of argument direction is "Input/Output". 

- #### Argument : Configuration Dictionary
```vb
in_configDIC = New Dictionary(Of String, Object)
' {
'     "Task Name" : "your task name (project)",
'     "Process Name" : "PRJ001_01_process_name_perform",
'     "Queue Name" : "Queue-PRJ001_01_process_name_perform",
'     "Mapping Queue Name" : "Queue-PRJ001_01_process_name_perform_mapping"
'     "Start Notification" : True,
'     "End Notification" : True,
'     "Start Mail To" : "email addresses (seperated with semicolon)",
'     "Start Mail Cc" : "email addresses (seperated with semicolon)",
'     "End Mail To" : "email addresses (seperated with semicolon)",
'     "End Mail Cc" : "email addresses (seperated with semicolon)",
'     "Exception Mail To" : "email addresses (seperated with semicolon)",
'     "Exception Mail Cc" : "email addresses (seperated with semicolon)",
'     "Is Encryption" : True,
'     "Workspace Root" : "directory (generated dynamically)",
'     "Workspace Exception Path" : "directory (generated dynamically)",
'     "Workspace Reference Path" : "directory (generated dynamically)",
'     "Workspace Work Path" : "directory (generated dynamically)",
'     "Workspace Input Path" : "directory (generated dynamically)",
'     "Workspace Temp Path" : "directory (generated dynamically)",
'     "Workspace Output Path" : "directory (generated dynamically)"
'     }
```

- #### Argument : Packing Dictionary
```vb
io_packDIC = New Dictionary(Of String, Object)
' {
'     "Process Data" : New System.Data.DataTable(),
'     "Queue Item" : New UiPath.Core.QueueItem(),
'     "Exceptions" : New List(Of Dictionary(String,Object))
'     "What You Loaded #1" : New Object(),
'     "What You Loaded #2" : New Object(),
'     "What You Loaded #N" : New Object(),
'     }
```

&nbsp; &nbsp; &nbsp; &nbsp; Why should Packing Dictionary be fixed during invoking workflow? As described above, the developer cannot modify variables or arguments at the scope of the state machine within the project with State Machine Logicalization. Then how the developer hand variables from the current state to the next state? At this point, Packing Dictionary helps the developer to communicate with each states, but it is different that it happens on the runtime, not the design time. Please take a look how it works at the below flowchart. 

<br>
<img src="https://github.com/hyundai-capital-rpa/arachne-uipath-document/blob/main/reference/invoking_workflow.png"> 
<br>

### **Package Directory**

- How package directory is organized on the runtime

  - **Directory called ".arachne-uipath\\.cp-workspace"** contains every directory which is used by the Client Process.

    - **Directory for workspace** &mdash; which of name is combinated with process id & queue item id dynamically &mdash; contains all files which are created on the runtime. As running a process with different queue items, the reason of combination is to prevent overwrite the storage of other processes because every storage of the process has to be guaranteed its independence. And also it is for preventing a developer not to store at the outside of package directory, so that we can ensure where the files created, or needed for the process on the runtime are.

      - **Directory called "exception"** stores screenshots whenever a process occurred an unexpected exception. The exception from UI Automation may not be traced only with general error trace log because it can be caused by another program like legacy system crash or not rendered UI. Especially, the exception "SelectorNotFoundException" may be a developer's mistake, but it can be occurred by not rendered in time. And it is hard to trace that the selector does not found due to network only with an error trace log.

      - **Directory called "reference"** stores reference files for business flow during the runtime. the reference files are downloaded on this directory from the Orchestrator at the start of the Client Suit Process.

      - **Directory called "work"** stores the files which are mainly used on a process. However, a developer should write or download the files on these subfolders &mdash; input, temp, and output &mdash;, not directly on this directory. Of coures, we cannot force a developer to store files in a specific folder, but these subfolders are seperated for making how the files flow on the runtime a little easier to see.

  - **Directory called ".arachne-uipath\\.csp-workspace"** contains every copied project directory of the Client Suit Process. 

    - **Directory for project** &mdash; which of name is created in guid format &mdash; is the highest verison project of a Client Suit Process which is copied from nuget package directory "%userprofile%\\.nuget\\packages\\{client suit process}\\{version}\\lib\\net45" on the runtime. As the main purpose of implementing the Client Suit Process is for minimizing the intervention of individual developers, it is created in guid format to make a developer hard to find this directory on the design time.
  
  - **Directory called ".arachne-uipath\\.settings"** is storage for other processes which are not under the concept of the Client Process & the Client Suit Process.

~~~
%userprofile%
  - .arachne-uipath
    - .cp-workspace
      - {process id}_{queue item id} (workspace)
        - exception
        - reference
        - work
          - input
          - output
          - temp
    - .csp-workspace
      - {guid} (project)
    - .settings
      - .chrome-extension
~~~

---
<br>
