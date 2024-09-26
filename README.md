# Inter-Process-Communication-IPC-Kotlin


## Sequence Diagram For the Entire IPC Mechanism on Both the apps.

![mahakali-Server Workflow (1)](https://github.com/user-attachments/assets/a8b31f45-1805-45ae-86f8-051e978df7e3)


# Inter Process Communication:

IPC, or inter-process communication, refers to a general concept of communication between separate processes. In Android, IPC encompasses two specific scenarios:

1. Communication between different applications.
2. Communication between processes within a multi-process application, where components like Activities, Services, Broadcast Receivers, or Content Providers are running in separate processes.


### Android offers three primary methods for inter-process communication (IPC):
  1. AIDL (Android Interface Definition Language)
  2. Messenger
  3. Broadcast

 - Although traditional Linux-based techniques (like socket communication or shared files) can also be used for IPC, Android’s official documentation recommends choosing one of these three Android-specific methods for better security. These approaches offer advantages such as authenticating the application initiating the communication.

## Which method should you use? 
If you're unsure which IPC method is right for your situation, you can refer to the following decision-making chart for guidance.

![image](https://github.com/user-attachments/assets/08f288bb-6ac4-4d16-95bd-c8e645257a61)

# Android IPC Mechanisms #1 — AIDL:

You should opt for AIDL (Android Interface Definition Language) if your application requires a multithreaded IPC structure. If concurrent operations are unnecessary, you can consider using the Messenger method instead.

With AIDL, one application can invoke methods in another application on the same device, a concept known as Remote Procedure Call (RPC). Since methods can be called from different processes or threads simultaneously, it's crucial to ensure the mechanism handling these calls is thread-safe.


## What is AIDL?

AIDL stands for Android Interface Definition Language. If you're familiar with the concept of IPC, you can think of AIDL as Android's version of an Interface Definition Language (IDL). This method is called AIDL because the communication interface between applications or processes is established using the AIDL language. AIDL has a syntax similar to Java, making it easier for developers familiar with Java to use it for inter-process communication on Android.

### Let's implement a scenario with the following features:

- The client retrieves the server's process ID (PID) and the number of connection requests the server has received.
- The client obtains the total count of connection requests made to the server.
- The client sends its own information, such as its PID and package name, to the server application. The server then displays this information as the details of the last connected client on the UI.

In this method, communication relies on the Binder architecture. To initiate a service, the `bindService` method is used. A component within the application or a remote process can bind to the service, and a binder object is returned to the component (such as an activity or service) that binds to it. We won’t delve deeper into the concept of Bound Services here, as it’s beyond the scope of this article.

In this example, the client will bind to the service of the server application, allowing the client to access the methods defined in the server.


### Creating the Server Application

First, start by creating a new project. In this example, we'll name it **IPCServer**.

### Creating an AIDL File

Next, create an AIDL file that contains the methods the client will call. To do this, right-click on the project explorer (on the left side of the IDE) and follow the steps:

### `New -> AIDL -> AIDL File`

When you create the AIDL file this way, it will automatically be placed in the `aidl` folder, which is located alongside the `java` and `res` directories. For example, in this project, the path to the file will be `src/main/aidl/com/piyush/ipcserver/`.

**Modifying the AIDL File**

Now, let's modify the AIDL file to add custom methods instead of the default `basicTypes` method. Here's what each method will do:

1. **getPid**: Retrieves the process ID (PID) of the server application.
2. **getConnectionCount**: Returns the number of client connection requests the server has received during its lifecycle.
3. **setDisplayedValue**: The client sends information about itself, such as the package name, PID, and a custom message, to the server. The server will display this information in its GUI.

Here's how the modified AIDL file should look:

// IServerAidl.aidl
package com.piyush.ipcserver;

// Declare the AIDL interface
interface IServerAidl {
    // Method to get the process ID of the server application
    int getPid();

    // Method to get the total number of client connection requests
    int getConnectionCount();

    // Method for the client to send its package name, PID, and custom text
    void setDisplayedValue(String packageName, int pid, String message);
}

With this structure, the client can call these methods to retrieve and send data to the server, while the server handles and responds to the client's requests.


When working with AIDL, there are specific restrictions on the types of parameters and return values that methods can use. Here's a list of all supported types:

- **Primitive types**: All Java primitive types, such as `int`, `long`, `char`, `boolean`, etc.
- **Arrays of primitive types**: Such as `int[]`, `float[]`, etc.
- **String**: Standard Java `String` type.
- **CharSequence**: For handling sequences of characters.
- **List**: The elements in the `List` must be one of the supported data types or other AIDL-generated interfaces or parcelables. You can also use parameterized types (e.g., `List<String>`). The other side will always receive an `ArrayList`, but the method is generated using the `List` interface.
- **Map**: Similar to `List`, all elements in the `Map` must be supported types or AIDL-generated parcelables. However, parameterized maps (e.g., `Map<String, Integer>`) are **not supported**. On the other side, a `HashMap` is returned, though the method is generated with the `Map` interface. For more flexibility, you can consider using a `Bundle` instead of a `Map`.

If you want to use custom classes as parameters or return types, they need to implement the **Parcelable** interface. This ensures that Android can serialize and parse them into a form the operating system understands for IPC.

By adhering to these type restrictions, you can ensure that your AIDL methods can effectively handle communication between processes.


### Directional Tags: in, out, and inout

In AIDL, all non-primitive parameters require a directional tag to specify the flow of data: **in**, **out**, or **inout**.

- **in**: Indicates an input-only parameter. This parameter sends data from the client to the server but does not receive data back.
  
- **out**: Specifies an output-only parameter. This means that the parameter will not contain meaningful data on input but will be filled with data during the method execution. For example, a method that copies an array of bytes might be defined as:
  
  void copyArray(in byte[] source, out byte[] dest);
  

- **inout**: Indicates that the parameter has significance in both input and output contexts. It can both send data to the server and receive data back. For example:
  
  void charsToUpper(inout char[] chars);
  

These tags are crucial because every parameter must be marshalled (serialized, transmitted, received, and deserialized) for IPC. By using the **in**, **out**, and **inout** tags, the Binder can optimize the marshalling process, skipping unnecessary steps for parameters that don’t require it, thus improving performance.

Thanks to Perihan Mirkelam for this insightful explanation!

### Generating the Stub

Once you have created your AIDL file, follow these steps to build the project:

1. Go to **Build** in the menu.
2. Select **Rebuild Project**.

After rebuilding, you will notice that a file with the same name as your AIDL file, but with a `.java` extension, is automatically generated. This file contains an interface that includes an abstract subclass named `Stub`, which implements the methods you defined in the AIDL file. In the server's service, you will extend your binder object from this `Stub` class.

### Defining Data Classes

1. **Create the Client Data Class**: Start by defining a data class for the client.

2. **Create a Singleton RecentClient Class**: Next, implement a singleton class named `RecentClient` to store information about the last connected client. Although you could maintain a list of all connected clients, this example will focus on displaying only the details of the most recently connected client in the UI.

### Creating a Service

1. **Create the Service**: Begin by creating a service class.

2. **Implement the Binder Object**: Extend your service class from the `Stub` class to create your binder object. In this class, implement the methods defined in your AIDL interface.

3. **Return the Binder Object**: Use the `bindService` method to return your binder object to the client.

4. **Update RecentClient**: When a client binds to or unbinds from the service, update the `RecentClient` object to ensure the UI reflects the latest connection information.

**Note**: If you plan to perform concurrent operations within the methods of the binder object, consider using Coroutines for better handling of asynchronous tasks.



### Defining the Service in the Manifest

Ensure to add the service definition in your `AndroidManifest.xml` file. Here’s how to do it:

1. **Declare the Service**: Inside the `<application>` tag, define your service.

2. **Add an Intent Filter**: Include an intent filter to specify the action associated with the service. This will mark the service as exported by default, allowing it to be accessed from other processes.

Here's an example of how your manifest entry might look:

<service
    android:name=".YourServiceClass"
    android:exported="true">
    <intent-filter>
        <action android:name="com.piyush.ipcserver.ACTION" />
    </intent-filter>
</service>

Make sure to replace `YourServiceClass` with the actual name of your service class. The action name defined here will be used in the client application to interact with the service.

Using ViewBinding
To simplify the code, we can utilize the ViewBinding library. Let's start by declaring its use in our project.

Modifying the Activity
Next, we will update the Activity to display the last connected client information. The UI layout will be hidden if no client is connected.

Creating the Client Application
Project Creation: Create a new project and name it IPCServer.

Project Structure: This application will serve as a foundation for exploring other IPC methods, including Broadcast and Messenger. Therefore, select Bottom Navigation Activity from the activity gallery during project creation.

Adding the AIDL File
After opening the project, add the AIDL file that was created in the server application. Follow these steps:

Navigate to New -> AIDL -> AIDL File.
Ensure that the AIDL file is located in the same directory in both applications. When created via the IDE, it will typically be under src/main/aidl/com/piyush/ipcclient/. Update it to src/main/aidl/com/piyush/ipcserver/.
Copy and paste the content from the original AIDL file, including the package name.
Setting Up the UI
Edit Menu: Modify the menu file located in the /res/menu/ directory.
Edit Navigation: Update the navigation file in the /res/navigation/ directory.
Edit String Resources: Change the string resources in the corresponding file.
Theme Customization: Alter the colors in the theme to distinguish it from the server application.
No Changes in Activity XML: Do not modify the Activity XML file.
Configuring Navigation and Binding in Activity
Next, create a fragment for handling client interactions.

Creating the Fragment
XML Layout: Create the XML layout for the fragment.
Connect/Disconnect Button: Implement a button to manage connection states.
Hidden Layout: Add a layout that appears when a connection is established, displaying information received from the server.
Implementing Business Logic in Fragment
View Binding: Handle view binding in the fragment.
Service Binding: Bind to the server service when the Connect button is pressed. Use the BIND_AUTO_CREATE flag for automatic service creation if it's not already running.
ServiceConnection Interface: Implement the ServiceConnection interface to listen for connection status.
Message Sending: When connected, send the message containing the package name and PID to the server and display the responses on the UI.
Unbinding: When the connection is broken, unbind from the service to clean up resources. Avoid remaining bound to the service if it’s not being used, as the system may terminate background services that lack user interaction.
Client Application Completion
Now that we have implemented the client application, it is ready to communicate with the server. The UI will reflect the interaction flow when invoking server methods.

Improving Applications for Messenger Support
Understanding Messenger
Messenger is a Handler sent to a remote process. This technique operates on the Binder architecture similar to AIDL but simplifies implementation. However, while Messenger is easier to use, it has limitations compared to AIDL:

AIDL supports concurrent operations, while Messenger queues and executes calls sequentially.
With AIDL, multiple calls can be received simultaneously from different processes/threads, whereas Messenger guarantees that only one message is processed at a time.
Server Application Enhancements
Manifest Declaration: Add the action "messengerexample" in the Manifest to be sent from the client to the service.

Service Update: Create a Handler object in the service to manage messages from the client. Update the RecentClient object to reflect the GUI based on incoming messages.

Messenger Creation: Create a Messenger using the Handler object. This object facilitates communication between the client and server. The msg.replyTo object from the client's message can be used to send replies back to the client.

Constants: Define the constants used as bundle keys in messages.

onBind Method: Update the onBind method in the service to return the Messenger object when a client binds with the "messengerexample" action.

Client Application Adjustments
Bundle Key Constants: Add the same constants used in the server application.

Fragment UI: Use the same XML layout as before, but with a different button name.

Fragment Update
Messenger Objects: Define two Messenger objects:

clientMessenger: For sending messages to the server.
serverMessenger: For the server to use when replying to messages.
Handler Definition: Create a handler to process incoming messages and update the UI accordingly.

Binding Logic: Bind to the server upon clicking the connect button and unbind upon disconnecting. When a connection is established, send a message to the server using message.replyTo = clientMessenger to specify the expected response.

Connection Loss Handling: Clear the GUI information when the connection is lost.

Final Fragment Implementation
The final version of the fragment will seamlessly manage communication between the client and server.

Exploring Broadcasts
Broadcasts are a familiar IPC technique for Android developers. Unlike Messenger and AIDL, broadcasts can be sent by the system or applications, allowing receivers to listen for specific broadcasts using BroadcastReceiver.

Security and Permissions
Receivers are typically exported and can receive broadcasts from any application. However, you can implement a simple security layer using permissions or action filters in the <receiver> tag in the Manifest.

Explicit Intents
For this example, we will send a broadcast to specific applications using explicit intents, defining both the package name and class name of the receiver to limit access.

Client Application Broadcasting
Sending Broadcasts: Upon button press, send a broadcast containing the client’s information and display the broadcast time on the screen.
Server Application Updates
Receiver Declaration: Export the receiver in the Manifest.
Broadcast Intent Filter: Add the filter to match the broadcast intent. It does not need to be a package name; using a unique string is preferred to avoid conflicts.
Receiver Class Creation: Implement the receiver class to handle incoming broadcasts.
UI Update: Update the client information on the UI when receiving broadcast data.

