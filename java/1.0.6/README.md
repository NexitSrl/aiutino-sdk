# AIutino-SDK-Java 1.0.6

Software Development Kit for AIutino integrations

This is an AIutino API client library to allow easy integration of AIutino tools third party applications.

This SDK is written in Java to allow easy integration of the AIutino System with java solutions.

Since version 1.0.3 the library requires at least **Java 8 or higher**

The *AIutino System* makes use of some elements through which it allows you to manage documents. These elements are the **Session**, **Workflows**, **Activities** and **Documents** and are organized in a tree.

    session
       |
       ├─ workflow
       |     ├─ activity 
       |     |     ├─ document 
       |     |     └─ document
       |     ├─ activity 
       |     |     ├─ document 
       |     |     ├─ document
       |     |     └─ document
       └─ workflow
             └─ activity 
                   └─ document 

In short, when you want to process a document with AIutino you need to have an activity connected to the workflow you want to use for the document. By sending the document to the activity, the document is processed using the linked workflow.

## Session

To start using AIutino SKD for java you have to obtain an instance of the **Session** object, that represents a set of operations made over a connection.

    Session session = Session.builder()
                          .userName("myUserName")
                          .password("myPassword")
                          .build();

A session requires at least the user name and the user password to authenticate the client.
A session can be used over multiple operations and doesn't need to be closed.

## Workflows

Once you have created a session you can navigate through the workspaces assigned to the users's organization

    List<Workflow> workflowList = session.getWorklfows();

Alternatively you can obtain a single workflow by calling 

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");

In this case  you have to pass as argument a valid *workflowId* own by the user.

## Activities

The workflow is the main entry point to start actions with *AIutino*.

### Create a new activity

First of all in a workflow it is possible to create new *activities* (collectors of documents that run the same workflow)
To create an activity, simply assign a name to the new entity using the **addNewActivity** method on a workflow object.

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");
    ActivityResponse response = workflow.addActivity("My new activity");

The *ActivityResponse* is an object that contains the *activityId*

    {
        "activityId" : "b729e7b4-08da-49c0-8b34-264912016382"
    }

### List and get the activities in a workflow

To list all the activities in a workflow you can invoke the **getActivities()** method on the workflow.

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");

    Activities workflowActivities = workflow.getActivities();

Alternatively it is possible to invoke the **getActivities(LocalDateTime from, LocalDateTime to)** to have a reduced list of activities. This variant returns only the activities included in the period between the *from* and *to* arguments. 

The result of the *getActivities* methods is an **Activities** object, that returns the activities in a paged way. 
Using the **hasMorePages()** and **nextPage()** you will obtain the list of activities page by page, as in the example below:

    Activities workflowActivities = workflow.getActivities();

    while(workflowActivities.hasMorePages()){
        List<Activity> activitiesList = workflowActivities.nextPage();
        [... do something with the list ...]
    }

If you previously know the *activityId* you can directly obtain an **Activity** object invoking the **getActivity(String id)** method on the workflow.

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");

### Send a document to an activity

To have a document processed by a workflow you can send it to an *activity* linked to the *workflow*.

To send a document you have to *prepare* the document boxing it into an **UploadingDocument** object.

An *UploadingDocument* needs 
- a file name (the simple name of the file like "test.pdf", not the file's path)
- a file content in a byte array
- a document type that is one of the values in the *com.nexitsrl.aiutino.apiclient.DocumentType* enumeration (PDF,TXT,JSON)

If you can load a file named "test.pdf" in a byte array named *fileContent*, you can declare the UploadingDocument in the fluent way using a builder pattern:

    byte[] contentBytes = .... ;  // we assume that you have already loaded this content

    UploadingDocument uploading = UploadingDocument.builder()
                                        .fileName("test.pdf")
                                        .type(DocumentType.PDF)
                                        .content(contentBytes)
                                        .build();

Once you have prepared the UploadingDocument you can invoke the **addDocument** method on the activity to send the file to *AIutino system*

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");

    Document document = activity.uploadDocument(uploading);

If the operation succeeds then you get a **Document** object that represents the document sent to AIutino and contains the *documentId* useful to search the document and obtain the processing results.

The **Document** object is a set of *metadata* that represents the document status. The object itself doesn't contain the source file neither the results produced by the processing workflow.

### List and get documents

To list all the documents that are present within an activity it is possible to invoke the **getDocuments()** method on the activity object.

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");
    Documents activityDocuments = activity.getDocuments();

Alternatively it is possible to invoke the **getDocuments(LocalDateTime from, LocalDateTime to)** to have a reduced list of documents. This variant returns only the documents included in the period between the *from* and *to* arguments. 

As for the activities list, the **Documents** object obtained is a *paged* container that you can navigate in a *Iterator-like* mode.
Using the **hasMorePages()** and **nextPage()** you can obtain the page by page list of Document objects, as in the example below:

    Documents activityDocuments = activity.getDocuments();

    while(activityDocuments.hasMorePages()){
        List<Document> documentsList = activityDocuments.nextPage();
        [... do something with the list ...]
    }

If you previously know the *documentId* you can directly obtain a **Document** object invoking the **getDocument(String id)** method on the *activity* object.

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");
    Document document = activity.getDocument("c7e3d6f4-ad56-42f2-bcd5-f217a24089ee");

### Retriving processing results

As mentioned above, the **Document** object is a set of *metadata* that represents the document status. The object itself doesn't contain the source file neither the results produced by the workflow processing.

Once you have uploaded a Document you can inspect the returned Document object and see the documentId property. Using the *documentId* after some time you can call the **getDocument** method to check the document status.

    {
        "documentId": "c7e3d6f4-ad56-42f2-bcd5-f217a24089ee",
        "activityId" : "b729e7b4-08da-49c0-8b34-264912016382",
        "sourceType" : "PDF",
        "resultType" : "JSON",
        "created" : "2022-12-10T15:37:18.123Z",
        "status" : "RUNNING"
    }

After the document has been uploaded it is possible to retrieve the source document invoking the getSource() method on the document.

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");
    Document document = activity.getDocument("c7e3d6f4-ad56-42f2-bcd5-f217a24089ee");
    byte[] sourceBytes = document.getSource();

Checking the **status** property you can know the document's processing phase.

The life cycle of a document is:
+ **NEW**: A document is just been acquired from the system.
+ **READY**: The document is ready to be processed.
+ **RUNNING**: The document is being processed.
+ **TEMPORARY_ISSUE**: An unattended error occurred during processing. The process can be run again later we better hopes of success.
+ **FINAL_ISSUE**: The document has been processed but it has an invalid content. It will no be processed again.
+ **CLOSED**: The document has been successfully processed and the result is ready.

When a document is in the **CLOSED** state you can obtain the process result invoking the **getResult()** method on the document object.

    Document document = activity.getDocument("c7e3d6f4-ad56-42f2-bcd5-f217a24089ee");
    byte[] resultBytes = document.getResult();

The content type of the result can be read using the Document.getResultType() method that returns a **DocumentType** value. The *DocumentType* object is an enumeration that can assume the values PDF, JSON or TXT.

### Workflow WebHooks

Due to it's asynchronous processing nature, *AIutino* doesn't return the processed document just when the file is uploaded. Is the client that have the burden to periodically check the document status and discover when the document is in the *CLOSED* state to be able to take the result.

In order to facilitate the writing of asynchronous operations on the client side, it is possible to configure *web-hooks* related to the workflow that are able to inform the client program when the processing of a document is finished.

The *Workflow* object allows the client to set a pair of web hook sets: one for *success processing* and one for *error processing*.

When you have obtained a reference of a Workflow object, you can firstly inspect if the workflow already has some registered *webhook* invoking the **getSuccessWebHooks()** and the **getErrorWebHooks()**

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");
    List<WebHook> successHooks = workflow.getSuccessWebHooks();
    List<WebHook> errorHooks = workflow.getErrorWebHooks();

A **WebHook** is an object that contains two properties: the calling *method* and the calling *url*.

    {
        "method" : "GET",
        "url" : "https://myurlhost/myurlpath"
    }

When a process has completed the system checks the Workflow and if one or more WebHooks are present it calls the webhook url sending the *documentId* to the called url.
If the webhook uses the *GET* method, a query string is attached to the url to send the *documentId* property to the destination:
    
    https://myurlhost/myurlpath?doc=c7e3d6f4-ad56-42f2-bcd5-f217a24089ee

If, in the contrary, the *POST* method is used then the call sends a form-data body content with the pair "doc" and the *documentId*

    doc=c7e3d6f4-ad56-42f2-bcd5-f217a24089ee

To customize a workflow and add one or more success or error web hooks first you have to build a **WebHook** instance using the WebHookBuilder provided by the workflow's getWebHookBuilder() method.

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");
    WebHook webHook = workflow.getWebHookBuilder()
                              .url("https://myurlhost/myurlpath")
                              .method("GET")
                              .build();

To join the WebHook to the workflow you can use the **addErrorWebHook** or **addSuccessWebHook** methods

    workflow.addSuccessWebHook(webHook);

To remove a previously added webhook you can invoke the **removeErrorWebHook** or the **removeSuccessWebHook**

When you have finished to set the hooks, the changes can be saved invoking the *save()* method on the workflow

    workflow.save();

After saving the changes the workflow persists the settings and all the future processing will be run using the custom user changes.

### Data management

Depending on the quantity of user data and the user *workflows* and document's organization into *activities*, the need to clean up the data may arise.
The AIutino SDK offers functions to delete *Documents* and/or *Activities*, to mantain the space clean and efficient. 
The *Workflows* are structural elements of the environment and cannot be deleted by the user.

#### Delete a document

A document can be deleted from the Activity it resides in. The operation removes the result of processing and the source file from the system. This operation cannot be undone.

    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");
    Document document = activity.getDocument("c7e3d6f4-ad56-42f2-bcd5-f217a24089ee")
    activity.deleteDocument(document);

If no exception are raised the operation is considered successfully done.

#### Delete an activity

Only an empty Activity can be deleted. An activity can be deleted from the Workflow it resides in. The operation removes the empty Activity and cannot be undone.

    Workflow workflow = session.getWorklfow("eac1e93c-775c-4edd-960a-167b4ca7a13e");
    Activity activity= workflow.getActivity("b729e7b4-08da-49c0-8b34-264912016382");
    workflow.deleteActivity(activity);

If no exception are raised the operation is considered successfully done.

